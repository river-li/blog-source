`getopt_long`是linux下C语言用来解析命令行参数的常用函数

这里主要写命令行参数的解析方法                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  

<!--more--->


## getopt_long

```C
#include <getopt.h>
int getopt(int argc,char* const argv[],const char *optstring);

int getopt_long(int argc,char *const argv[],const char *optstring,const struct option *longopts,int *longindex);

int getopt_long_only(int argc,char* const argv[],const char* optstring,const struct option *longopts,int *longindex);
```

相关的一共有三个函数

- `getopt`
- `getopt_long`
- `getopt_long_only`

参数中，`argc`和`argv`都应该和`main`的参数相同

`optstring`表示短选项字符串

> 例如`a:b::cd`，表示的optstring字符串
>
> 有`-a`,`-b`,`-c`,`-d`四个参数，其中：
>
> 不带冒号表示命令行选项 -c 、-d
>
> 带一个冒号表示后面有一个参数 -a 100
>
> 带两个冒号，表示后面有一个可选参数，如果带参数则不能用空格 -b200 或-b

`longopts`表示长选项的结构体

其最后一行必须要用0填充

```C
struct option
{
    const char *name;
    int has_arg;
    int *flag;
    int val;
}

 static struct option longOpts[] = {
      { "daemon", no_argument, NULL, 'D' },
      { "dir", required_argument, NULL, 'd' },
      { "out", required_argument, NULL, 'o' },
      { "log", required_argument, NULL, 'l' },
      { "split", required_argument, NULL, 's' },
      { "http-proxy", required_argument, &lopt, 1 },
      { "http-user", required_argument, &lopt, 2 },
      { "http-passwd", required_argument, &lopt, 3 },
      { "http-proxy-user", required_argument, &lopt, 4 },
      { "http-proxy-passwd", required_argument, &lopt, 5 },
      { "http-auth-scheme", required_argument, &lopt, 6 },
      { "version", no_argument, NULL, 'v' },
      { "help", no_argument, NULL, 'h' },
      { 0, 0, 0, 0 }
    };
```

`val`表示指定函数找到该选项时的返回值

`longindex`指向的变量将记录当前找到参数符合`longopts`里的第几个元素的描述，即longopts的下标值



这个函数的返回值有这么几种情况：

- 如果短选项找到，返回短选项对应字符
- 如果长选项找到，如果flag为NULL，返回val；如果flag不为空，返回0
- 如果选项没有在长短字符里面，或者存在二义性，返回"?"
- 解析所有字符都没有找到，返回-1
- 如果选项忘了加参数，如果其第一个字符是“：”返回":",否则返回“？”

除了这几个参数，还有几个可以使用的全局变量

- optarg: 当前选项对应的参数值
- optind: 下一个将被处理的argv下标值
- opterr: 设置在遇到错误时是否要输出
- optopt: 未被标识的选项