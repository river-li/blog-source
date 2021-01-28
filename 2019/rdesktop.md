## rdesktop

rdesktop是linux下面的远程桌面，可以用来连接windows的rdp服务

rdp服务一般开3389端口

在渗透测试时可能经常需要用到

常用的命令

```bash
rdesktop -u user -p pass 192.168.1.1 -f
```

- -u参数后面接用户名
- -p参数接密码
- -f选项意思是全屏
- -g 1024*720 不想全屏也可以这样设置分辨率

除了这几个还有一个比较重要的参数:

- -r 作用是多媒体重定向

可以通过`-r`的参数将声卡映射过去，也可以将本地的文件夹映射

例如：

```bash
rdesktop -u user -p pass -r disk:aa=/root/Documents [ip]
```

这样会将linux下面的`/root/Documents`文件夹在windows中作为一个磁盘出现

也可以映射成一些其他的东西

```bash
rdesktop -u user -p pass -r sound:local [ip]
```

可以将本地的声卡映射过去，这样在远程桌面播放声音时本地这里会响

