linux什么都比windows好，除了游戏少这一点

不过其实一般玩的游戏也就那么几个，能支持了就满足了

从16年起就一直在玩守望先锋，中间有几次都想着在Linux上装一下，结果没搞成

一是由于当时用linux用的还比较少，很多地方都不太会配置

二来也是因为当时水平比较菜，各种官方文档看起来都费劲

现在好歹也是个3年的Linux User了，于是又想起来想着搞一搞，还真就让我给装成了。



## 显卡驱动

首先，想打游戏肯定离不开显卡驱动

(然而Nvidia的显卡驱动在Linux下面就是个天坑)

之前无数次翻车都是在显卡驱动上

而且之前每一次出问题，由于不会修又只能重装系统

直到后来从头撸了一遍Arch WIki，对linux安装的过程也有了点了解

其实和写代码一样的，哪怕图形界面崩了拿个U盘看着log就能debug

慢慢修总能修好的，不至于每次都重装

废话不多说了，具体安装驱动的方法可以回顾[这篇博客](http://river-li.me/2018/12/07/nvidia/)

记得一定要看到最后，因为有一个包需要删掉(bbswitch)



## 相关软件包

首先，想在linux上玩游戏，有一款软件是一定不能错过的，那就是**Lutris**

Lutris可以管理linux下面各种平台的游戏，都可以在这里面运行

安装起来也是十分的方便;

安装好lutris之后直接在官网搜索想玩的游戏点install就可以了

有一个需要注意的点，如果使用了向我之前那样安装驱动的方法,那么在需要显卡运行程序时必须要有primusrun启动的

在安装好守望先锋这样的游戏之后需要在System Options里设置

![lutris_20190702004159](https://static.hack1s.fun/images/2021/02/24/lutris_20190702004159.png)

同时，要发挥出显卡的性能，还需要安装DXVK

但是DXVK支持的GPU一般都有一点久，最新的nvidia GPU有可能不支持，可以在lutris的github里的wiki看一看自己的GPU是否支持

安装DXVK时可以使用pacman

```bash
sudo pacman -S dxvk-bin
```

安装完成后需要运行

```bash
setup-dxvk
```

来安装，之后在lutris里对游戏设置相应的dxvk版本即可

![lutris_20190702004621](https://static.hack1s.fun/images/2021/02/24/lutris_20190702004621.png)

注意在安装时选择的winee的版本，最好按照官方的推荐，不然大概率运行不了



wine的版本管理也可以直接在lutris中完成



## 运行

这些都设置完成之后就可以启动一下看一看能不能运行

同时也可以开一个终端运行指令

```bash
nvidia-smi
```

来查看GPU的状态，如果成功在显卡跑起来会有相应的进程的

![terminal_20190702005130](https://static.hack1s.fun/images/2021/02/24/terminal_20190702005130.png)

最后放一张运行lutris里库的游戏图

![lutris_20190702005257](https://static.hack1s.fun/images/2021/02/24/lutris_20190702005257.png)

没错，我又开始下载APEX来玩了

这么多游戏可以玩，还要windows有啥用

嘻嘻