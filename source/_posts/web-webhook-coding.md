---
title: git代码自动部署，coding中webhook使用  
date: 2016-01-19 22:32:00  
tags:  
    - githook
    - webhook
categories:
    - WEB前端
---

> 环境：  
    NodeJs: 4.0+  
    git: 1.8+  
> 官方说明: [webhook说明](https://open.coding.net/webhook.html)  

# 工作原理
基于git的githook功能，coding提供的webhook服务，github也有类似服务。  
监听对远程仓库的操作，执行相应操作，此处webhook提供的处理是发送一个http请求到指定url地址。这时就可以在部署服务器上开个路由监听这个地址的访问，如果是coding发出的，则git pull拉去代码，并重启服务等。  

# 步骤
## 1.webhook监听创建
{% asset_img webhook01.jpg webhook设置界面 %}  
在此处新建一个监听，填入目标URL地址和加密口令，一般是监听push事件。 

<!-- more -->

## 2.部署服务器处理webhook请求
参考官方写法：[coding案例写法](https://open.coding.net/webhook.html)
```js
var process = require('child_process');

module.exports = function(req, res, next) {
    console.log('print', req.body);
    console.info(req.body["token"]);
    if('口令' === req.body['token'] ){
        //console.info(process);
        process.exec('git pull origin master:master', {'cwd':'/www/'}, 
            function (error, stdout, stderr) {
                //配置了upstart后的重启服务
                process.exec('restart mynode');
                console.log('stdout========================\n' + stdout);
                console.log('stderr========================\n' + stderr);
                if (error !== null) {
                    res.send('<pre>fail!!!\n' + stdout + error + '</pre>');
                } else {
                    res.send('<pre>done!!!\n' + stdout + '</pre>');
                }
            });
    } else {
        console.log(' failed token ')
        res.send('<pre>token不正确?</pre>');
    }
}
```
之后可在步骤1的设置界面出测试webhook是否成功，在配置界面会有webhook发送接收的记录信息。  





