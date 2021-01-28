netfilter的相关知识，主要是网络程序设计防火墙中用到的一些内容

netfilter一共在网络报文的流向中加了5个钩子，位置分别在:

- NF_IP_PRE_ROUTING	在报文路由前处理
- NF_IP_FORWARD 在IP转发向另一个NIC前处理
- NF_IP_POST_ROUTING 在报文路由后处理
- NF_IP_LOCAL_IN 在本地
- NF_IP_LOCAL_OUT

用户自定义的钩子函数返回值必须是

- NF_DROP
- NF_ACCEPT
- NF_STOLEN
- NF_QUEUE
- NF_REPEAT
- NF_STOP

中的一个；

网络防火墙的编写主要就是在将自定义函数挂在钩子上，通过判断数据包和防火墙的路由表判断进一步的操作。

<!--more--->

## 前导知识

### 内核模块编程

防火墙的编写以及netfilter模块的利用一般需要在内核态

在内核加载一个新的模块

下面是一个内核中的hello world模块

```C
#include <linux/module.h>
#include <linux/init.h>

static int __init helloworld_init(void){
    printk(KERN_ALERT"hello world\n");
}

static void __exit helloworld_exit(void){
    printk(KERN_ALERT"Hello world exit\n");
}
           
module_init(helloworld_init);
module_exit(helloworld_exit);
```

之后调用时使用`insmode`和`rmmod`



Q：为什么需要使用内核模块编程？

A：内核级的防火墙需要能够分析、检测所有 的数据包，因此需要工作在内核级别；而用户态下并没有这样的权限。



## Hook

在内存中有一个nf_hook_ops结构的链表，每一个元素用于标识一个钩子

这个结构定义如下:

```C
struct nf_hook_ops{
    struct list_head list;
    nf_hookfn *hook;
    struct module *owner;
    int pf;
    int hooknum;
    
    int priority;
}
```

其中list_head设置为{NULL,NULL}即可

nf_hookfn指向用户自定义的hook函数

pf指定协议族，例如ipv4值为PF_INET

hooknum是想注册钩子的位置，取值为上面那五个中的一个



### 注册钩子

```C
int nf_register_hook(struct nf_hook_ops *reg);
```

向函数传入一个nf_hook_ops，则会将钩子加载在内存中的列表中，这样会在设置的时机调用钩子函数



### 注销钩子

```c
void nf_unregister_hook(struct nf_hook_ops *reg);

int nf_unregister_hooks(struct nf_hook_ops *reg, unsigned int n);
```

第二种方法可以一次性注销n个钩子函数，这里的reg是个链表



### 其他注册注销函数

netfilter中还有其他的注册函数，例如控制socket选项的钩子函数

```C
int nf_register_sockopt(struct nf_sockopt_ops *reg);
void nf_unregister_sockopt(struct nf_sockopt_ops *reg);
```

实际上，实际应用注册钩子时一般都需要注册socket选项的函数

通过设置用户自定义的sockopt才能够实现对命令的调用。这样可以支持动态的修改配置



同时需要注意，在处理网络的数据包时，需要从用户空间复制到内核空间，钩子函数处理完毕后再从内核空间复制回用户空间；



### 钩子函数处理的例子

通过设置钩子函数，实现以下功能：

- 禁止向某ip地址发包
- 禁止某端口响应数据
- 禁止ping回显

对与上面三个功能，钩子分别需要挂在LOCAL_OUT、LOCAL_IN、以及LOCAL_IN

只需要使用ip_hdr获取ip报文的头部，对比其中的ip地址、端口号、或是协议类型即可

如果需要支持动态修改配置,需要加上sockopt的设置



实现时的代码流程：

首先可以确定的是过滤的条件以及钩子函数都在内核级别；

因此需要模块编程，加载模块的函数

```C
static int __init init(void)
```

这个函数内部需要：注册钩子函数

```C
nf_register_hook(&nfin)
```

这个nfin应该是一个nf_hook_ops结构体

这个结构体应该是一个全局变量，并且里面的`.hook`应该指向的是处理函数

```C
static struct nf_hook_ops nfin=
{
    .hook = nf_hook_in,
    .hooknum = NF_IP_LOCAL_IN,
    .pf = PF_INET,
    .priority = NF_IP_LOCAL_OUT
}
```

结构体应该是这样的一个内容

里面的nf_hook_in是处理函数

```C
static unsigned int hf_hook_in(...)
```

这个函数内部应该实现对数据包的判断，并根据条件返回那几个返回值中的一个。