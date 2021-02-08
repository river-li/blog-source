用IDA反编译遇到的一些问题和解决方案

<!--more-->

## 报错Decompile Error

提示为某一个地址analysis failed

![image-20201106111005929](https://static.hack1s.fun/images/2021/02/08/image-20201106111005929.png)

找到这个对应的地址，发现是一个`call scanf`

可能是这个`scanf`的参数解析出现了错误

在scanf上按Y重新定义

写上正确的定义

![image-20201106111142655](https://static.hack1s.fun/images/2021/02/08/image-20201106111142655.png)

之后就可以反编译了



## positive sp value has been found

遇到的提示是这个地址的sp为正值

![image-20210105193807432](https://static.hack1s.fun/images/2021/02/08/image-20210105193807432.png)



解决办法：

首先在option中打开stack pointer

![image-20210105194236271](https://static.hack1s.fun/images/2021/02/08/image-20210105194236271.png)

之后可以看到函数中显示出了一个指示当前栈帧的值

![image-20210105194346031](https://static.hack1s.fun/images/2021/02/08/image-20210105194346031.png)

这是原本出现问题的地方值应该是一个负数，在这个add esp的位置按option+k

之后设置这里的sp值，最终使retn这个位置的绿色数字为00即可

![image-20210105194458736](https://static.hack1s.fun/images/2021/02/08/image-20210105194458736.png)

