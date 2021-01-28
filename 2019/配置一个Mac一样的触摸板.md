对于学计算机的人来说,Macbook最大的吸引力主要有两点

- 好用的触摸板手势
- 正统Unix

Unix就不说了,用起来肯定是比Windows舒服的

触摸板手势这一点主要倒是能够在Coding的时候不用频繁离开键盘去拿鼠标

类似的道理ThinkPad的小红点也有不少人喜欢

不过显然还是干不过MacBook的手势,毕竟确实很方便

考研复习中又想摸鱼了,就把Linux的触摸板搞了一下

今天就要在Linux上配置一个比Mac还要好用的触摸板


<!--more-->


## 安装包

首先要解决触摸板驱动

```bash
sudo pacman -S libinput xdotool xinput xf86-input-libinput libinput-gestures
```

上面的指令一连串的把配置触摸板的包也装了

主要就是`xdotool`和`libinput-gestures`

---

之后用`xinput`来看一下能不能找到触摸板

```bash
xinput list
```

![1.png][1]

在这个图里还显示了键盘的设备,还有连的一个微软的无线鼠标

从上往下第二个就是触摸板

找到触摸板的话就说明可以用了

之后来看一下参数

```bash
xinput list-props "ELAN1010:00 04F3:3012 Touchpad"
```

![2.png][2]

这里面就是触摸板的参数了

主要就是需要关注一下下面的`Device Node`

这个的值是设备在`/dev/`目录下的编号,调试的时候要用

众所周知,`Linux`下万物皆文件


## 配置手势

配置手势主要原理是相应的触摸板动作映射成为键盘上的键

这里用的是`xdotool`

在`libinput-gestures`装完后会多出来一个文件`/etc/lib-input-gestures.conf`

当然,这个是系统的配置

如果要对用户个人进行配置,可以在`~/.config/`下面也创建一个这个文件

下面是我的配置:

```
gesture swipe up 3  xdotool key ctrl+F9
gesture swipe down 3  xdotool key super+d

gesture swipe up 4  xdotool key ctrl+F10
gesture swipe down 4  xdotool key super+d

gesture swipe left	3   xdotool key super+Right
gesture swipe right	3   xdotool key super+Left
gesture swipe left	4   xdotool key super+Right
gesture swipe right	4   xdotool key super+Left

gesture pinch in	xdotool key ctrl+minus
gesture pinch out	xdotool key ctrl+plus

```

这里呢,`swipe`是轻扫,`pinch`是捏和

我主要做了这么几件事情:

- 设置四指和三指左右扫切换桌面
- 设置二指捏和进行缩放
- 四指向上显示所有桌面的窗口
- 三指向上显示当前桌面的窗口
- 三指和四指向下显示桌面

本来是三指四指设置了一些不同的功能,但是不知道为什么,识别率有一点低

可能是触摸板本身的问题,因此就直接全部映射成相同的功能了

至于后面的键位,每个桌面环境可能不一样

本身我用的是KDE,不过对这些键又重新设置过

如果你也是用的KDE,可以在Settings里面看一下Shortcut

下面设置自启动

```bash
libinput-gestures autostart
```

之后登出重新进入桌面即可


[1]: http://42.193.111.59/usr/uploads/2021/01/616959750.png#vwid=782&vhei=327
[2]: http://42.193.111.59/usr/uploads/2021/01/1985722264.png#vwid=1479&vhei=675