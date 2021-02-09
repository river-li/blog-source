最近因为看了[友崎君](https://www.bilibili.com/bangumi/media/md28231807/)，比较想玩attafami（neta大乱斗）

就找了找资源，结果发现软件版本太新了，switch主机系统跟不上

就只好更新一下switch的系统；

为了防止以后更新找不到靠谱的教程翻车，这里记一下流程

<!--more-->

更新系统主要分两个部分

1. 更新SXOS
2. 更新Switch主机系统



## 更新SXOS

这一步很简单，只需要替换SD卡根目录的`boot.dat`就可以，不过要注意做好备份；

这个`boot.dat`可以在TX的官网下载，也可以在这个人的[博客](https://shipengliang.com/games/sx-os-tx-os-下载.html), 更新的还是蛮勤快的；



## 更新主机系统

在这个人的[博客](https://shipengliang.com/games/switch-firmware-固件下载.html)下载最新的固件包，一般是选择大白兔的固件包；

没有用过NSP格式的安装，不知道会不会成功，一般其他人都是用的双系统更新，单系统实惨；一直感觉是被电玩店的老板坑了，应该装个双系统的，唉。



下载了固件包之后解压得到文件夹，放到SD卡里面

用Mac拷贝进去会发现文件夹在switch里面没办法正常解析，网上查了一些资料后发现是因为Mac的目录会包含一些奇怪的索引文件，导致switch的shell没办法正确识别；

在拷贝完之后需要执行一下下面这个命令

```bash
sudo chflags -R arch /Volumes/SDCARD_NAME
```



之后在switch上启动大白兔插件，进入要更新的固件目录之后选择右下角的choose

![image-20210209183338844](https://static.hack1s.fun/images/2021/02/09/image-20210209183338844.png)

之后根据SD卡的分区格式选择，我的SD卡是exFat的格式，所以就选exFat

![image-20210209183438431](https://static.hack1s.fun/images/2021/02/09/image-20210209183438431.png)

之后插件就会对固件进行加载，加载完成后选择右下角的select firmware

![image-20210209183559217](https://static.hack1s.fun/images/2021/02/09/image-20210209183559217.png)

硬破设备这里在安装之前要确定AutoRCM这个选项是关的，之后start installation

（我安装的时候这里不需要点击这个选项，跟着默认直接走就可以）

安装完成之后reboot就可以了

![image-20210209183844968](https://static.hack1s.fun/images/2021/02/09/image-20210209183844968.png)