CTF里常用到curl，但是对各个参数不太熟悉，准备好好整理一下用法



HTTP请求方法

HTTP1.1中有8种规定的请求方法(method)

curl通过指定参数`-X`可以设置请求方法

```bash
curl -X POST http://www.baidu.com
```



需要加POST的参数的话可以使用`-d`

```bash
curl -X POST -d name=alice -d pwd=123456 http://xxx.com
```



设置请求头可以用参数`-H`

```bash
curl -H "User-Agent: curl"
```



设置cookie可以直接用`--cookie`

例如：

```bash
curl --cookie "admin=1;" http://example.com
```



