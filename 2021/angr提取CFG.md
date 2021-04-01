angr内置的Analyzer就可以分析提取程序的CFG，写两个简单的例子；

```bash
mkvirtualenv angr
pip install angr
pip install angr-utils
```



angr本身没有提供输出的功能，所以需要另外安装一个库`angr-utils`

```python
import angr
from angrutils import plot_cfg

proj = angr.Project('./binary',load_options={"auto_load_libs":False})
cfg = proj.analyses.CFG()

plot_cfg(cfg,'output.png')
```

这里首先有几个选项

`load_options={"auto_load_libs":False}`如果不关闭的话angr会在加载的时候load其他的依赖库

导致画出来的CFG巨...大

![image-20210331105054061](https://static.hack1s.fun/images/2021/03/30/image-20210331105054061.png)

这里只比较一下size吧，画出来就太大了；



另外，angr中analyse后可以生成两种CFG

静态的`CFGFast`以及动态模拟生成的`CFGEmulated`

`CFGFast`是用静态分析的方法生成的CFG，这种方式生成的CFG特别的快，但是这也导致准确率会比较低，因为有很多控制流转移只有在执行时才会被解析，据angr说其他流行的逆向分析工具都是用的这种方法，这样得到的输出也是和他们的结果类似；

![cfgfast](https://static.hack1s.fun/images/2021/03/31/cfgfast.png)

`CFGEmulated`则是使用符号执行来获取CFG的，从理论上就会更加精确，但是显然也会更慢；

虽然相对来说可能更精确，但是实际上由于模拟执行的准确度问题，也会有一些差异，这里的因素包括系统调用、缺失相关需要硬件的功能等等；

![cfgemulated](https://static.hack1s.fun/images/2021/03/31/cfgemulated.png)

用`angr-utils`生成的CFG是静态的图片，没有交互效果；

如果要生成类似IDA那种可以交互的图可以看https://github.com/axt/cfg-explorer

这个可以生成出来一个Web中的交互页面；