这是Pwntools中的一个格式化字符串利用的自动化类

```python
FmtStr(exec_fmt,offset=None,padlen=0,numberwritten=0)
```

传入的参数是一个函数，用于和漏洞程序通信的；

例如
```python
def exec_fmt(payload):
	p = process('./pwn')
	p.send(payload)
	return p.recv()

f = FmtStr(exec_fmt)
```

![image-20210408223101023](https://static.hack1s.fun/images/2021/04/08/image-20210408223101023.png)

可以看到加载程序的时候就会反复执行测试出offset

