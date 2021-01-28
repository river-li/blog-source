分析的内容为Linux中ip-utils中的ping
为网络程序设计这门课要求的小作业


<!--more-->


## 获取源码

首先要知道ping的程序在哪个包里
在命令行使用`man ping`查看手册
最下面会写着来自于iputils
百度找到源码
与ping相关的文件主要有:

- ping.c 
- ping_common.c
- ping_common.h

程序的main函数位于ping.c，从这里开始分析

## main

![48.png][1]
Main函数非常的长，前面设置了许多属性，还有很多if判断语句用于判断设置的参数

除了定义的内容，可以看到设置了capabilities以及locale

之后下面建立了用于ping程序的icmp的socket
在这下面判断socket是否建立成功，并且获取到errno

这下面可以看到一个while循环，不停的调用getopt获取命令行参数

下面使用switch语句对相应的参数进行设置


![0.png][2]

在最下面的default中调用了usage函数，其中内容如下：


![1.png][3]

在switch语句之后，是对参数有效性进行检查，对于有问题的部分调用usage给用户

![2.png][4]
下面的一个while循环比较重要，类似我们课上例子的ping程序的主函数部分

![3.png][5]
这里可以看到，如果argv是ip地址则，hostname=target，否则（即是域名）调用gethostbyname函数获取实际的ip地址

这下面是一些检测

![4.png][6]

这里最初初始化时是全0，所以第一次调用时一定会执行，查看内容可以看到



![5.png][7]


![6.png][8]
比较核心的地方在这个connect，根据报错的信息可以知道这里的目的是首先尝试创建一个socket并connect到目标主机，如果是广播地址就报错，或向用户确认是否要ping广播地址


这里的一部分判断本机地址是否是0.0.0.0

![7.png][9]

这个检测目标地址是否为0，如果是0.0.0.0则将目标地址设置为本机的地址，变成ping 127.0.0.1

下面是检测icmp_sock是否建立成功

![8.png][10]

如果出错则退出

这个地方我们往上看可以看到之前也检测过一次icmp_sock是否小于0，但那时的解决方式是重试了一次，这里出错则直接退出。上面的判断：

![9.png][11]

下面device是加了-i参数选择出口网卡interface时的内容，之后对msg作了一些设置


![10.png][12]


![11.png][13]

这里针对ping广播地址的情况进行一些判断

interval太小则会直接退出，这个值是两个包间间隔的时间长短

pmtudisc这个值是根据ping –M参数后的内容设置的，在ping广播的情况下-M后的值必须是do，不能是don’t和want


![12.png][14]

这里对using_ping_socket是否为0的两种情况

可以看到之前一次创建socket如果失败，则会将using_ping_socket的值设置为1

因此using_ping_socket的情况是创建socket失败一次的情况下对socket设置了几个option


![13.png][15]

这里是针对参数-R时记录路由设置参数

![14.png][16]

这里对-T设置参数，-T作用是设置timestamp

![15.png][17]

这个地方的作用是设置没有设置-R –T时的参数
可以查看设置这个options 的地方

![16.png][18]

![17.png][19]

这里根据注释可以得知作用是粗略估计数据包的长度

之后用sock_setbufs设置socket的buf大小

再下面检测了一些参数以及数据包长度；

整个程序的主要功能在最下面的嵌套中，从main函数的底部看起


![18.png][20]

可以看到底部printf打印了运行ping时常常看到的信息

之后调用了setup函数以及main_loop
Setup和main_loop函数定义在ping_common.c中

Setup函数的定义：


![19.png][21]

根据注释可以得知，这个函数的作用是设置并检查参数

跳过前面一部分检测的内容

![20.png][22]

这里设置了SNDTIMEO来防止当网卡接口很慢或停止时永久阻塞，将小于1000的时间间隔统一设置为了1000


![21.png][23]

设置RCVTIMEO优化避免冗余轮询

![22.png][24]
设置ident这个值用来标识主机，使用的是进程的pid的后16位

![23.png][25]
下面三句注册了三个信号，绑定了对应的函数

之后是设置信号集set初始化并清空，之后改变信号屏蔽字

![24.png][26]

deadline是设置的-w参数后面的值，即超时时间

itimerval是一个定时器

---

## main_loop

下面看main_loop的定义


![25.png][27]
整个函数的核心内容是这个for循环，for循环前面判断退出条件

在这下面是一个比较重要的部分

![26.png][28]
在一个循环中调用pinger函数

这个pinger函数是真正用于发送icmp包的地方

这里先放着继续向下看

![27.png][29]
折叠了if后的内容后可以看到下面又进入了一个for循环

这里的for循环的作用是接受icmp的回显包

上图中可以看到调用parse_reply，这个函数是用来解析处理回显包的

其中内容主要是设置了recev的相关内容以及调用函数recvmsg

![28.png][30]
前面学习socket时了解到recvmsg是用于接受消息队列中消息的函数，这里的作用就是从socket中读取一个msg

---

## pinger

在pinger函数前面有一段注释：

![29.png][31]
可以看到，pinger函数的主要作用是compose 并transmit ICMP的回显包

下面就参考着注释分析一下pinger函数的内部

![30.png][32]
首先判断是否到达了退出条件，如果达到则返回1000

![31.png][33]
这个条件主要目的是设置tokens和cur_time

这里再往下直到pinger函数结束都是resend这个部分

![32.png][34]
resend这里首先就调用了send_probe函数，之后对其返回值进行判断，根据不同返回值调用不同的部分，最终如果前面出错则调用一些处理，返回。

因此我们可以判断出这一部分中实际的发送函数是send_probe,正常返回0，其他情况出错
下面进入这个函数

## send_probe

![33.png][35]

最主要的是send_to和那个sendmsg

如果使用的是cmsg，则设置数据包并调用sendmsg发送数据

否则直接使用send_to向socket发送数据
最后判断返回值是否与len相等，相等则返回0，否则返回cc即send的返回值

为了进一步了解这个len的值是如何得到的，我们进入build_niquery和build_echo两个函数看看

![34.png][36]
可以看出这个build_niquery函数是构造了一个请求包，并用cc这个值记录它的实际大小
这个ni_hdr数据结构内部：

![35.png][37]
得知这一部分是处理icmp6数据包的内容

这个结构体的大小也应该是发送的请求icmp6的数据包大小

![36.png][38]
这个是创建了一个回显包

计算并最终返回回显包的长度

对于pinger的分析到此结束

---

## parse_reply

![37.png][39]
首先看到注释

这个函数的作用是分析获取到的icmp包，判断是否属于自己，是的话则将其打印

这个判断的原因是ICMP的接口会受到所有的回显包，有可能同一台主机上有多个用户在使用ping程序

![38.png][40]
这里的内容是检查ip头部

这里主要是设置了一些属性，判断的地方只有ip包的长度，如果数据包长度过短则退出

下面检测icmp部分

![39.png][41]
可以看到，调用了一个in_cksum函数

看名字即可看出这个函数是用来求校验和的

---

## in_cksum

![40.png][42]
查看注释，不难理解算法

使用了一个32位的累加器，将16位的字加上去，最终将高16位和低16位相加

最终结果需要取反

Icmp的校验和是icmp报文中所有16位字段的补码的总和的16位补码

![41.png][43]
下面这里是当得到的报文不是icmp_echoreply类型时进入

出现这种情况，有可能是icmp包本身类型不对，也有可能是原本的icmp包在传输中出错
下面的一个大括号用来处理

![42.png][44]
长度出错

![43.png][45]
类型出错，或这个包并不是发给我们的，即收到了别人的回显包

![44.png][46]
输出icmp包的序列号以及来源的地址

如果csfailed，即在求校验和时出错则输出bad checksum

![45.png][47]
看到default这里的注释是Must Not，原因是icmp包类型与上面的几个都不同，则一定是错误

![46.png][48]

下面这里是正常时继续运行的内容，输出了受到包的时间，以及来自的主机ip
最终下面作了两个检测，正常退出

![47.png][49]

到此整个ping程序中比较重要的部分就分析完毕了


[1]: http://42.193.111.59/usr/uploads/2021/01/3953324606.png#vwid=593&vhei=598
[2]: http://42.193.111.59/usr/uploads/2021/01/3772387087.png#vwid=556&vhei=790
[3]: http://42.193.111.59/usr/uploads/2021/01/2291643792.png#vwid=307&vhei=562
[4]: http://42.193.111.59/usr/uploads/2021/01/1288739646.png#vwid=385&vhei=321
[5]: http://42.193.111.59/usr/uploads/2021/01/3590666185.png#vwid=546&vhei=818
[6]: http://42.193.111.59/usr/uploads/2021/01/1331680471.png#vwid=382&vhei=107
[7]: http://42.193.111.59/usr/uploads/2021/01/3785749042.png#vwid=400&vhei=179
[8]: http://42.193.111.59/usr/uploads/2021/01/2716089500.png#vwid=638&vhei=407
[9]: http://42.193.111.59/usr/uploads/2021/01/2599446819.png#vwid=467&vhei=66
[10]: http://42.193.111.59/usr/uploads/2021/01/4266792547.png#vwid=333&vhei=115
[11]: http://42.193.111.59/usr/uploads/2021/01/1653214607.png#vwid=320&vhei=115
[12]: http://42.193.111.59/usr/uploads/2021/01/3916582161.png#vwid=481&vhei=138
[13]: http://42.193.111.59/usr/uploads/2021/01/4181178988.png#vwid=519&vhei=244
[14]: http://42.193.111.59/usr/uploads/2021/01/2069181609.png#vwid=622&vhei=274
[15]: http://42.193.111.59/usr/uploads/2021/01/4038356632.png#vwid=764&vhei=449
[16]: http://42.193.111.59/usr/uploads/2021/01/1410766531.png#vwid=715&vhei=247
[17]: http://42.193.111.59/usr/uploads/2021/01/22535329.png#vwid=730&vhei=395
[18]: http://42.193.111.59/usr/uploads/2021/01/2864666850.png#vwid=705&vhei=334
[19]: http://42.193.111.59/usr/uploads/2021/01/781272352.png#vwid=380&vhei=319
[20]: http://42.193.111.59/usr/uploads/2021/01/873331156.png#vwid=546&vhei=107
[21]: http://42.193.111.59/usr/uploads/2021/01/205607115.png#vwid=885&vhei=506
[22]: http://42.193.111.59/usr/uploads/2021/01/1016905168.png#vwid=532&vhei=151
[23]: http://42.193.111.59/usr/uploads/2021/01/771461873.png#vwid=609&vhei=229
[24]: http://42.193.111.59/usr/uploads/2021/01/788961494.png#vwid=627&vhei=130
[25]: http://42.193.111.59/usr/uploads/2021/01/1480591344.png#vwid=372&vhei=50
[26]: http://42.193.111.59/usr/uploads/2021/01/3417837705.png#vwid=353&vhei=166
[27]: http://42.193.111.59/usr/uploads/2021/01/141924675.png#vwid=478&vhei=396
[28]: http://42.193.111.59/usr/uploads/2021/01/506981012.png#vwid=494&vhei=461
[29]: http://42.193.111.59/usr/uploads/2021/01/973446974.png#vwid=367&vhei=121
[30]: http://42.193.111.59/usr/uploads/2021/01/1984806858.png#vwid=683&vhei=579
[31]: http://42.193.111.59/usr/uploads/2021/01/2544809896.png#vwid=491&vhei=556
[32]: http://42.193.111.59/usr/uploads/2021/01/255242210.png#vwid=710&vhei=343
[33]: http://42.193.111.59/usr/uploads/2021/01/3790183487.png#vwid=641&vhei=57
[34]: http://42.193.111.59/usr/uploads/2021/01/1198755591.png#vwid=497&vhei=498
[35]: http://42.193.111.59/usr/uploads/2021/01/842686239.png#vwid=489&vhei=491
[36]: http://42.193.111.59/usr/uploads/2021/01/3811300905.png#vwid=491&vhei=688
[37]: http://42.193.111.59/usr/uploads/2021/01/4034327289.png#vwid=466&vhei=404
[38]: http://42.193.111.59/usr/uploads/2021/01/466806600.png#vwid=419&vhei=109
[39]: http://42.193.111.59/usr/uploads/2021/01/3700657264.png#vwid=469&vhei=393
[40]: http://42.193.111.59/usr/uploads/2021/01/1122014360.png#vwid=674&vhei=420
[41]: http://42.193.111.59/usr/uploads/2021/01/2301990359.png#vwid=664&vhei=604
[42]: http://42.193.111.59/usr/uploads/2021/01/988926756.png#vwid=600&vhei=727
[43]: http://42.193.111.59/usr/uploads/2021/01/3179331955.png#vwid=752&vhei=604
[44]: http://42.193.111.59/usr/uploads/2021/01/3385559757.png#vwid=729&vhei=623
[45]: http://42.193.111.59/usr/uploads/2021/01/194053504.png#vwid=388&vhei=81
[46]: http://42.193.111.59/usr/uploads/2021/01/340032525.png#vwid=363&vhei=73
[47]: http://42.193.111.59/usr/uploads/2021/01/2540102689.png#vwid=545&vhei=161
[48]: http://42.193.111.59/usr/uploads/2021/01/798301189.png#vwid=219&vhei=101
[49]: http://42.193.111.59/usr/uploads/2021/01/1936924813.png#vwid=743&vhei=286