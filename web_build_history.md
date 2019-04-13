---
title: 本站建设当中的问题与解决方法
tags:                      
- 记录
categories:                
- 记录
date: 2017-10-1
---

**以下所使用的内容均为过去的记录，网站已经不使用下面的方式**
**如有疑问，本人不负责解答**

本网站使用的是 阿里云linux一键安装web环境 

本人并不会进行相干的优化改进，只会使用网络已有的教程进行 大量的搜索 以解决，

如果你不使用 Aapche +  SSL 下安装WordPress，这篇文章或许对你用处不大，

至于 Nginx 下安装 SSL，阿里云有相应的教程，

我原本是在 www文件夹 下安装的 WordPress，

但是Apache的SSL安装完，默认的index.php 主页 在文件夹  A下（具体忘了），

所以我就把index.php删掉，在这个 A文件夹下安装的WordPress。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~我是华丽的分割线~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

以下两个网站推荐看看：

w3school

Codecademy网站网站

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~我是华丽的分割线~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

WordPress的的更新插件失败“需要访问您网页服务器的权限”

WordPress更新插件失败“需要访问您网页服务器的权限”， 请输入您的FTP登录凭据以继续。 如果您忘记了您的登录凭据（如用户名、密码），请联系您的网站托管商。

----------------------------------

```bash
chown -R www /usr/share/nginx/html ##(修改成网站域名目录)

chmod -R 777 /usr/share/nginx/html
```

----------------------------------

`chown -R www` 命令给予权限，如果你是自己安装的，没有使用阿里云的脚本，那么所有者应该是 `chown -R www-data`

原文的做法我没有全部用到，我仅仅 做了第一步便可以了

我在 A 下安装了 B，我以为只需要给 B 权限即可，无奈不行，重新做了一遍，发现需要给 A 权限。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~我是华丽的分割线~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PHP实现HTTP与HTTPS转化

Apache 安装完 SSL ，index.php修改代码实现 http 自动跳转 https

网页开头加入以下代码，或者 我的做法是，index.php代码全部删除，替换为下列代码：

代码推荐  点击上方链接  原文复制

---------------------------------

```php
<？php 
// http转化为https 
if（$ _SERVER [“HTTPS”] <>“on”）
{ 
$ xredir =“https：//”。$ _ SERVER [“SERVER_NAME”]。
$ _SERVER [ “REQUEST_URI”]; 
header（“Location：”。$ xredir）; 
} 
？>
```

如果网页使用HTTP访问，在网页开头加入以下代码：

```php
<？php 
// https转化为http 
if（$ _SERVER [“HTTPS”] ==“on”）
{ 
$ xredir =“http：//”。$ _ SERVER [“SERVER_NAME”]。
$ _SERVER [ “REQUEST_URI”]; 
header（“Location：”。$ xredir）; 
} 
？>
```

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~我是华丽的分割线~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

关于 WordPress使用QQ登录：QQ互联

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~我是华丽的分割线~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Apache的配置HTTPS协议（ZT）

1，“c：/apache/conf/extra/httpd-ssl.conf的第80行上的语法错误：ErrorLog接受一个参数错误日志的文件名”或者“C：/ apache / conf / extra / httpd- ssl.conf：SSLCertificateFile接受一个参数，SSL服务器证书文件（'/ path / to / file'-PEM或DER编码）“ 

解决方法：
文件路径加双引号

2，”语法错误在线76的C：/apache/conf/extra/httpd-ssl.conf：SSLSessionCache：（？mod_socache_shmcb） 'shmcb'会话缓存不支持（已知名称），也许你需要加载相应的socache模块“ 

解决办法：
打开httpd.conf，找到LoadModule socache_shmcb_module modules / mod_socache_shmcb.so，把前面的注释去掉。

