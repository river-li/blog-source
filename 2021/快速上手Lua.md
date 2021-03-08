## 变量和控制流

首先基本的变量赋值比较类似Python，变量本身可以直接赋值

```lua
-- 注释

num = 42
s1 = 'aaa'
s2 = "bbb"
s3 = [[
multi-line
strings
]]
```

有一个不太一样的机制，就是垃圾回收，可以undefine变量

```lua
s1 = nil
```

这样就可以将前面定义的变量取消，回收这个变量

循环需要靠`do`、`end`进行标识

```lua
while num<40 do
  num=num+1
end
```

或者用`for`循环

```lua
for i = 1,100 do
  num=num+1
end
```

`for`循环还可以指定step，也可以在循环条件处设置变量

```lua
for j=100,1,-2 do 
  num=num+j
end
```





`if`条件语句虽然不需要使用`do`但是需要有`end`结束

```lua
if num>40 then
  print('over 40')
elseif num=40 then
  print('equal 40')
else
  print('below 40')
end
```



布尔变量，一些其他语言中0或`''`都会当作`false`

但是lua中只有`nil`和`false`是`false`，0和`''`都是`true`

如果lua中赋值时用到了一个未出现过的变量，会把它当作`nil`来处理，并不会报错

```lua
u = Whatisthis --赋值为未出现过的变量，这时u==nil
```







## 函数

函数的基本结构如下

```lua
function fib(n)
  if n<2 then return 1 end
  return fib(n-2)+fib(n-1)
end
```

另外就是lua可以支持匿名函数和闭包，函数的返回值可以是一个函数指针这样的

```lua
function create_func(x)
  return function(y) return x+y end
end
```



## 表

lua中的表是lua中唯一的复合数据结构

类似于php中的array，是一种哈希表，同时也可以当成列表来用

```lua
t = {key1 = 'value1', key2 = false}

t.key3="123"
t.key2=nil
```

创建新的项只需要直接赋值新的键值对就可以，删除某个内容的话就使用赋值为`nil`的方法



表的迭代

```lua
for key,val in pairs(t) do
  print(key,val)
end
```



当作列表使用

```lua
v = {1,2,3,4,5}
for i = 1,#v do
  print(v[i])
end
```

其中`#v`这样的可以获取表的长度，但是需要注意的是这个长度只包含表中索引了的项，如果索引提前断在某个地方就无法获取到正确的长度了

![image-20210301170041889](https://static.hack1s.fun/images/2021/03/01/image-20210301170041889.png)

例如这里在原本的基础上增加一个键值对，破坏了索引，就没办法获取长度了



当成列表时如果要像键值那样迭代可以用这样的

```lua
for _,val in pairs(t) do
  print(val)
end
```





### 类

lua中并没有内置类这样的功能，但是可以通过表来实现类似的效果

```lua
Dog = {}                                   -- 1.

function Dog:new()                         -- 2.
  newObj = {sound = 'woof'}                -- 3.
  self.__index = self                      -- 4.
  return setmetatable(newObj, self)        -- 5.
end

function Dog:makeSound()                   -- 6.
  print('I say ' .. self.sound)
end

mrDog = Dog:new()                          -- 7.
mrDog:makeSound()  -- 'I say woof'         -- 8.

```

也可以进一步实现继承的效果

```lua
LoudDog = Dog:new()                           -- 1.

function LoudDog:makeSound()
  s = self.sound .. ' '                       -- 2.
  print(s .. s .. s)
end

seymour = LoudDog:new()                       -- 3.
seymour:makeSound()  -- 'woof woof woof'      -- 4.

```







## 模块

不管哪一种编程语言，肯定都是需要用到外部库的

lua引入第三方库的方式一般是下面这样使用`require`

```lua
local mod = require('mod')
```

实际上`require`可以理解为

```lua
local mod = (function()
  <contents of mod.lua>
end)()
```

即，将一个外部的lua文件当成函数执行了一次

所以如果是在`mod.lua`中写在`local`的方法和变量就是无法直接调用的

使用`require`导入的库是经过cache的，在导入的时候只会执行一次，第二次不会执行，如果想要不经过cache，可以使用`dofile`

```lua
dofile('mod.lua')
```

也可以将文件先加载进来，之后再运行

```lua
f = loadfile('mod.lua') -- 这里并不会执行
f() -- 这里才会运行
```

