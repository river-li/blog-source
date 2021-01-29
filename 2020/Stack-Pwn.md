学习到的栈相关的内容


<!--more-->


## 栈迁移

首先，在栈溢出发生时栈的一个基本结构是比较简单的

![image-20201119101751227.png][1]

如果我们可以溢出的长度比较有限，只能控制ebp以及return address这两个地方

那么就可以考虑使用栈迁移这样一个技巧

核心有两个要求：

1. 能够在内存可执行区域写入一定长度的代码；
2. 能够控制ebp以及return address



执行栈迁移时，控制ebp为我们控制的一个地址，return address则填写一个`leave;ret;`的gadget地址

leave指令相当于`mov esp,ebp;pop ebp`

所以整个gadget相当于执行了`mov esp,ebp;pop ebp;ret`

这样一个gadget实际上做了什么事情呢？

![image-20201119103212438.png][2]

首先`mov esp,ebp`这个指令清空了子函数的栈，esp和ebp现在都指向old ebp的位置

之后`pop ebp`将ebp的值设置为栈中保存地址

最后要执行`ret`跳转到栈中保存的返回地址处



如果我们这里控制old ebp为一个其他段，例如bss段中的地址，ret addr中是这个gadget的地址

在函数第一次执行到返回这样的一个过程中，我们将ebp成功放到了bss段上，但是其实esp还没有被控制

接下来执行到ret这个地方时又执行了一遍`ret addr`这里保存的gadget

第二次的这个过程就让esp变成了我们填写的old ebp的位置的值



整个这样一个过程，我们通过控制的这样两个值，让esp指向了我们需要的一个位置

同时ebp其实也是可以控制其中的值的，但是因为它对我们shellcode的执行并没有影响，所以可以随便设置


[1]: http://42.193.111.59/usr/uploads/2021/01/2029726206.png#vwid=882&vhei=894
[2]: http://42.193.111.59/usr/uploads/2021/01/3146744373.png#vwid=1970&vhei=986