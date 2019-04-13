---
title: Linux 命令记录
tags:                      
- 随笔
categories:                
- 随笔
- 教程
date: 2018-11-7
---

# screen

screen是一个很好用的窗口管理工具，具体的细节不关心。
对我而言，最大的用处是为了防止后台突然断开连接。
比如下面的情况，

- 正在源码编译MySQL，但是需要断开连接，或者担心连接不稳定。
- 客户端由于长时间没有活动导致断开。
- 单纯的想让一个进程后台执行，同时可以随意地调回前台。

这些，都可以通过`screen`命令做到

```bash
screen 
```

直接输入命令，会自动进入screen环境，但是不推荐使用。
因为进入后台的命令，所以显示的名称无法分辨。

如果需要自定义后台显示的名称，需要使用 `-S name`
比如添加一个正在编译MySQL的后台进程

```bash
screen -S compile_mysql
```

但是这样也不好用，因为没有日志。
没有日志，还当什么运维啊，
`-L`会在当前目录下，添加日志，自动保存屏幕上的输出，名称为 `screenN.log`

```bash
screen -L -S compile_mysql
```

那么如何后台呢，
让前台screen进程进入后台，需要使用键盘上的组合键
按住`ctrl + A`不要松开，再按住 `D`，即可后台
此时进程进入`[detached]`状态

查看后台screen进程，并根据`ID`调回指定的进程

```bash
$ screen -ls
    There is a screen on:
        5721.compile_mysql  (Detached)
    1 Socket in /var/run/screen/S-root.

$ screen -r 5721
```

如果需要退出，使用`exit`即可
