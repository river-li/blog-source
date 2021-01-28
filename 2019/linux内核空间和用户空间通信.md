Linux 下用户空间和内核空间通信主要有这么两个方法

`netlink`

或者时`/proc/`虚拟文件

正好两个都在防火墙的设计中用到了，整理下用法和内容

<!--more--->

## NetLink

netlink的作用是在linux下用户空间和内核空间进行双向的通信

可以理解成其作用是在内核空间和用户空间之间创建了一个socket

两者可以使用这个socket来互相发送数据

### 用户空间设计

和socket的模式类似，创建的过程主要也分以下几个步骤：

- socket()
- bind()
- sendmsg()
- recvmsg()
- close()

```C
int s = socket(AF_NETLINK,SOCK_RAW,NETLINK_ROUTE);
```

在使用bind时，需要将一个一个套接字结构和一个地址结构绑定

地址结构的原型：

```C
struct sockaddr_nl
{
    sa_family_t nl_family;
    unsigned short nl_pad;
    __u32 nl_pid;
    __u32 nl_groups;
} nladdr;
```

其中pid需要是当前进程的pid，可以用下面的获得

```C
nl_pid = getpid();

bind(s,(struct sockaddr*) &nladdr, sizeof(nladdr));
```



在通信时使用的sendmsg和recvmsg两个函数

使用的数据结构和平常的socket通信中的消息类型比较类似

```C
struct nlmsghdr
{
    __u32 nlmsg_len;
    __u16 nlmsg_type;
    __u16 nlmsg_flags;
    __u32 nlmsg_seq;
    __u32 nlmsg_pid;
}
```

实际发送时的结构:

```C
struct iovec vec;
iov.iov_base = (void *)nlmh;
iov.iov_len = nlmh->nlmsg_len;
msg.msg_iov = &vec;
msg.msg_iovlen =1 ;
```



### 内核空间

内核空间与用户空间有很大的差异

不过差异主要也是在接口的部分

创建socket

```c
struct sock *netlink_kernel_create(int unit,void (*input)(struct sock *sk, int len));
```

内核空间创建的套接字后一个参数是一个函数，作用是将这个输入函数绑定为套接字的回调函数

创建成功后返回`structure sock`指针类型的值可以对这个值操作来操作内核的socket

当用户空间通过socket发送信息后，会调用这个被绑定了的函数

```c
void input(struct *sock,int len)
{
    struct sk_buf *skb;
    struct nlmsghdr *nlh = NULL;
    u8 *payload = NULL;
    
    while((skb = skb_dequeue(&sk->receive_queue))!=NULL)
    {
        nlh = (struct nlmsghdr *)skb->data;
        payload = NLMSG_DATA(nlh);
    }
}
```

在需要想用户空间发送数据时，要区分单播地址还是多播地址

```c
NETLINK_CB(skb).groups = local_groups;
NETLINK_CB(skb).pid = 0;
NETLINK(skb).dst_groups = dst_groups;
NETLINK_CB(skb).dst_pid = dst_pid;
```

首先都需要设置发送地址和接受的地址

之后如果发送单播

```c
int netlink_unicast(struct sock *ssk,struct sk_buff *skb,u32 pid, int nonblock);
```

参数ssk是创建socket时返回的那个指针

skb->data应该指向要发送数据的缓冲区

发送多播时

```c
int netlink_broadcast(struct sock *ssk,struct sk_buff *skb,u32 pid,u32 group,int allocation);
```



最后需要释放socket时

```c
void sock_release(struct socket *sock);
```



## 虚拟文件proc

proc文件系统时一种虚拟文件系统，在proc虚拟文件系统中可以实现用户空间和内核空间通信

### 文件结构

核心的文件数据结构时`struct proc_dir_entry`,用来表示一个虚拟的文件

```C
struct proc_dir_entry
{
    const char *name;
    mode_t mode;
    uid_t uid;
    gid_t gid;
    struct inode_opeartions *proc_iops;
    struct file_operations *proc_fops;
    strutc proc_dir_entry *parent;
    ...
    read_proc_t *read_proc;
    write_proc_t *write_proc;
    void *data;
    atomic_t count;
}
```

### 创建文件

创建文件目录的函数原型：

```c
struct proc_dir_entry *proc_mkdir(const char *dir_name, struct proc_dir_entry *parent);
```

这个函数在parent下创建一个名为dir_name的文件夹，创建失败时返回NULL



```c
struct proc_dir_entry *create_proc_entry(const char *name, mode_t mode, struct proc_dir_entry *parent);
```

这个函数会在文件系统中创建一个虚拟文件，文件名为name指定的字符串

权限由mode指定，而parent说明此文件的位置



对于parent这个参数，有这么几个预定义的值

- proc_root 代表路径/proc

- proc_net 代表/proc/net

- proc_bus 代表/proc/bus

- proc_root_dirver 代表/proc/drivers

- proc_root_fs 代表/proc/fs

  

### 删除文件

删除文件时需要指定文件名和父目录

```c
void *remove_proc_entry(const char *name, struct pro_dir_entry *parent);
```

### 文件的读写

在用户态写入文件，内核态读取文件，就可以实现用户态和内核态的文件交互

```c
int write_proc_t(struct file *file,const char __user *buffer, unsigned long count, void *data);
```

buffer时传过来的数据指针，data是指向私有数据的指针



在用户空间的程序读取创建的proc文件时，内核会分配给proc读取程序一页大小的内存空间

读文件的函数

```c
int proc_read(char *page,char **start,off_t offset, int count, int *eof, void *data);
```



### 内核空间

内核空间中对于proc文件操作的一整套接口都会发生变化

#### 打开文件

```C
struct file *filp_open(const char *filename, int flags, int mode);
```

#### 文件读写

````c
ssize_t vfs_read(struct file *filp, char __user *buffer,size_t len, loff_t *pos);
ssize_t vfs_write(struct file *filp, const char __user*buffer,size_t len, loff_t *pos);
````

其中filp是由前面`filp_open`返回的文件指针

buffer是读出文件要读到的缓冲区，或是写入文件来自的数据

pos是指针位置，指定从什么位置开始读写文件

#### 关闭文件

```c
int flip_close(struct file*filp, fl_owner_t id);
```

