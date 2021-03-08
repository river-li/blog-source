2020年S&P上的一篇文章阅读笔记

KARONTE: Detecting Insecure Multi-binary Interactions in Embedded Firmware

论文看完了，笔记还没整理完:cry:

## 摘要

IoT设备的安全分析主要面临的几个难点包括：

1. IoT系统上的软件功能多是hardware-dependent，并且在执行时需要复杂的配置，这导致动态分析技术非常的艰难；
2. 很多功能并不是在一个Binary中实现的，大多是多个二进制程序共同实现一个功能，这导致现有的静态分析技术很难跨二进制分析其中传播的数据；

这篇文章提出了一个用于静态分析固件的工具，KARONTE

> a static analysis approach capable of analyzing embedded-device firmware by modeling and tracking multi-binary interactions.

利用静态污点的方法对二进制间的交互、信息传播进行监控并进一步分析；

作者团队对53个固件样本进行测试，成功检测出了46个0day

之后对899个各种大小、各种复杂度的样本做了大规模的测试，表明了KARONTE的有效性。


<!--more-->


## Introduction

设想一个例子：

一个web server程序接受来自用户的16字节输入，之后传给了一个handler binary，这个二进制程序又将输入复制了两次，其中一份传给了另一个handler binary。而这个最后接受输入的二进制程序没有对这个缓冲区长度进行检查；

这样在使用到针对单个二进制程序的分析工具分析时，就有可能将第三层的二进制文件的问题标为positive，但是实际上前面传过来的程序已经经过了长度检查，就导致假阳性非常的高；而这每一个假阳性的例子都需要安全研究人员花费大量的时间来分析。

另一方面，如果在使用单个二进制静态分析工具分析时我们只关注接受网络输入的程序，这样就没有办法分析到更加深层、更加复杂的漏洞。

因此一个跨二进制的静态分析方法是非常必要的。

为什么不说动态分析的工具呢？现存的动态分析工具有一些是试图通用地对固件进行模拟，但是这样的工作成功率都不是很高，一般在13%-21%；而其他有一些动态分析的工具是尝试直接对某个特定型号的设备进行动态分析，用到了很多动态分析的技巧，包括fuzzing、符号执行、动态污点等等，有的甚至将这些都一起用起来，虽然可能能够达到比较广的测试面，但是就有些显得过于低效了。



这篇文章提到的Karonte是实现了多个二进制的静态分析，可以追踪固件中多个二进制程序之间传输的数据流，并进一步发现漏洞。这个系统的构建基于了这样的一个假设：二进制程序间通信时是使用到了一个特定集合的*Inter-Process Communication（IPC）*范式，并且利用这些范式的共性来检测将用户输入引入到固件中的位置，并进一步确定固件之中不同组件的交互。之后这些交互被标为污点来追踪组建间的数据流，最后污点和约束被用于检测程序的安全薄弱点。

这个系统发现的一些漏洞有的使用之前的静态分析工具也可以发现，但是会有大量的假阳性警报，比较而言Karonte将警告的数量降低到了2:722，也就是Karonte的2个误报，普通的静态工具会有722个误报，大大降低了安全人员分析量。



## Background

### 固件中的IPC

检测用户输入是如何在嵌入式设备固件中传播的是一个公认的难题，并且还存在很高的假阳性。作者团队观察到在实践中，程序是通过一个有限集合的`communication paradigms`，即IPC来实现的。

一个IPC是通过一个特定的*key*来定义的，并且这个key是每一个参与的进程都会需要用到的。因为这个key在通信中的很多程序都需要用到，因此一般在开发时是硬编码进程序的;因此在静态分析时可以追踪这个*key*来判断可能被攻击者控制的输入在程序中的传播方式

文章将IPC分成了几个类型，下面是每一种类型及对应的data key：

- Files，文件名
- Shared Memory，the backing file name
- Environment Variables，环境变量名
- Sockets，sockets's endpoint
- Command Line Arguments，被唤起的程序名

关注的这几个类型都是可以比较大量的传输数据的方式，其他包括信号这样的机制也可以实现进程间的通信，但是问题在于信号无法传输大量的数据，威胁程度就没有那么高。

在一个智能灯泡中用到了Shared Memory这样的机制，并且是使用了静态的内存共享区域



## Approach Overview

系统主要关注的漏洞是内存破坏型漏洞以及DOS漏洞，但是这个系统也可以比较轻易的进行拓展。Karonte对固件进行分析时主要经过了五个步骤

![image-20201113163407553](https://static.hack1s.fun/images/2021/02/24/image-20201113163407553.png)

- Pre-processing

使用binwalk解压固件



- Border Binary Discovery

这一步是对固件中解压了的二进制程序进行分析，自动化的将存在对外接口的二进制程序进行检索。这些Border Binary是攻击者控制的输入能够进入固件内存的一个接口；

对于每一个这样的二进制程序，这个模块会标记程序中可能会由攻击者控制的数据点位置。



- Binary Dependency Graph Recovery

给定一个border binary集合，karonte会生成一个有向图BDG；BDG是通过反复迭代leveraging一个*Communication Paradigm Finder*集合模块，来恢复出来的



- Multi-binary Data-flow Analysis

给定一个BDG中的二进制程序，利用一个静态污点分析引擎来追踪数据是如何在二进制程序间传播的以及这些数据在传播时有哪些约束；之后将这些约束以及数据传播到在BDG中与b程序连接的其他程序



- Insecure Interactions Detection

最后检测insecure的攻击者可能控制的数据有可能触发的危险操作，并进行报告 
