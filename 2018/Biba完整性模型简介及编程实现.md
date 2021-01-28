## 前言

在一门名叫信息系统安全导论的课上老师留了一个作业，写一个报告简单介绍一下Biba模型，并且用编程描述一个场景实现一下。

嗯，听起来非常扯对不对，更扯的是后来老师也不知道自己到底留了点啥，结果每次有人去问具体要求他说的都不一样，得，那就随便做点东西得了。

<!--more-->

## 访问控制模型

访问控制的主要作用是让得到授权的主体访问客体，同时组织没有授权的主题访问客体。访问控制可以划分为自主访问控制（**DAC**,Discretionary Access Control) 和强制访问控制（**MAC**,Mandatory Access Control) 两种类型。

### 自主访问控制

用户拥有权限对自己创建的文件进行访问，同时可以将权限授给其他用户，最典型的例子是linux下的访问控制，常见的chmod 777啥的就是修改了对各个类型的用户的权限，按照顺序的三位分别代表：

  ` U 文件拥有者的权限

  ` G 文件所属的用户群组

  ` O 其他人

  而三个数字是三位二进制数的值，分别是rwx 即读、写、执行;

  7就是可读可写可执行，4就是只能读...

 而每个客体都有一个所属的限定主体，以及一个访问控制列表ACL



### 强制访问控制

强制访问控制为每个主体、每个客体分配一个安全等级，当进行操作时对比主体和课题的等级，以确定能否进行相应操作。

典型的模型有：

  - Lattice模型

  - Bell-LaPadula模型

  - Biba模型



## Biba访问控制模型

Biba完整性模型是一种强制访问控制（MAC）模型，其对文件的访问控制建立在完整性上，并不关心保密性，主要有一下几条准则：

1. **设 i、j 是两个完整性级别，如果 i 级别的实体比 j 具有更高的完整性，则称完整性级别 i 绝对支配 j ,记作： i<j  ;**

2. **设 i 和 j 是任意两个级别，当且仅当$i(O)<=j(S)$则S可以写O ;**

3. **当且仅当 i(S1)<=i(S2) ，S2可以执行S1;**

4. 设S是主体，O是客体，min=min {i(S),j(O)},不管完整性如何，S都可以读O，但是读完后i (S)=min  

5. 不管完整性如何，任何主体都可以读任何客体

6. 当且仅当i (S)<=j(O) ，主体S可以读客体O


  Biba模型又分为：

- Biba 低水标模型------->同时满足1、2、3、4

- Biba 环模型------------>同时满足1、2、3、5

- Biba 严格完整性模型--->同时满足1、2、3、6

低水标模型的规则4可以理解为，当主体读过低完整性的内容后，可能会受到潜在影响，因此它的可信度就会降低;

环模型并不控制读，仅控制写与执行;

严格完整性模型一旦满足6,就已经满足了4,实质上也满足低水标的要求。

归纳总结一下，即读高写低


## 编程实现

下面使用python模拟简单的功能：

```python
import sh
import os

UserLevel={}
UserPasswd={}
FileDict={}
```

首先引入需要的模块，并且建立三个字典用于存储主客体的完整性等级

简单分析一下需要实现的功能，可以主要划分为一下几点：

- 用户登录功能
- 对不同用户完整性评级
- 对不同文件完整性评级
- 用户访问文件时的等级对比
- 阅读低级文档后主体的完整性降级（选）

此处我选择使用严格完整性模型，模拟出一个类似终端登录的程序

#### 用户登录模块

1. 读取/创建本地用户列表
2. 当用户登录时输入用户名和密码进入系统
3. 在登录状态下输入访问（读/写）文件的命令->调用评级对比模块
4. 允许读写或拒绝读写


```python
def log_in():
    print("login   :",end='')
    global user
    user=input()
    if user in UserPasswd:
        print("password:",end='')
        passwd=input()
        if UserLevel[user]==passwd:
            print("Welcome!\n")
        else:
            print("Wrong password")
            Err_Report(user,0)
```

登录出现密码错误的情况则调用Err_Reportn函数将情况写入系统日志

### 操作模块

1. ls
2. read file
3. write file
4. run object
5. su user
6. catlog


当用户登录之后程序等待用户的指令，再分别调用指令相应函数检测是否有权操作

```python
def R_file(name,str):
    if FileDict.get(str)==False:
        Err_Report(type=3)
        return

    if UserLevel[name]>FileDict[str]:
        print("Permission Denied!\nYou can only read a higher level file!")
        Err_Report(name,type=1)
    else:
        with open(str,"r") as f:
            content=f.read()
            print(content)

def W_file(name,str):
    if len(str)<3:return
    if FileDict.get(str[1])==False:
        Err_Report(type=3)
        return

    if UserLevel[name]<FileDict[str[1]]:
        print("Permission Denied!\nYou can only write a lower level file!")
        Err_Report(name,type=2)
    else:
        with open(str,"w") as f:
            f.write(str[2:])
        
def get_instruction(name):
    print("[%s@Biba %s]:" % (name,os.getcwd()),end='')
    ins=input()

    if ins.isspace()==True:
        return True

    ins=ins.split( )
    if(ins[0]=="exit"):
        return False
    else:
        if(ins[0]=="ls"):
            print(sh.ls())
        elif(ins[0]=="read"):
            R_file(name,ins[1])
        elif(ins[0]=="write"):
            W_file(name,ins)
        elif(ins[0]=="help"or ins[0]=="?"):
            Help_info()
        elif(ins[0]=="change" && len(str)==2):
            Change_User(ins[1])
        return True
```