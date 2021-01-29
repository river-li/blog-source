---
title: VPS
copyright: true
mathjax: true
date: 2020-03-12 02:30:01
tags:
- linux
categories:
---

# VPS

很早以前就一直想整理这么一个东西，整理一下VPS都能做什么

今天在尝试装一个软件的时候VPS出了一些问题，所以重新装了一些常用的软件，也顺便整理一下吧



## Blog

写博客可以说是最基本的功能之一了

在github的gitpage中可以直接用的几个常用静态网页框架都可以直接在VPS上部署博客

复杂一些的还可以用Wordpress，像我就为了方便直接也是用了hexo

一般在重装VPS之后要部署博客都做这么几件事：

1. 修改ssh端口
2. 在root的目录下心间一个dir
3. 将这个dir设置为bare的git repository
4. 将博客clone在网站代理的目录

具体需要

```bash
mkdir blog.git
cd blog.git
git init --bare
vim hooks/post-receive
```

这个文件内容

```bash
#!/bin/sh
unset GIT_DIR
cd /var/www/html
git checkout -f
```

由于我的博客是一式两份部署在了github page和VPS上，所以直接可以这样做

这样每次更新博客，这个repo收到deploy的要求之后就会自动执行git hooks去相应的目录更新网站内容



## Anki Server

Anki是一个记忆的软件，可以规划好时间背单词，会自动按照记忆曲线分配每天需要背的任务，Anki也有非常强的功能，可以用CSS、html等内容定制模板

并且是一个全平台的软件，所以一个比较好的学习方式就是在电脑上学习时制作卡片，然后每天用手机来学习；

但是这个软件有个问题就是服务器在美国，而且由于人家是非盈利的，就导致国内速度特别慢；

而有一个VPS就可以在自己的VPS上部署服务器，这样同步速度完全就不是一个档次了

安装方式

```bash
pip install AnkiServer
ankiserverctl.py adduser <username>
ankiserverctl.py start
```

官方的说明中说使用nginx还需要特别的配置一下转发

这里就略去了

## Proxy

作代理也是一个很重要的功能，国内访问一些网站速度经常比较慢，用VPS访问会快很多，VPS上可以部署ss和v2ray之类的工具

这些就不说怎么安装了，一键脚本多的是

最近我还发现了一个有趣的工具，bbr

它甚至直接给系统换了一个内核，使得网络质量比较差的情况下网速有很大的提升；

```bash
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```

直接运行这个指令，然后按照提示安装就可以了，实测确实还是有些效果的



## Web Server

Web Server主要可以选择apache或者nginx

网上相关的配置资料也有很多；一般我选择的小型便宜VPS就直接用nginx了，相对可以快一些

安装十分简单

```bash
apt install nginx
service nginx start
```

有时候也需要修改一些配置，一般改一下访问权限什么的

最多也就改一改`site-available`



## Frp

用frp可以将内网的机器某个端口映射到服务器的具体某个端口处

不过如果把Windows的3389之类的映射到公网IP，那会有很大概率被攻击

所以这种情况需要设置一个强密码

另外之前也了解到了一些辅助的安全手段，比如`port knocking`

就是在请求之前必须先发一个特定的包，之后才会进入允许握手的状态

相关资料：

- [https://en.wikipedia.org/wiki/Port_knocking](https://en.wikipedia.org/wiki/Port_knocking)
- [https://wiki.archlinux.org/index.php/Port_knocking](https://wiki.archlinux.org/index.php/Port_knocking)

这种手段也可以用在SSH上



## Jupyterlab

毕竟VPS是一台全天24小时运行的机器，拿来运行爬虫之类的肯定比自己的工作机好得多，那么配置一个Python的环境就比较重要了

在学机器学习的时候用了一段时间jupyter，感觉jupyter lab比notebook好用一些，一般就配置这个；

安装方法：

```bash
pip3 install jupyterlab jupyter
jupyter lab --generate-config
```

之后编辑配置文件，主要要设置一下password

password可以用python运行得到

```bash
python -c "from notebook.auth import passwd;passwd()"
```

最后运行就可以了

```bash
nohup jupyter lab --no-browser &
```



## 网盘

待续