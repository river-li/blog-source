---
title: zsh
date: 2018-12-09 17:26:38
tags:
- linux
copyright: true
---

之前就一直知道zsh，但是一直懒得弄，今天突然找点事做配好了zsh

不得不说zsh实在是太强了

<!--more--->

### 安装

arch系直接`pacman -S oh-my-zsh`

装好之后会有一个提示，按照提示执行：

> cp /usr/share/oh-my-zsh/zshrc ~/.zshrc

完了之后oh-my-zsh就装好了



deepin和debian系使用`sudo apt install zsh`

之后还需要`wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh`



不过想要切换默认的shell为zsh还需要执行这个命令

`chsh -s /bin/zsh`



这时候在term里面输入`zsh`就能进入zsh的窗口里

想要用bash替代zsh还需要修改终端的默认指令，不过这个依图形界面而异，就不统一说明方法了

一般就是在终端的配置文件那里修改bash为zsh即可



### 优化

首先是zsh的主题

在zshrc里面修改里面的theme就可以

例如我改成了random，之后每一次启动zsh就会得到不同的特效主题



在plugins那一部分里面，默认有git这个插件，有一些比较好用的也最好加上

我现在装的有

- git
- extract
- autojump
- bat
- zsh-autosuggestion

后面三个是需要再单独安装包的

分别`pacman -S`

另外最后需要在zshrc中加

```zsh
source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
```



#### debian系

配置autojump需要执行一句

`sudo apt install autojump`

之后

`[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh`

之后在zshrc中plugins处加入autojump

---

配置autosuggestion

`git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions`

之后在zshrc中plugins中加入zsh-autosuggestions

----

还可以配置history插件，直接在zshrc中加入即可，作用是在zsh的terminal中输入history后可以出现历史命令

---

bat这个插件除了macOS和arch以外都需要编译安装

```bash
git clone https://github.com/sharkdp/bat.git
```

