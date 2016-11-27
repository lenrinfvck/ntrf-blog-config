---
title: Node服务器搭建 - 阿里云ECS  
date: 2016-01-17 01:30:00  
tags:  
    - nodejs
    - express
    - mongodb
categories:
    - WEB前端
---

> [服务器]： 阿里云ECS，1核512MB，流量收费(￥28.8/月，￥0.72/GB)  
 [系统]：CentOS6.5 32位  
 [环境]：Node + Express + Mongodb3.0.6  
 [本机]：OSX EI Capitan  

Linux，OSX直接以ssh链接，Win使用putty链接

<!-- more -->

# 1.服务器基本操作
首先选配安装，设置root密码，获得公网IP。

## 1.1 服务器登陆
主机控制台`ssh root@[ip]`登陆后按提示输入root的密码。  
如：`ssh root@110.12.32.12`  

## 1.2 安装FTP
服务器控制台执行`yum install vsftpd -v`，  
设置为开机启动`chkconfig vsftpd on`
*注：如果出现安装到一半链接断掉，再次执行yum时会出现进程锁，可以`ps -ef|grep yum`查找进程id后`kill -9 id`结束该进程。*  

## 1.3 使用SFTP
不使用FTP，直接用FileZila/WinSCP等软件进行SSH链接来传输文件。

## 1.4 更新git
删除原始git  
`yum remove git`  
从github上下载一个最新版：  
```bash
cd /usr/local/src
wget https://github.com/git/git/archive/master.zip
unzip master.zip
#编译安装
cd /usr/local/src/git-master
make configure
./configure --prefix=/usr/bin
make
make install
#覆盖原有命令
\cp -fr /usr/local/src/git-master/git /usr/bin
git --version
```
error解决：
```bash
#make时与zilb有关的错误
yum install zlib-devel -y
#运行git clone https，https报错
yum install curl-devel
cd /usr/local/src/git-master 
./configure
make
make install
```

# 2.安装Node
安装nvm用于管理node，详情见[http://fengmk2.com/blog/2014/03/node-env-and-faster-npm.html](http://fengmk2.com/blog/2014/03/node-env-and-faster-npm.html)    

注：退出ssh后需要重新nvm use的问题。
```bash
#使用具体版本号
nvm use 4.1.2
nvm alias default 4.1.2
```

# 3.安装Mongodb
```bash
#下载安装包并解压，32系统只能安装对应的i686的32位版本  
wget https://fastdl.mongodb.org/linux/mongodb-linux-i686-3.0.6.tgz
tar -zxvf mongodb-linux-i686-3.0.6.tgz

#移动到安装目录
mv mongodb-linux-i686-3.0.6 /usr/local/mongodb

#新建数据库目录
mkdir -p /app/mongodb/db
mkdir -p /app/mongodb/logs

#添加mongodb.conf文件
cd /usr/local/mongodb/bin
vi mongodb.conf
#文件内容 START#
dbpath = /app/mongodb/db #数据文件存放目录  
logpath = /app/mongodb/logs/mongodb.log #日志文件存放目录  
port = 27017  #端口  
fork = true  #以守护程序的方式启用，即在后台运行  
nohttpinterface = true  
#auth = true  #使用用户模式连接，暂时不开启
#文件内容 END#

#启动
/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/bin/mongodb.conf
#启动mongodb命令行
/usr/local/mongodb/bin/mongo

#重启
ps -ef | grep mongodb #得到mongodb进程id
kill -15 id
/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/bin/mongodb.conf
```
创建用户
```bash
#在mongo命令行下
use admin
db.createUser({user:'admin',pwd:'admin',roles:[{role:'userAdminAnyDatabase',db:'admin'},{role:'readWriteAnyDatabase',db:'admin'}]})
#之后在config文件中将auth=true启用，并重启mongodb，之后就只能使用相应用户才能登陆
```

>图形界面  
>mongoHub   兼容性较好，最新版支持mongodb3.0+  
>Robomongo  界面简单清晰，用户模式不支持mongodb3.0+，需要修改认证模式，比较麻烦。

# 4.Node部署
## 4.1 从Git部署代码
```bash
git clone https://github.com/xxx/xxx.git
cd xxx
node app.js
```

## 4.2 持久运行forever
ssh断开后node程序会终止，使用forever持久运行  
```bash
#使用forever持久运行
npm install -g forever
forever start app.js

#其他
forever stop app.js     #停止程序
forever list    #运行中的清单
forever -w start app.js     #源码修改时自动重启
```

## 4.3 Upstart维护正常运行
服务器重启或崩溃后重启使用，同理可用于Vagrant等本地虚拟环境  
```bash
#安装
yum install upstart
#在/etc/init下添加配置文件
touch /etc/init/mynode.conf
```

`mynode.conf`内容如下
```bash
#自带的upstart版本比较低，有些语法不支持，以下写法亲测有效
author "lenrinfvck"
description "mynode"

#start on (local-filesystems and net-device-up IFACE=eh0)
start on startup
stop on shutdown
respawn  
#env NODE_ENV=production  
#此node路径为nvm设置的PATH路径，非nvm安装的使用其他相应路径
exec /var/gitclone/nvm/versions/node/v4.1.2/bin/node /www/app.js 
```

`mymongodb.conf`内容如下
```bash
description "lrfmad-mongodb"
author "lenrinfvck"
limit nofile 20000 20000
kill timeout 300
respawn
respawn limit 2 5
start on runlevel [2345]
stop on runlevel [06]
exec  /usr/local/mongodb/bin/mongod --config /usr/local/mongodb/bin/mongodb.conf
```

启动Upstart  
`start mynode`以及`start mymongodb`  
>\>mynode start/running, process 6770  

*之后可以尝试重启服务器，测试环境是否启动*



