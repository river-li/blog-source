---
title: Heap-Pwn
copyright: true
mathjax: true
date: 2020-9-19 10:02:56
tags:
- linux
- pwn
categories:
- Pwn
---

学pwn过程中学到的堆相关知识

<!--more-->



## 堆基本知识

堆是可以提供动态分配的内存，在程序虚拟地址中是一块连续的线性空间，从低地址向高地址增长，管理堆的程序被称为堆管理器；

在linux pwn里面一般关注的是ptmalloc2，是glibc2.3以后集成的堆管理器。



需要注意的是，在linux中有这样一个基本的内存管理思想：

> **只有当真正访问一个地址的时候，系统才会建立虚拟页面与物理页面的映射关系**

### 堆的基本函数

#### malloc

```c
malloc(size_t n);
```

关于malloc需要注意的是：

1. malloc返回的是对应内存块的指针
2. 当n为0时，返回当前系统允许的堆最小内存块
3. 当n为负数时，由于大多数系统中`size_t`是一个无符号数，所以会分配很大一块空间，但是一般情况下都会失败；



#### free

```c
free(void *p);
```

和malloc可以说是一个对应关系，free用于释放内存块

1. p为空指针时不进行任何操作；
2. p已经被释放后，再次释放会导致乱七八糟的效果(double free)
3. 如果释放的空间很大，一般会直接还给系统以减小程序占用的空间，除非禁用`mallopt`



### 背后的系统调用

malloc函数和free函数是程序猿编程时调用的程序，但是实际上背后运行的并不是这些东西，而是通过更简单的函数系统调用实现的





#### brk

brk函数实际上执行的操作很简单，就是移动堆的指针，有些类似于移动esp的函数；

通过移动栈上的esp就可以让栈增大或缩小，同理移动brk用于控制堆的大小。



![程序内存布局](./program_virtual_address_memory_space.png)

brk函数和start_brk最初指向的地址是一致的，都位于bss段向上一个随机偏移（如果不开启ASLR则是直接在bss段的结尾）

brk通过移动heap的顶端来实现heap的扩容和回收



#### mmap

mmap就是memory map，从名字看也是和内存管理相关的；

mmap的好处在于可以分配私有匿名的内存空间

mmap的可以用来在虚拟内存地址空间中分配地址空间，创建和物理内存的映射关系；

映射关系可以分文件映射和匿名映射两种，文件映射是文件映射进虚拟内存，匿名映射则是一片全0的空间；又可以根据共享方式分为私有映射和共享映射，其中私有映射是这个进程独有的，其他进程无法读写修改，也不会反应到物理内存中；因此总结一下一共存在四种映射。



这里用一个例子来看一下mmap和对应的munmap函数对进程虚拟内存的影响；在linux中可以查看`/proc/pid/maps`来看内存的布局

```c
int main()
{
        int ret = -1;
        printf("Welcome to private anonymous mapping example::PID:%d\n", getpid());
        printf("Before mmap\n");
        getchar();
        char* addr = NULL;
        addr = mmap(NULL, (size_t)132*1024, PROT_READ|PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
        if (addr == MAP_FAILED)
                errExit("mmap");
        printf("After mmap\n");
        getchar();

        /* Unmap mapped region. */
        ret = munmap(addr, (size_t)132*1024);
        if(ret == -1)
                errExit("munmap");
        printf("After munmap\n");
        getchar();
        return 0;
}
```

运行的程序就是这样简单的调用mmap和munmap，可以趁程序在getchar等待输入的时候查看内存布局

![munmap前后内存区域差异](./image-20200923073653910.png)

这里面框出来的两个就是munmap执行前后的差别；

在ctfwiki里面写到运行的结果

```text
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/syscalls$ cat /proc/6067/maps
08048000-08049000 r-xp 00000000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
08049000-0804a000 r--p 00000000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
0804a000-0804b000 rw-p 00001000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
b7e21000-b7e22000 rw-p 00000000 00:00 0
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/syscalls$

sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/syscalls$ cat /proc/6067/maps
08048000-08049000 r-xp 00000000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
08049000-0804a000 r--p 00000000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
0804a000-0804b000 rw-p 00001000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
b7e00000-b7e22000 rw-p 00000000 00:00 0
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/syscalls$
```

这应该是运行在32位系统下的效果，mmap分配的匿名空间在b7开头的地址空间；

而图中测试在64位下，mmap分配的区域在7f开头的内存空间，和加载的so文件比较接近



---

## 堆溢出

### 原理

堆溢出和栈溢出的差异在于，堆上不存在可以直接影响控制流的返回地址，因此一般无法直接通过堆溢出控制EIP

漏洞利用的思路一般是：

1. 寻找堆分配函数
2. 寻找危险函数
3. 确定填充长度



#### 寻找堆分配函数

堆分配函数一般是malloc、calloc、realloc等；

其中calloc在分配内存区域之后会将其置0；

realloc函数则和另外两个有些不同，具有两个参数

```C
realloc(ptr,size);
```

根据ptr和size的不同，realloc函数实际执行的效果也是不同的；

- 当 realloc(ptr,size) 的 size 不等于 ptr 的 size 时

- 如果申请 size > 原来 size

- - 如果 chunk 与 top chunk 相邻，直接扩展这个 chunk 到新 size 大小
        - 如果 chunk 与 top chunk 不相邻，相当于 free(ptr),malloc(new_size) 

- 如果申请 size < 原来 size

- - 如果相差不足以容得下一个最小 chunk(64 位下 32 个字节，32 位下 16 个字节)，则保持不变
        - 如果相差可以容得下一个最小 chunk，则切割原 chunk 为两部分，free 掉后一部分

- 当 realloc(ptr,size) 的 size 等于 0 时，相当于 free(ptr)

- 当 realloc(ptr,size) 的 size 等于 ptr 的 size，不进行任何操作



#### 危险函数

危险函数是可能导致泄漏信息或是溢出的函数

包括：

- gets、scanf、vscanf
- sprintf
- strcpy、strcat、bcopy

等



#### 确定填充长度

malloc函数执行时实际分配的内存和请求的数量未必是相同的；

因为ptmalloc在内存分配时是按照系统2倍最小字长进行对齐的；

也就是说32位系统按照8字节对齐，64位系统按照16字节对齐。



#### 特殊的堆溢出

有时候会出现一个字节的溢出，这一般是边界检查不严格造成的，这一般称为off-by-one漏洞。

off-by-one漏洞的利用思路：

1. 溢出字节为任意字节时可能通过修改下一个块结构的大小导致结构重叠，进而泄漏数据或是覆盖其他块的数据；
2. 溢出字节为NULL字节时，可以使得下一个块的`prev_in_use`位被清零，这样前块被认为是free，这时可以
    1. unlink进行处理；
    2. 或是伪造`prev_size`导致块间重叠；

### 实践

以几道CTF来练一下手



#### Asis CTF 2016 b00k

首先检查程序的安全机制

![](heap-overflow/image-20200926194107247.png)

是一个64位程序，并且除了gs，其他几个安全机制都开启了

运行一下程序，功能是一个图书管理的程序

![image-20200926202511078](heap-overflow/image-20200926202511078.png)

反编译得到main函数的伪代码

![image-20200926195340455](heap-overflow/image-20200926195340455.png)

点开每一个具体的函数之后，可以发现其中读取文字内容时是使用到了程序自己定义的一个新函数

![image-20200927204517434](heap-overflow/image-20200927204517434.png)

我们暂且将其称为`new_read`

![image-20200927204730211](heap-overflow/image-20200927204730211.png)

可以看到这个函数在调用时，第一个参数是指向一个缓冲区的指针；

第二个参数是要读入的长度，减去了一个null byte，这样的读是没有问题的

![image-20200927213357312](heap-overflow/image-20200927213357312.png)

但是对于修改author name的函数，在调用这个read时设置的长度忘记了减一，这里有可能存在一个byte的溢出

设置author name为一个32字节长的内容

之后创建一个book之后，调用第四个方法print

就会出现这样的效果

![image-20200928094900882](heap-overflow/image-20200928094900882.png)

看到除了author name之外还出现了一些其他的字符串



在Create book的函数中，实际分配图书的结构部分代码如下：

![image-20200928110059338](heap-overflow/image-20200928110059338.png)

存储的位置是在off_202010这个地方的指向的位置

![image-20200928110142911](heap-overflow/image-20200928110142911.png)

也就是从202060开始的区域，而这里可以看到author_name的字符串是存储在202040开始的0x20个字节处的，因此溢出1个字节正好可以覆盖掉第一个book的结构体中id这样的一个字段中的值



利用的思路：

1. 使author name长度为32
2. 创建book1，覆盖掉author name最后的NULL byte；并且要让book的description占比较大的空间，在其中伪造一个book结构
3. 调用print泄漏第一个book结构的地址
4. 创建第二个book
5. 更改author name，再次覆盖掉book1的一个字节，这样使得其指向的空间变大，并且指向了book1的description部分，实际上是一个伪造的book
6. 由于有edit函数和print函数，且book1中伪造的book是可控的，导致实际上可以任意地址读写



首先读到地址

![image-20200929210106951](heap-overflow/image-20200929210106951.png)

这边转换之后得到一个数字，连接到gdb里面查看一下内存

![image-20200929210131532](heap-overflow/image-20200929210131532.png)

可以看到这一块区域首先是一个1，之后是一个指向`0x5433d372b0`的指针，紧跟着是一个`0x5433d372e0`的指针

输出一下发现`b0`这里面是32个字节的b；也就是我们输入的书的name区域

![image-20200929211751690](heap-overflow/image-20200929211751690.png)

之后发现另一个`e0`这个区域是我们输入的description

![image-20200929212122897](heap-overflow/image-20200929212122897.png)

有内容的部分包括0，上面有128个字节，之后一个0，一个31；

我们之前有学过了chunk的结构，首先每一个chunk起始的一个字是存着前一个chunk的size；之后是自己这个chunk的size

![img](https://upload-images.jianshu.io/upload_images/20044065-905e3579f0f56746.png?imageMogr2/auto-orient/strip|imageView2/2/w/703)



## Chunk-Extend

chunk extend在对堆进行扩展时可能会出现overlapping的情况

即不同的堆块之间造成重叠；

利用需要以下条件：

- 程序中存在基于堆的漏洞
- 漏洞可以控制chunk header中的数据



类似的还有一种利用方式是chunk shrink

### 原理

由于在ptmalloc中对堆的各种操作是很频繁的，因此其中的很多操作直接是以宏的形式实现的；

```c
#define chunksize(p) (chunksize_nomask(p) & ~(SIZE_BITS))
#define chunksize_nomask(p) ((p)->mchunk_size)

#define next_chunk(p) ((mchunkptr) (((char *)(p))+chunksize(p)))

#define prev_size(p) ((p)->mchunk_prev_size)
#define prev_chunk(p) ((mchunkptr)((char *)(p) - prev_size(p)))

#define inuse(p) ((((mchunkptr) (((char *)(p))+chunksize(p))) -> mchunk_size) & PREV_INUSE)
```

