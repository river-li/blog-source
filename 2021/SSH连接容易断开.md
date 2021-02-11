SSH连接一段时间不处理之后容易断开，没反应，最后弄的必须要关闭终端；

在网上查了一下解决办法；

在客户端的`/etc/ssh/ssh_config`中增加两行

```
TCPKeepAlive=yes
ServerAliveInterval 10
```

这样即使很长时间不动也不会断开了

