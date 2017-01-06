---
title: particleground源码阅读笔记 | 轻量级canvas粒子插件
date: 2017-01-07 00:39:14
tags:  
    - canvas
categories:
    - WEB前端
---
{% asset_img bg.jpg%}

首先可以来看下这个插件的demo：[https://jnicol.github.io/particleground/](https://jnicol.github.io/particleground/)  
一种比较常见的粒子背景，有连线，有3d视差效果。  
代码：[github地址](https://github.com/jnicol/particleground)  

源代码很短，本文也算不上啥解析文章，只是阅读笔记。 

# 1.基本的粒子动画  
在canvas画布上绘制复数实心圆点即粒子，每个粒子都有它的粒子对象实例，用于记录它的大小、位置信息以及速度，通过timeout来更新粒子的位置形成粒子动画。大概类似下面这样：  

```js
var allParticles = [];
// 初始化粒子
for(var i = 0; i < 100; i++) {
    var p = new Particle();
    allParticles.push(p);
}
setTimeout(function() {
    allParticles.forEach(function(item) {
        // 更新粒子位置
        item.update();
    });
    // 根据各个粒子的位置绘制到canvas上
    draw(allParticles);
}, 30);
```

<!-- more -->

# 2.particleground源代码
该插件可作为jquery插件使用，同时也直接在window上挂载了相应方法，使用时只需传入舞台dom对象和配置信息即可。  

```js
particleground(document.getElementById('particles'), {
    dotColor: '#5cbdaa',
    lineColor: '#5cbdaa'
});
```
此方法会去实例化Plugin构造函数，然而返回的是一个有闭包关系的对象。  
```js
function Plugin() {
    // 一些模块的私有变量，由于存在闭包，会持续存储着
    var particles = [];
    var mouseX = 0;
    var mouseY = 0;
    ... ...

    // 一些方法函数定义
    function init() {}
    function start() {}
    init();

    return {
      option: option,
      destroy: destroy,
      start: start,
      pause: pause
    };
}
```
> 如上，执行逻辑只有一个`init()`函数。   

## 2.1 初始化 - init()
```js
// 浏览器能力检测  
var canvasSupport = !!document.createElement('canvas').getContext;

function init() {
  if (!canvasSupport) { return; }
  // 创建canvas标签添加到页面
  ... ...
  // 计算需要的粒子数
  // density - 稠度，即每隔一定像素空间才生成一个粒子
  var numParticles = Math.round((canvas.width * canvas.height) / options.density);
  // 创建粒子对象实例，并缓存起来方便使用
  for (var i = 0; i < numParticles; i++) {
    var p = new Particle();
    p.setStackPos(i);
    particles.push(p);
  };

  // 一些事件的绑定
  window.addEventListener('resize', function() { ... ... }, false);
  document.addEventListener('mousemove', function(e) { ... ... }, false);
  // 当移动端支持重力感应事件时使用
  if (orientationSupport && !desktop) {
    window.addEventListener('deviceorientation', function () { ... ... }, true);
  }
  // 绘制并开始动画
  draw();
  // 触发事件回调  
  hook('onInit');
}
```
> 其中，在调用时就会执行的主要是`new Particle()`创建粒子对象，以及`draw()`的绘制。

## 2.2 粒子构造函数 - Particle()
```js
function Particle() {
  this.stackPos;
  this.active = true;
  // 随机生成粒子的Z轴位置，制造出不同层级粒子的视差效果
  this.layer = Math.ceil(Math.random() * 3);
  // 为粒子生成随机位置属性
  this.position = {
    x: Math.ceil(Math.random() * canvas.width),
    y: Math.ceil(Math.random() * canvas.height)
  }
  // 根据配置的粒子移动方向以及最大最小速度，生成随机速度
  this.speed = {}
  // X轴方向速度
  switch (options.directionX) {
    case 'left':
      ... ...
    case 'right':
      ... ...
    default:
      // 正负随机，即生成随机方向及大小的速度
      this.speed.x = +((-options.maxSpeedX / 2) + (Math.random() * options.maxSpeedX)).toFixed(2);
      this.speed.x += this.speed.x > 0 ? options.minSpeedX : -options.minSpeedX;
      break;
  }
  // Y轴方向速度，同上
  ... ...
}
```

## 2.3 初始化完成，开始绘制 - draw()
```js
function draw() {
  // 绘制前清除画布
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // 更新粒子位子
  for (var i = 0; i < particles.length; i++) {
    // 更新的结果主要是改变粒子位置属性[x, y]，每次调用会叠加一次速度值
    particles[i].updatePosition();
  };
  // 绘制粒子
  for (var i = 0; i < particles.length; i++) {
    // 粒子位置已计算更新，追加上偏移值进行绘制，并找出与该点距离为一定范围的点进行线条绘制
    particles[i].draw();
  };
  // timeout实现动画
  if (!paused) {
    // 实际就是调用了setTimeout()，返回值raf即timeout的id，可以用于清除延时
    raf = requestAnimationFrame(draw);
  }
}
```
> 如上，比较重要的就是粒子实例方法`.updatePosition()`

## 2.4 计算粒子位置 - particle.updatePosition()
```js
Particle.prototype.updatePosition = function() {
  if (options.parallax) {
    if (orientationSupport && !desktop) {
      ... ...
      // 以上为重力感应时的逻辑，在PC时直接使用鼠标位置
    } else {
      pointerX = mouseX;
      pointerY = mouseY;
    }
    // 计算偏移值，[winW/2, winH/2]即画布中点，比较鼠标位置和中点计算出一个偏移向量
    // 依照此偏移量来计算偏移值，再乘以不同的Z轴系数就能制造视差效果了
    this.parallaxTargX = (pointerX - (winW / 2)) / (options.parallaxMultiplier * this.layer);
    this.parallaxOffsetX += (this.parallaxTargX - this.parallaxOffsetX) / 10; 
    this.parallaxTargY = (pointerY - (winH / 2)) / (options.parallaxMultiplier * this.layer);
    this.parallaxOffsetY += (this.parallaxTargY - this.parallaxOffsetY) / 10; 
  }

  var elWidth = element.offsetWidth;
  var elHeight = element.offsetHeight;
  
  // 计算时校验超出边沿的情况, 超出时反向移动
  switch (options.directionX) {
    case 'left':
      ... ...
    case 'right':
      ... ...
    default:
      if (this.position.x + this.speed.x + this.parallaxOffsetX > elWidth || this.position.x + this.speed.x + this.parallaxOffsetX < 0) {
        this.speed.x = -this.speed.x;
      }
      break;
  }

  // 累加速度到位置上
  this.position.x += this.speed.x;
  this.position.y += this.speed.y;
}
```

到此，粒子系统初始化到定时触发动画的代码大致就是这些了。



