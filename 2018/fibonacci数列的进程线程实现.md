## 前言

​        操作系统第二次的大作业，留了三个部分：分析linux下进程的结构体;写一个用fork函数让子进程输出fibonacci数列，同时父进程等待的程序;使用子线程计算fibonacci数列，并用父线程输出数列。

<!--more-->

## Fibonacci数列

这个数列的第一项第二项为1,之后的每一项都是前两项的和，即数列前几项为：

1,1,2,3,5,8,13......

很容易理解，也不难实现对不对

## 子进程输出分析与实现

那我们先来仔细看一看题目的要求

> ​        *使用系统调用fork（）编写一个C程序，它在子程序中生成Fibonacci数列，序列的号码将在命令行中提供。由于父进程和子进程都有它们的数据副本，对子进程而言，输出序列i是必要的。程序退出前，父进程调用wait（）调用来等待子进程结束。执行必要的错误检查以保证不会接受来自命令行的负数号码。*

​        这样看来就比较清楚了，只要让子进程输出序列，父进程等待就行了，那么下面就是几个部分的实现：

- 从命令行读取输入的数字
- fork
- 求fibonacci数列
- 子进程输出
- 父进程等待

​        下面就这几部分，分别介绍思路

#### 读取参数

​        这部分功能很好实现，只要一个 argc argv 就能搞定，不过多赘述

​        值得一提的就是C的标准库中stdio中有一个函数 *int atoint(char \*)*可以将字符串直接转化为整形的数字，知道这个函数的话就省了很多的事情，当然你自己编一个这样的函数也费事不到哪去。

​        另外，对于参数的处理，argc!=2 以及 argv[1]<=0 两种情况报错，其余情况继续执行。

####  fork

​        fork会返回一个子进程的pid，同时会创建一份父进程的copy给子进程，数据和代码完全相同。之后子进程和父进程都是从fork的下一条指令继续执行，唯一的不同点在于pid在子进程中值为0,而在父进程中为子进程的进程号。

​        因此在此处利用两者pid的区别使父进程等待、子进程执行。

#### 子进程的输出

​        关于Fibonacci数列的计算，有一些编程基础的人都会知道应该使用递归，常常会看到一种求解方法，例如：

```c
int fib(int n)
{
    if (n==1 || n==2) return 1;
    else return fib(n-1)+fib(n-2)
}
```

​        看起来简单啊，有哪里不对吗？

​        一个字：慢。求个四五十，能算好久。因为从后向前分解，数字比较大的话会分解很多个分支，最下面的子节点太多了。改进方法也很简单，改成从前往后递归就会快很多，具体代码此处略。

#### 父进程的等待

​        在头文件*<sys/wait.h>*中有一个wait函数，在父进程中执行这个函数即可。关于wait函数的具体介绍可以看一看《UNIX环境高级编程》的第八章第六节（我看的是第二版）。

>  wait(NULL);

## 子线程计算父线程输出分析与实现

同样还是先看题目要求：

> 使用JAVA、Pthread或win32线程库编写一个多线程程序生成Fib数列，程序应当创建一个新线程来产生Fib数，把这个序列放在共享数据段，当线程执行完毕后父线程输出子线程产生的数列，由于子线程结束前父线程不能开始输出序列，因此必须等待子线程结束。

​        类似的，将任务划分为以下几个部分：

- 读取数据
- 创建线程
- 子线程计算，同时父线程等待
- 父线程输出
- 求Fib数列

​        关于Fibonacci数列的求法和读取数据部分这里略去不讲，其余几个部分分别作简单介绍。

#### Pthread

​        所有与pthread相关的函数都在pthread.h头文件中。名字简单易懂，一看这个函数就是创建子线程的，查阅相关资料后，这个函数的原型为：

>  int pthread_create(pthread_t *restrict tidp, const pthread_attr_t *restrict attr, void *(start_rtn)(void), void *restrict arg);

创建成功返回0,否则返回错误编号，几个参数的含义：

- tidp:指向新线程ID的变量
- attr:用于定制各种不同的线程属性
- start_rtn:函数指针，线程开始执行的函数的名字
- arg:函数的唯一void类型指针参数，当需要传递多个参数时，需要用结构体封装

​        attr参数是一个结构指针，结构中的元素是线程运行的属性，具体可以看《UNIX环境高级编程》第十一章。

​        我们这里直接用（也可以直接仿照课本——《操作系统概论》——上的例子来改），一个pthread_attr_init函数用于初始化属性结构，一个pthread_create函数用于创建线程;之后main函数调用pthread_join函数运行新的子线程，并在此时等待;当子线程运行结束后调用pthread_exit函数退出，同时给父线程一个信号继续执行，具体用法见代码。

