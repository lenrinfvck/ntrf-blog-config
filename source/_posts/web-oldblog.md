---
title: 博客迁移基本完成，纪念下坑掉的老blog  
date: 2016-11-27 18:41:00  
tags:  
    - WEB项目
    - nodejs
    - express
    - mongodb
categories:
    - WEB前端
---

{% asset_img 01.jpg 新老站点合影 %}

# 1.技术栈对比
## 现在
使用基于[hexo](https://hexo.io/)的静态博客，本地是hexo搭建的动态页面，部署时使用自带命令生成静态html文件，之后push到github后通过gitpage呈现。文章书写主要依靠markdown和html。 
  
静态博客，免费服务器，免费域名[lenrinfvck.github.io](https://lenrinfvck.github.io)。  
虽然是静态页面，但是可以借由很多第三方云服务使用一些动态功能呢，如回复统计一类的。    
## 以前
自己使用 node+express+mongodb，部署在阿里云服务器上，linux主机预装centOS的裸机。所有动态数据存入数据库动态展示，文章使用了动态编译markdown的形式，同时也自行构建了一些语法功能，如嵌入标签分类和时期等。   

代码部署在[coding](https://coding.net/)的git上，使用[webhook](/2016/01/19/web-webhook-coding/)自动同步到服务器。

动态博客，阿里云计流量28.8元/月，域名[lenrinfvck.cn](https://lenrinfvck.cn) (服务器到期) 39元/年，维护博客本身就很烦，更难得写文章。忙起来后就没咋弄了，坑掉了。

<!-- more -->

# 2.相关博文
+ [Node服务器搭建 - 阿里云ECS](/2016/01/19/web-webhook-coding/)  
+ [git代码自动部署，coding中webhook使用](/2016/01/17/web-node-server/)  
