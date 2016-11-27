---
title: 在vagrant搭建基于CentOS6的LAMP开发环境  
date: 2016-01-19 21:41:23  
tags:  
    - php
    - vagrant
categories:
    - WEB前端
---

>关于vagrant安装和使用：[vagrant使用](https://github.com/lenrinfvck/ntrf-doc/blob/master/%E7%8E%AF%E5%A2%83%E5%8F%8A%E5%B7%A5%E5%85%B7/Vagrant%E4%BD%BF%E7%94%A8.md)

本次是在virtual+vagrant下安装的centos，在此基础上安装LAMP环境.  
centos.box下载：[http://www.vagrantbox.es/](http://www.vagrantbox.es/) 

>相关版本：
>LINUX系统: centOS6.4 i386 Minimal

<!-- more -->

# 准备工作
## (1)配置防火墙，开启80端口、3306端口
修改如下文件  
>/etc/sysconfig/iptables  

可直接vi修改，或则使用filezila等软件修改  
    
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT  
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT  

重启防火墙 `$ /etc/init.d/iptables restart`
## (2)关闭SELINUX
修改如下文件  
>/etc/selinux/config  

    #SELINUX=enforcing //注释掉
    #SELINUXTYPE=targeted //注释掉
    SELINUX=disabled //增加

重启系统`shutdown -r now`  

# 安装LAMP
## (1)安装Apache
    yum install httpd  

等待安装，根据提示按y结束  
启动Apache `/etc/init.d/http start`  
开机自启动 `chkconfig httpd on `

## (2)安装MySQL
### 按序输入如下指令  

询问是否要安装，输入Y即可自动安装,直到安装完成    
```bash
yum install mysql mysql-server
```
启动MySQL  
```bash
/etc/init.d/mysqld start  
```
设为开机启动  
```bash
chkconfig mysqld on
```
拷贝配置文件（注意：如果/etc目录下面默认有一个my.cnf，直接覆盖即可）  
```bash    
cp /usr/share/mysql/my-medium.cnf /etc/my.cnf  
```
### 设置root密码

```bash
mysql_secure_installation  
#回车，根据提示输入Y(输入密码，默认为无，即回车)  
#输入2次密码，回车  
#根据提示一路输入Y  
#最后出现：Thanks for using MySQL!  
```

MySql密码设置完成，重新启动 MySQL：
```bash
/etc/init.d/mysqld restart  #重启
/etc/init.d/mysqld stop  #停止
/etc/init.d/mysqld start  #启动
```
## (3)安装PHP5 
### 安装php5
```bash
yum install php  
```
根据提示一路Y  

### 安装PHP组件，使 PHP5 支持 MySQL
```bash
yum install php-mysql php-gd libjpeg* php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt
```

这里选择以上安装包进行安装  
根据提示输入Y回车  
```bash
/etc/init.d/mysqld restart #重启MySql  
/etc/init.d/httpd restart #重启Apche  
```

### 升级php-5.5
删除现有 php-common
```bash
yum -y remove php-common
```
设置新源
```bash
rpm -Uvh https://mirror.webtatic.com/yum/el6/latest.rpm
```
安装新的 php 以及 php 插件
```bash
yum install php55w php55w-cli php55w-common php55w-gd php55w-ldap php55w-mbstring php55w-mcrypt php55w-mysql php55w-pdo php55w-pecl-memcache php55w-devel php55w-xml php55w-xmlrpc php55w-opcache

/etc/init.d/httpd restart #重启Apche
php -v #检查版本 
```
# 相关配置
## (1)Apache配置
配置文件如下
>/etc/httpd/conf/httpd.conf  

+ **/var/www/html**  
    代码根目录，使用vagrant时改为`/vagrant`，同时设置权限`chown apache.apache -R /var/www/html`

+ **ServerTokens OS**  
在44行 修改为：ServerTokens Prod （在出现错误页的时候不显示服务器操作系统的名称）  
    
+ **ServerSignature On**  
在536行 修改为：ServerSignature Off （在错误页中不显示Apache的版本）  
    
+ **Options Indexes FollowSymLinks**  
在331行 修改为：Options Includes ExecCGI  
FollowSymLinks（允许服务器执行CGI及SSI，禁止列出目录）  
    
+ **#AddHandler cgi-script .cgi**  
在796行 修改为：AddHandler cgi-script .cgi .pl （允许扩展名为.pl的CGI脚本运行）
    
+ **AllowOverride None**   
在338行 修改为：AllowOverride All （允许.htaccess）  
    
+ **AddDefaultCharset UTF-8**  
在759行 修改为：AddDefaultCharset GB2312　（添加GB2312为默认编码）  
    
+ **Options Indexes MultiViews FollowSymLinks**  
在554行 修改为 Options MultiViews FollowSymLinks（不在浏览器上显示树状目录结构）
    
+ **DirectoryIndex index.html index.html.var**  
在402行 修改为：DirectoryIndex index.html index.htm Default.html Default.htm
    
+ **index.php Default.php index.html.var**  
（设置默认首页文件，增加index.php）  
    
+ **KeepAlive Off**  
在76行 修改为：KeepAlive On （允许程序性联机）  
    
+ **MaxKeepAliveRequests 100**  
在83行 修改为：MaxKeepAliveRequests 1000 （增加同时连接数）  
```bash
/etc/init.d/httpd restart #重启  
rm -f /etc/httpd/conf.d/welcome.conf /var/www/error/noindex.html #删除默认测试页
```

## (2)PHP配置
配置文件如下

>/etc/php.ini  

+ **date.timezone = PRC**  
在946行 把前面的分号去掉，改为date.timezone = PRC  

+ **expose_php = Off**  
在432行 禁止显示php版本的信息  

+ **magic_quotes_gpc = On**  
在745行 打开magic_quotes_gpc来防止SQL注入  

+ **short_open_tag = ON**  
在229行支持php短标签





