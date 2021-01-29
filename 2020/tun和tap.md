在使用qemu的系统模式来模拟嵌入式设备的固件时，一般需要专门为虚拟机来配置网络，这使用到的功能就是Linux中比较基本的虚拟网络设备相关的功能，即tun和tap

<!--more-->



## 介绍

tun和tap相当于是linux上创建出来的虚拟网络设备，使用软件的方式模拟出来实现硬件的功能，在创建虚拟机的时候，一般VMM会自动创建出新的虚拟网卡，就是利用了这样的功能。

tun是工作在第三层的网络层设备，而tap是工作在第二层的链路层设备

tun表示虚拟的是点对点设备，tap虚拟的是以太网设备

在创建了tap或tun之后，在`/dev/net/`目录下的`tun`文件就会处于激活状态，这时就可以通过读写这个设备，实现内核空间和用户空间的数据交互；这中交互方式也是linux的一种设计思想（万物皆文件）

创建tun/tap可以使用tunctl或者ip tuntap这两种方式



## tunctl

tunctl不是系统自带的，首先需要安装相关的包

```bash
apt-get install uml-utilities
```

创建的虚拟网卡设备上的操作，不会影响到物理设备，也可以说通过这种方式实现了一种隔离，因此VPS、云服务等等一般都是用这些来实现的；

```bash
tunctl -t tap0 -u `whoami`
```

创建设备，并将其分配给了当前的用户

为这个设备分配IP地址并启动

```bash
ifconfig tap0 192.168.0.1/24 up
```

如果要删除这个设备，同样可以使用`tunctl`

```bash
tunctl -d tap0
```



## ip tuntap

ip tuntap可以创建tap设备，也可以创建tun设备

```bash
ip tuntap add dev tap0 mod tap
ip tuntap add dev tun0 mod tun
```

两者差异只在于后面的mod指定设备类型

需要删除设备时使用

```bash
ip tuntap del dev tap0 mod tap
ip tuntap del dev tun0 mod tun
```

在需要指定设备的ip地址时，同样可以使用ifconfig

或者可以使用ip命令中的address

```bash
ip address add dev tap0 192.168.2.1/24
```



