---
title: EaselJs - DIsplayObject相关笔记  
tags:
    - canvas
    - creatJs
---

EaselJs主要负责对画布元素的抽象，定义各种对象，管理画布
>官方文档：[中文文档](http://www.createjs.cc/easeljs/docs/modules/EaselJS.html)  

### 绘图流程
创建舞台，容器 -> 创建`DisplayObject`相关实例并添加进舞台
<!-- more -->
### DisplayObject - 展示元素基类
##### 【Props】
属性 | 限制 | 描述
--- | --- | ---
alpha   | Number[0~1]   | 透明度  
id      | Number        | 元素id  
name    | String        | 元素名称  
visible| Boolean        | 是否可见  
cursor  | String[同Css]  | 指针样式  
x, y    | Number[px]     | 相对于父级的坐标位置  
regX, regY | Number[px]  | 旋转中心点  
scaleX， scaleY| Number[默认1] | 缩放
rotation| Number        | 旋转角度  
parent  | obj[readOnly] | 父级容器或则stage  
mask    | Obj[Shape]    | 遮罩对象  
shadow  | Obj[Shadow]   | 阴影  
hitArea | Obj[DisplayObject] | 点击区域  
stage   | Obj[readOnly] | 返回添加进的那个stage实例，没有添加时为null  

##### 【Methods】
方法 | 描述
--- | ---
addEventListener | 同原生  
off/removeEventListener | 注销事件监听  
on  | 能传递跟多可配置项的绑定事件  
dispatchEvent(event) | 发送广播事件  
clone | 复制该实例  
draw(ctx, [ignoreCache=false]) | 将元素画到ctx画布上  
setTransform(x, y, sX, sY) | 设置变换属性  
getBounds()  | 获取矩形边界 x,y,w,h的对象
setBounds(x, y, w, h) | Shape等无法获取，但可以设置  
globalToLocal(x, y) | 把全局坐标转换为相对于该实例的坐标  
localToGloball(x, y) | 与上相反  

##### 【Events】
事件 | 描述
--- | ---
added   | 该元素被添加后触发  
pressmove | mousedown后滑动时触发
pressup | 滑动释放  
removed | 广播这个元素被删除  
tick | stage渲染时触发，重绘或则追加绘制时  

### Bitmap - 位图
##### 【构造函数】 
`new createjs.Bitmap(imageOrUri)`  
`imageOrUri HTMLImageElement | HTMLCanvasElement | HTMLVideoElement | String`  
  
##### 【私有属性方法】
名称 | 描述
--- | --- 
image | 存储的img对象或则canvas,video对象  

### Graphics - 矢量对象
【构造函数】 `new createjs.Graphics()`  
创建后调用各种方法进行绘制。  
待补充

### Shape - 形状对象
【构造函数】 `new createjs.Shape(Graphics)`  
参数是一个矢量形状对象，由于Graphics只是纯粹提供矢量绘图。而Shape提供了一次包装，从而能使用DisplayObject中的对图形对象的一些基本操作。  

### SpriteSheet - 雪碧图基类、帧动画
该对象理论上是一张关键帧动画的雪碧图的抽象类。  
##### 【构造函数】
```js
var data = {
    //images可为img的dom对象，或则URI路径
    images: ["sprites.jpg"],
    /*控制每帧大小，等大帧，width, height, regX, regY 控制大小，
        spacing为间隔, 
        margin图像边缘,
        count帧数，缺省时会自动计算
      非等大帧时使用x, y, width, height, imageIndex*, regX*, regY*的复数数组
    */
    frames: {width:50, height:50},
    /*可以设置各种动画，动画名:[帧索引]的键值对
        1. 值为Number，表示为单帧动画.
        2. Array，[start, end, next, speed]，next为动画名，speed是速度倍数.
        3. Object， 如下，帧为任意单帧的集合，也支持next，speed.
    */
    animations: {
        stand:0,
        run:[1,5],
        jump:[6,8,"run"],
        walk: {
            frames: [1,2,3,3,2,1]
        },
        shoot: {
            frames: [1,4,5,6],
            next: "walk",
            speed: 0.5
        }
    },
    //帧率
    framerate: 20
};
var spriteSheet = new createjs.SpriteSheet(data);
```

##### 【属性方法】
名称 | 描述  
--- | ---  
complete | boolean，表示图片是否加载完成  
framerate, animations | 如上构造函数中的属性  
getFrame(index) | 获取存储了某一帧的对象字段  
getFrameBounds(index, [rect]) | 返回某一帧的碰撞面积的一个矩形对象  
getNumFrames(animation) | 返回某动画的总帧数  
getAnimation(name) | 通过动画名获取其配置对象  

### Sprite - 雪碧图展示类
##### 【构造函数】 
`new createjs.Shape(spriteSheet)`  
实际超控雪碧图帧动画的播放，和在画布上的各种行为。  
默认展示第一帧。  

##### 【属性方法】
名称 | 描述  
--- | ---  
framerate | 帧率  
spriteSheet | 雪碧图基类实例  
paused | boolean(false)，设为true时默认不会播放动画  
currentFrame | 获取当前帧序号，相对于全雪碧图  
currentAnimation | 当前播放的动画的名字  
currentAnimationFrame | 当前播放的帧在当前动画中第几帧  
advance([time]) | ???  
.play()/.stop() | 开始播放当前动画  
.gotoAndStop(frame/name)/.gotoAndPlay() | 移动到某帧或则某动画并播放和停止 

##### 【Evnet】
事件 | 描述
--- | ---
change   | currentFrame被修改时，正常播放和调用gotoAndPlay之类的    
animationend | 当前动画播放完毕  

### Text - 文字类
##### 【构造函数】 
`new createjs.Text("Hello World", "20px Arial", "#ff7700")`    
一个文本只支持一种字体样式，字体必须在绘制前加载，尽量缓存文本实例。  

##### 【属性方法】
名称 | 描述  
--- | ---  
text/font/color | 对应构造函数，color支持CSS相关颜色，以及canvas的填充样式  
maxWidth | 最大宽度  
outline | 外边框大小
textAlign | 支持"start", "end", "left", "right", "center" 
textBaseline | 基线
lineHeight/lineWidth | 行高，行宽
.getMetrics() | 返回width，height，行相关数组数据  

### Container - 容器
##### 【构造函数】 
`new createjs.Container()`  
分组，预合成，类似外面套一层div。也类似与div，容器也有一定多余消耗，最好不要用于包括一个子集。  

##### 【属性】
名称 | 限制 |描述  
--- | --- | --- 
children | Array | 存储子节点   
mouseChildren | Boolean(true) | false时点击子元素，点击事件会直接在容器触发   
tickChildren | Boolean(true) | false时点击子元素，tick事件不触发到子集    
numChildren | Number(readOnly) | 返回子集数 

##### 【方法】 
名称 |描述  
--- | --- 
.addChild(child)/.removeChild(c) | 添加/删除DisplayObject子集 
.swapChildren(child1, child2) | 替换某个子集 
.contains(child) | 判断child是否是其子孙级，返回为Boolean 
.getObjectsUnderPoint(x, y, [mode=0]) | 获取某点下的所有子集，其顺序由虚拟深度决定，mouseEnabled禁用时无法获取 
.getChildByName(name)/.getChildAt(index) | 获取子节点 
.getChildIndex(child) | 获取index 
.sortChildren(fn(c1, c2, options)) | 类似.sort()，自定义排序规则 
.addChildAt(c, index) | 定向添加子集到某序号处 
.cloneChildren(container) | 克隆所有子节点并添加到其他container 
.removeAllChildren() | 移除所有子节点 

### DOMElement - DOM图层
##### 【构造函数】 
`new createjs.DOMElement(htmlElement)`  
把原生dom对象封装成DisplayObj类型  

##### 【属性】  
名称 | 描述  
--- | --- 
htmlElement | 存取原生dom 

### Stage - 舞台
##### 【构造函数】 
`createjs.Stage("canvasElementId")`  
传入CanvasId来创建舞台，Stage继承自Container  

##### 【属性方法】  
名称 | 描述 
--- | --- 
autoClear | (true)默认为每一帧绘制前都清空舞台  
canvas | 存储舞台canvas对象，可以重新赋值，但是注意禁用之前的舞台  
drawRect | 设置绘制区域  
mouseInBounds | boolean, 表示鼠标是否在舞台区域  
mouseX, mouseY | 鼠标坐标 
nextStage | 链表结构，设置另一个舞台，当主舞台有事件触发也传递给下一个舞台  
preventSelection | boolean，可以调用类似preventDefault()的效果  
.clear() | 清除舞台  
.enableDOMEvents(boolean) | 开启或禁用事件  
.update() | 刷新舞台，会触发tick事件  
.toDataURL([bgColor], [mimeType="image/png"]) | 将舞台元素导出为base64的图片

##### 【事件】  
名称 | 描述 
--- | --- 
drawstart/drawend | 每一帧绘制开始和结束  
tickstart/tickend | 类似上面的draw，但tickend在drawend之前  

### MovieClip - 剪辑对象
##### 【构造函数】 
`new createjs.MovieClip(mode, start, loop, labels)`  
+ mode: INDEPENDENT(默认，独立播放), SINGLE_FRAME(静帧), SYNCHED(同步播放)  
+ start: 开始帧  
+ loop: 是否循环  
+ labels: (obj) 标签与对应帧的键值对，如`{start: 20}`  

##### 【属性】
名称 | 描述  
--- | ---  
autoReset | boolean, true时在默认模式下，播放完后跳到第一帧  
startPostion | number, 起始帧  
currentFrame/currentLabel | number/string, 当前帧/当前标签  
paused | boolean, 可设置为true，暂停播放  
duration | number, 此剪辑对象的总秒或帧数  

##### 【方法】
方法 | 描述  
--- | ---  
.play()/.stop() | 播放, 暂停  
.gotoAndPlay(index)/.gotAndStop(index) | 前往某帧并播放、暂停

