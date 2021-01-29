写爬虫的时候希望能有个终端里的进度条，但是用logging出现的信息有点太多了，于是希望能像一些开源软件那样出现进度条，查了一下发现有一个叫做`progress`的库，简单记录学一下用法。


<!--more-->


## Bar

首先看一下库中的样式组成

![category.jpg][1]

总体而言有这三种类型

- bar
- counter
- spin



以最常用的bar为例，一段简单的代码如下

```python
import time
from progress.bar import Bar

bar = Bar('Processing', max=100, fill='@', suffix='%(percent)d%%')
for i in range(100):
    time.sleep(0.1)
    bar.next()
bar.finish()
```



运行时效果如图

![example.gif][2]



这段代码中三个参数的含义分别是

- Max: 最大数量
- Fill: 填充用的字符，一般可以选择`*`,`%`,`#`,`@`等
- suffix: 百分比的标志



## Spin

spin是另一类的样式，就是原地转圈那种样子
![spin.gif][3]

同样先来段示例代码

```python
from progress.spinner import Spinner
import time

spinner = Spinner('Loading ')

for i in range(100):
    spinner.next()
    time.sleep(0.1)

spinner.finish()
```

这个调用起来就更简单了，不过没办法显示具体的进度

只要在状态没结束前一直吊用next方法就可以了

## 注意事项

在bar和 spin结束之后记得要用`bar.finish()`，否则会卡在这个界面


[1]: http://42.193.111.59/usr/uploads/2021/01/510759994.jpg#vwid=460&vhei=166
[2]: http://42.193.111.59/usr/uploads/2021/01/2481530743.gif#vwid=420&vhei=210
[3]: http://42.193.111.59/usr/uploads/2021/01/2551442411.gif#vwid=420&vhei=208