CTF里常用到curl，但是对各个参数不太熟悉，准备好好整理一下用法



HTTP请求方法

HTTP1.1中有8种规定的请求方法(method)

curl通过指定参数`-X`可以设置请求方法

```bash
curl -X post http://www.baidu.com
```



设置cookie可以直接用`--cookie`

例如：

```bash
curl --cookie "admin=1;" http://example.com
```

