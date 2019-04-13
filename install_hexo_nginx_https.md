---
title: install hexo + nginx + https
tags:                      
- HEXO
- 建站
- nginx
- nodejs
- 阿里云
categories:                
- 教程
- Hexo
date: 2018-11-5
---

本文是对下面两篇文章的提取，如果有疑问，请查看原文档

> <https://qiming.info/%E9%98%BF%E9%87%8C%E4%BA%91CentOS%E4%B8%8BHexo+Nginx%E5%BB%BA%E7%AB%99%E8%BF%87%E7%A8%8B/>
> <https://staunchkai.com/hexo_SSL.html>

## 安装软件

```bash
yum -y install git nodejs screen expect nginx unzip

yum -y update && reboot
```


### 验证

```bash
node -v
npm -v
```

## 安装 hexo

目前npm官方源在国内访问并不稳定，如果你无法直接安装，请更换国内npm源

```bash
npm config set registry https://registry.npm.taobao.org

npm install -g hexo-cli
```

完成后输入`hexo version`，若显示一系列版本信息则安装成功。

## 初始化与使用

安装 Hexo 完成后，执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

```bash
mkdir -p /www/hexo

hexo init /www/hexo

cd /www/hexo
```

关于文件的说明，查看本文开始的教程链接
比较重要的是配置文件`_config.yml`

### 第一次运行

输入`hexo generate`命令来生成静态文件。

- 生成的静态文件存放在public文件夹中
- 此命令可简写为`hexo g`

然后输入`hexo server`(可简写为`hexo s`)前台启动服务
默认情况下，端口号为`4000`，结束访问按`Ctrl+C`

所以服务器启动成功后，在浏览器中输入`http://IP:4000`，即可看到第一次运行的状况

## 安装并配置Nginx

安装Nginx主要是想让网站能随时随地访问

### 安装 Nginx

```bash
yum -y install nginx
```

### 启动Nginx

```bash
systemctl start nginx
```

此时，用浏览器访问`IP`便可以看到 Nginx 的测试页面，即表示Nginx启动成功！

### 使Nginx开机自动启动

```bash
systemctl enable nginx
```

### 配置服务器访问路径

`vi /etc/nginx/nginx.conf`
将默认的 `root /usr/share/nginx/html` 修改为: `root /www/hexo/public`

确定是否修改正确

```bash
nginx -t
# 如果正确会输出下面两行

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### 重启Nginx

```bash
systemctl restart nginx
```

此时再次访问你的IP地址，应该显示hexo初次运行的样子
如果出现错误，根据本文开始提到的链接进行修改

## 配置Hexo

### 站点配置

```yml
# Site
title: QIMING.INFO #网站标题
subtitle:          #网站副标题
description:       #网站描述，可以是你喜欢的一句话
keywords:          #网站关键词  
author: Qiming     #你的名字
language: zh-CN    #网站使用的语言
timezone:          #网站时区，默认使用服务器的时区
```

其余的文件内容可以使用默认值，萌新可不必修改，具体参见[官方文档](https://hexo.io/zh-cn/docs/configuration.html)

注：修改yml文件时应注意“:”后面应加空格

### 切换主题

将你喜欢的主题，下载到themes文件夹，然后修改站点配置文件`_config.yml`里的`theme`设定即可
例如next主题：

```yml
theme: next
```

#### 主题的配置

每个主题也会有一个`_config.yml`文件，在此主题的根目录下。

我使用的NexT主题有着丰富详细的使用说明，其内置了很多实用功能如第三方评论、文章阅读次数统计、动态背景、站内搜索等，具体设置可参考其[官方使用文档](http://theme-next.iissnan.com/)

## 开始使用

### 创建文章

你可以执行下列命令来创建一篇新文章。

```bash
hexo new post <title>
```

注：post可以不写，因为其为layout的默认值，其余layout还有page（页面）和draft（草稿）。

之后，你的`source/_post`文件夹下会产生一个`<title>.md`的文件。

### 编辑文章

#### Front-matter

打开此`<title>.md`后你会发现在最上方有`以---分隔的区域`，这部分即称为`Front-matter`，用于注定个别文件的变量
例如：

```yml
title: 阿里云CentOS下Hexo+Nginx建站过程 #标题
tags:                      #标签
- HEXO
- 建站
- nginx
- nodejs
- 阿里云
categories:                #分类（注：不支持同级并列分类）
- 教程
- Hexo
date: 2018-04-20 15:52:08  #文件建立时间
```

### 写作

Hexo仅支持Markdown语法，我是使用的 `VSCode + markdown 插件`，写好后，粘贴到<title>.md里

### 运行

写完文章后，在你的博客根目录下输入：

```bash
cd /www/hexo

hexo g
```

hexo就会将写好的文章生成静态网页并存放在`public`文件夹下

---

如果nginx配置正常的话，你现在已经可以通过访问`IP地址`来看到你的博客了！

注：hexo里还有两个常用命令没有讲到，分别是`hexo clean`和`hexo deploy`（可简写为`hexo d`）：

- clean：清除缓存文件 (db.json) 和已生成的静态文件 (public文件夹)。在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。
- deploy：用于部署生成的静态文件(public文件夹)，本文中，因为配置了nginx服务器的访问地址直接指向了public文件夹，所以用不到此命令。

## 开启 https

作为一个ITer，自然要追求更好的效果。不开启 https 也不会影响网站的使用，但是会有安全警告。

### 获取 https 证书

现在国内的云服务商，都会提供免费的https证书，我使用的是阿里云。免费证书发放很快，只需要几分钟验证即可。

### 上传证书到服务器

在服务器上创建一个文件夹，用于存放证书文件。并将两个文件上传至服务器

```bash
mkdir /www/ssl

ls /www/ssl
    cert-....crt
    cert-....crt
```

### 证书安装

`vi /etc/nginx/nginx.conf`
可通过 `nginx -t` 命令查看配置文件位置。在 `listen 446 的 server` 后面在添加如下：

```cnf
server {
    listen       443 ssl http2 default_server;
    listen       [::]:443 ssl http2 default_server;
    server_name  lixyz.net;
    root         /www/hexo/public;

    ssl_certificate "/www/ssl/cert-...crt";
    ssl_certificate_key "/www/ssl/cert-...key";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
```

配置完成后，使用 `nginx -t` 命令检测是否有误


### 设置 http 自动跳转到 https

Nginx 支持 rewrite，编辑 Nginx 的配置文件，在 listen 80 的 server 中添加语句，如下：

```cnf
server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  lixyz.net;
    root         /www/hexo/public;

    rewrite ^(.*) https://lixyz.net$1 permanent;   # 添加的语句

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;
```

配置完成后，使用 `nginx -t` 命令检测是否有误

### 重启nginx

```bash
systemctl restart nginx
```

使用带 https 的域名进行访问即可
