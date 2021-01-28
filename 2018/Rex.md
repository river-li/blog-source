rex是Shellphish开发的用于自动化漏洞挖掘的漏洞分类器，该团队用这个库参加CGC获得了很好的名次，但是这个库的依赖十分混乱，导致不是很容易看懂代码，本文整理了Rex的安装过程及对原理、源代码的分析。

<!--more--->

### 环境搭建

rex在github上的地址：[https://github.com/shellphish/rex](https://github.com/shellphish/rex)

clone下来看到其中有一个**requirements.txt**

可以看到其中涉及到的依赖有：

> angr
>
> tracer
>
> angrop
>
> compilerex
>
> shellphish-qemu
>
> povsim

`pip install -r requirements.txt`安装其中涉及到的依赖时会出错，建议分别单独安装

如果单独使用pip安装时出错，就在github上寻找相应的地址下载下来安装

以compilerex为例：地址：[github.com/mechaphish/compilerex](https://github.com/mechaphish/compilerex)

下载下来后在其目录中输入`python3 setup.py install`

依赖全部安装完成后在rex目录中也执行该命令



之后在ipython交互界面中尝试导入rex：`import rex`

 如果没有错误则成功安装

如果出现差错，建议直接clone出问题的库源码安装（build）

**compilerex** 需要cp到python解释器的目录里面，因为在使用rex时会需要编译c代码生成pov



### Crash类

下面开始分析源码，目录结构如下：
{% asset_img tree.png 目录结构%}

 其中crash.py中有一个Crash类，是初始化时使用到的一个类;

#### Crash.__init\_\_()

```python
def __init__(self, binary, 	#Path to the binary which crashed. 
             crash=None,  #String of input which crashed the binary
             pov_file=None,  #CGC PoV describing a crash.
             aslr=None,  #Analyze the crash with aslr on or off
             constrained_addrs=None,  #List of addrs which have been constrained during exploration.
             crash_state=None, #An already traced crash state.
             prev_path=None,  #Path leading up to the crashing block
             hooks=None,   #Dictionary of simprocedure hooks, addresses to simprocedures.
             format_infos=None, 
             rop_cache_tuple=None, 
             use_rop=True, 
             fast_mode=False,
             explore_steps=0, 
             angrop_object=None, 
             argv=None,
             concrete_fs=False, 
             chroot=None, 
             rop_cache_path=None,
             trace_timeout=10, 
             input_type=CrashInputType.STDIN,
             port=None, 
             use_crash_input=False,
             tracer_args=None,
             initial_state=None):
    ...
```

可以看到，参数巨多，因此我们只关注最重要的几个; 

initi函数主要对类的属性进行了初始化，作了一些检测和判断来确定缺省值，有一个重要的属性：**self.project**



```python
...
self.project = angr.Project(binary)
...
```

这里对输入的二进制文件创建了一个angr的Project;

下面是关于Rop的部分操作，比较复杂，在此首先略过，最后再来看对于Rop的操作。

```python
if crash_state is None:
    ...
    pass
else:
    self.state = crash_state
    self.prev = prev_path
    self._t = None    
```

如果crash_state在创建时没有声明，这部分的代码会调用tracer来获取crash_state

展开详细一点看这部分，其中有一行

```python
r = tracer.QEMURunner(binary=binary, input=input_data, argv=argv, trace_timeout=trace_timeout, **tracer_args)
```

这里调用了tracer创建了一个QEMURunner对象

为了方便了解Tracer到底是个啥，我特意去找了QEMURunner类的描述和其中的注释

> Trace an angr path with a concrete input using QEMU

就是用QEMU跟踪具体input到达的angr路径

这里还算比较好理解，input_data现在就是导致crash的输入，r这个对象就是要跟踪导致这个输入的程序执行路径

好的，我们继续

下面有一个部分，判断了一下input_type,然后根据这个input_type 生成了一个input_file

CrashInputType是个啥？在文件头那里发现是从enum.py引入的，去看了看：

> class CrashInputType:
>
> 	STDIN=0type was an arbitrary-write, this routine doesn't care about taking advantage
> 	    # of the write it just wants to try to find a more valuable crash by pointing the write
> 	    # at some writable memory
> ​	POV_FILE=1
>
> ​	TCP=2
>
> ​	UDP=3

就这些，看来是要根据这个输入到底是否需要用到网络文件设定的一个分类

我们继续看

```python
        if input_type == CrashInputType.TCP:
            input_file = input_sock = SimFileStream(name="aeg_tcp_in", ident='aeg_stdin')
            output_sock = SimFileStream(name="aeg_tcp_out")
            socket_queue = [ None, None, None, (input_sock, output_sock) ] # FIXME THIS IS A HACK
        else:
            input_file = stdin_file = SimFileStream(name='stdin', ident='aeg_stdin')
```

反正不管是啥类型都要创建个input_file

这个file是用`SimFileStream`生成的，又是angr的一个部分，具体怎么实现的就先不管了，反正是angr模拟出来的文件

下面吧啦吧啦又初始化了一大堆东西，有这么一行比较重要

```python
simgr.run()
```

**这是angr的factory运行起来了啊！**可以说从最开始初始化一大堆就是为了这么个run，all right，我们再仔细看一下这个simgr是怎么创建的

上面设定的一些属性里面有一个是我们关心的地方：

> self._t = angr.exploration_techniques.Tracer(trace=r.trace, resiliency=False, keep_predecessors=2, crash_addr=r.crash_addr)
>
> simgr.use_technique(self._t)
>
> simgr.use_technique(angr.exploration_techniques.Oppologist())

这个technique名字有点意思，感觉上像是漏洞利用技巧，我们看看它这部分有点啥

Tracer类的描述

> An exploration technique that follows an angr path with a concrete input

再看看这个Oppologist

> The Oppologist is an exploration technique that forces uncooperative code through qemu

在simgr运行了之后，对得到的结果进行了一些判断，并且得到内存的map，之后有这么一句

> self._triage_crash()

之后开始对crash进行分类



#### Crash._triage\_crash(self)

大致一眼看上去，可以看到这个函数进行了几个判断，判断漏洞类型是否是任意转移、ip任意写、部分ip任意写、bp任意写、部分bp任意写几种类型。

我们一个一个看，看看是如何判断漏洞类型的。

##### Arbitrary Transmit

```python
zp = self.state.get_plugin('zen_plugin') if self.os == 'cgc' else None
if zp is not None and len(zp.controlled_transmits):
    l.debug("detected arbitrary transmit vulnerability")
    self.crash_types.append(Vulnerability.ARBITRARY_TRANSMIT)
```

emmmm有点微妙，第一行做了一个判断，看来rex只能对cgc系统的机器检测是否有这种漏洞，那我们暂且略过，时间充裕的话再来看。



##### IP Overwrite

```python
if self.state.solver.symbolic(ip):
    # how much control of ip do we have?
    if self._symbolic_control(ip) >= self.state.arch.bits:
        l.info("detected ip overwrite vulnerability")
        self.crash_types.append(Vulnerability.IP_OVERWRITE)
    else:
        l.info("detected partial ip overwrite vulnerability")
        self.crash_types.append(Vulnerability.PARTIAL_IP_OVERWRITE)
```

调用的一个_symbolic_control(ip)函数定义如下：

```python
def _symbolic_control(self, st):
        '''
        determine the amount of symbolic bits in an ast, useful to determining how much control we have
        over registers
        '''
    sbits = 0
    for bitidx in range(self.state.arch.bits):
        if st[bitidx].symbolic:
            sbits += 1
    return sbits
```

可以看到这个函数将输入之后的一段按位判断是否被符号化，如果被符号化则说明是可控制的状态

结合上面那部分可以得知，如果所有的比特都能被我们控制，则认为是IP Overwrite，如果只有一部分可控，则是partial ip overwrite



##### BP Overwrite

```python
if self.state.solver.symbolic(bp):
    # how much control of bp do we have
    if self._symbolic_control(bp) >= self.state.arch.bits:
        l.info("detected bp overwrite vulnerability")
        self.crash_types.append(Vulnerability.BP_OVERWRITE)
    else:
        l.info("detected partial bp overwrite vulnerability")
        self.crash_types.append(Vulnerability.PARTIAL_BP_OVERWRITE)
    return

```

与前面同理，判断BP的可控制比特



##### Write & Read

这部分可以看到首先是创建了一个symbolic_actions列表

之后对crash前state进行的操作进行判断，如果是对内存（mem）进行操作的类型

则将其添加至列表中

再对列表中的各个操作（action）进行判断，判断其中的写操作是否符号化

如果存在符号化的写指令，则被判断为**Write-What-Where**

如果存在写指令，但没有被符号化，则判断为**Write-X-Where**

如果存在读指令，则被判断为**Arbitary-Read**

---

当以上两个函数执行完毕，我们得到了一个列表：**self.violating_action**

里面存放着这个crash的类型

---



#### Crash.explore(self,path_file=None)

这个模块的函数主要是对前面已经得到的漏洞类型进行进一步的漏洞利用

具体可以对这部分利用的漏洞类型只有Write-X-Where、Write-What-Where、Arbitary-Read

这部分函数首先判断是否属于前面的类型，之后再调用相应的利用函数



##### Crash._explore_arbitrary_read(self,path_file=None)

我们先直接看到这个函数的最后面

```python
self.__init__(self.binary,
             new_input,
             explore_steps=self.explore_steps+1,
             constrained_addrs=self.constrained_addrs + [self.violating_action],
             use_rop=use_rop
             angrop_object=self.rop)
```

咦？有点熟悉，这不是最初初始化的函数吗，看来这个函数在最后又重新创建了一个新的Crash

这也符合explore那里的描述

> explore a crash futher to find new bugs

这样看来new_input这个参数就是能够导致读到flag的输入了

向上看到这个输入的构造：

```py
new_input = ChallRespInfo.atoi_dumps(self.state)
```

真是让人头秃，又是新的类和函数调用

ChallRespInfo是最开始从angr里import的

```py
from angr.state_plugins.trace_additions import ChallRespInfo, ZenPlugin
```

我们去看一看这个函数的定义

```python
atoi_dumps(state,require_same_length=True)
```

这里就是将可以执行到`self.state`状态的输入dump出来作为crash

因此在这一步之前对state进行了修改，看一下前面与state有关的操作

首先将constraint赋初值为`None`，之后在每一个page和region进行判断

如果page在violating_action中，则认为这里的内存是可以约束的



##### Crash._explore_arbitrary_write(path_file):

首先可以看到注释:

> crash type was an arbitrary-write, this routine doesn't care about taking advantage 
>
> of the write it just wants to try to find a more valuable crash by pointing the write
>
> at some writable memory

首先获取loader中所有的ELF文件，对其进行判断筛选得到可写的文件

之后对可写的文件逐页判断是否在violating_action.addr中，最后将执行到可利用地址的输入dump出来作为一个新的输入，再创建一个新的Crash对象



#### Crash.exploit()

```python
def exploit(self,blacklist_symbolic_explore=True,**kwargs):
    """
    craft an exploit for a crash
    """
    	factory = self._prepare_exploit_factory(blacklist_symbolic_explore, **kwargs)
        factory.initialize()
        return factory
```

创建了一个factory，初始化后return

正好这个`_prepare_exploit_factory`函数就在上面

前面一些初始化操作之后，判断系统类型

返回了一个`ExploitFactory`或者是`CGCExploitFactory`对象

这个类是继承了Exploit类的，描述是

> Exploit factory object responsible for managing exploits and creating exploit objects





### Examples

官方的README中实例如下：

```python
# triage a crash
>>> crash = rex.Crash("./legit_00003", b"\x00\x0b1\xc1\x00\x0c\xeb\xe4\xf1\xf1\x14\r\rM\r\xf3\x1b\r\r\r~\x7f\x1b\xe3\x0c`_222\r\rM\r\xf3\x1b\r\x7f\x002\x7f~\x7f\xe2\xff\x7f\xff\xff\x8b\xc7\xc9\x83\x8b\x0c\xeb\x80\x002\xac\xe2\xff\xff\x00t\x8bt\x8bt_o_\x00t\x8b\xc7\xdd\x83\xc2t~n~~\xac\xe2\xff\xff_k_\x00t\x8b\xc7\xdd\x83\xc2t~n~~\xac\xe2\xff\xff\x00t\x8bt\x8b\xac\xf1\x83\xc2t~c\x00\x00\x00~~\x7f\xe2\xff\xff\x00t\x9e\xac\xe2\xf1\xf2@\x83\xc3t")
>>> crash.crash_types
['write_what_where']
>>> crash.explorable()
True
# explore the crash by setting segfaulting pointers to sane values and re-tracing
>>> crash.explore()
# now we can see that we control instruction pointer
>>> crash.crash_types
'ip_overwrite'
# generate exploits based off of this crash
# it may take several minutes
>>> arsenal = crash.exploit()
# we generated a type 1 POV for every register
>>> len(arsenal.register_setters) # we generate one circumstantial register setter, one shellcode register setter
2
# and one Type 2 which can leak arbitrary memory
>>> len(arsenal.leakers)
1
# exploits are graded based on reliability, and what kind of defenses they can
# bypass, the two best exploits are put into the 'best_type1' and 'best_type2' attributes
>>> arsenal.best_type1.register
'ebp'
# exploits can be dumped in C, Python, or as a compiled POV
>>> arsenal.best_type2.dump_c('legit3_x.c')
>>> arsenal.best_type2.dump_python('legit3_x.py')
>>> arsenal.best_type2.dump_binary('legit3_x.pov')
# also POVs can be tested against a simulation of the CGC architecture
>>> arsenal.best_type1.test_binary()
True
```

这里用到了一个CGC二进制文件：[https://github.com/angr/binaries](https://github.com/angr/binaries) 

这个repo下面的defcon24目录下有一个`legit00003`程序

首先使用AFL fuzzing出来一个crash

```
b"\x00\x0b1\xc1\x00\x0c\xeb\xe4\xf1\xf1\x14\r\rM\r\xf3\x1b\r\r\r~\x7f\x1b\xe3\x0c`_222\r\rM\r\xf3\x1b\r\x7f\x002\x7f~\x7f\xe2\xff\x7f\xff\xff\x8b\xc7\xc9\x83\x8b\x0c\xeb\x80\x002\xac\xe2\xff\xff\x00t\x8bt\x8bt_o_\x00t\x8b\xc7\xdd\x83\xc2t~n~~\xac\xe2\xff\xff_k_\x00t\x8b\xc7\xdd\x83\xc2t~n~~\xac\xe2\xff\xff\x00t\x8bt\x8b\xac\xf1\x83\xc2t~c\x00\x00\x00~~\x7f\xe2\xff\xff\x00t\x9e\xac\xe2\xf1\xf2@\x83\xc3t"
```

在ipython中导入rex库，创建一个Crash对象

