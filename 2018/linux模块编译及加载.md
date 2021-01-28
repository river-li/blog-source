## 前言

大学狗的一门课，操作系统原理及安全留的一项作业，要求自己编译linux内核，在网上找过一些资料后尝试完整编写内核，但是发现有个很麻烦的问题就是内核完整编译每次都需要很长时间，如果之前添加的系统调用出现bug，debug时又要重新编译一遍，非常浪费时间，于是找了一些资料，只编译内核的模块。

<!--more-->


## 准备环境

1. VirtualBox虚拟机 - - - > 百度可以直接下载

2. Parrot Security OS  -  -  -  >  https://www.parrotsec.org/

安装过程虚拟机过程省略

(选择parrot os的原因是电脑上本来就装着, 其他debian系的linux系统同理)



## 编写模块并加载进内核

首先是源代码

这里是一个简单的printk调用，可以在警告中输出信息

hello.c

```C++
#include <linux/init.h>
#include <linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");
static int hello_init(void)
{
printk(KERN_ALERT "Hello enter\n");
return 0;
}
static void hello_exit(void)
{
printk(KERN_ALERT "Hello exit\n");
}
module_init(hello_init);
module_exit(hello_exit);
```

makefile

```
obj-m := hello.o
CURRENT_PATH := $(shell pwd)
LINUX_KERNEL := $(shell uname -r)
LINUX_KERNEL_PATH := /usr/src/linux-headers-$(LINUX_KERNEL)
all:
        $(MAKE) -C $(LINUX_KERNEL_PATH) M=$(CURRENT_PATH) modules
clean:
        rm *.ko
        rm *.o
```

makefile的主要作用是自动化编译的过程，具体作用以及语法可以百度找到很多

在一个新建目录中添加好这两个文件后就可以在该目录下编译了

在此打开终端输入 *make*

没有报错信息则编译成功，会发现当前文件夹中出现了hello.ko hello.o等文件

之后加载模块：*sudo insmod hello.ko*

卸载模块:*sudo rmmod hello*

当加载模块时会执行刚刚文件中的*module_init()*函数,卸载模块时执行*module_exit()*函数

我们自己写的hello程序在加载和卸载时分别会产生内核警告信息，要查看显示的信息：

*sudo dmesg*



可以看到出现了想要的信息

**PS：使用这种方法编译模块便于调试，不用完整的编译内核可以节省很多时间，常常在编写驱动时这样做**