拿到的第三台机器的权限，还是一个active的，虽然也是个easy


<!--more-->


## foothold

首先进行nmap扫描

![nmap.png][1]

发现开启了80端口和22端口

下面对80端口的目录进行扫描

![gobuster.png][2]

发现了两个目录，`/artwork`和`/music`

两个目录分别访问了一下，在`/artwork`没有什么发现，但是在`/music`发现存在一个登录的位置

![music.png][3]

进去发现是一个叫做ona的东西，仔细看一下发现是`opennetadmin`

![ona.png][4]

于是更新了一下`exploitdb`,搜索一下有没有相关的漏洞

![exploitdb.png][5]

运行，获取到`www-data`权限

![foothold.png][6]



## Jimmy

获得到`www-data`权限后，在当前的目录下查看文件，找到了一个配置文件中包含密码

![pass.png][7]

之后`ls /home` 发现存在两个用户，分别是`jimmy`和`joanna`

用这个文件的密码尝试登录，发现是`jimmy`的密码

![jimmy.png][8]



## joanna

获取到`jimmy`的权限后，在根目录没有发现`user.txt`，因此可能在`joanna`的目录中

继续查看文件，在`/var/www/internal`中发现了一个`php`文件

![jimmy.png][9]

根据名字推测可能是开放了只有内网才可以访问的端口，在`apache`的配置文件目录那里找到了一个配置文件

![apache.png][10]

文件提示开放的端口是52846

结合上面看到的`main.php`使用`jimmy`的权限发起请求，得到了`joanna`的`ssh-key`

![ssh-key.png][11]

这个key经过了加密，使用ssh2john转换之后再用john进行爆破，这里字典用的是`rockyou.txt`

![rockyou.png][12]

这里得到了`joanna`的`passphrase`

使用这个key连接，得到joanna的权限

![joanna.png][13]



## root

获取到joanna的权限后上传了一个`lse.sh`

扫描之后发现存在sudo不需要密码的问题

![sudo.png][14]

最后结合gtfobins中nano的条目绕过，获得root权限

![root.png][15]


[1]: http://42.193.111.59/usr/uploads/2021/01/314140650.png#vwid=736&vhei=358
[2]: http://42.193.111.59/usr/uploads/2021/01/3616582370.png#vwid=949&vhei=546
[3]: http://42.193.111.59/usr/uploads/2021/01/19697721.png#vwid=2471&vhei=938
[4]: http://42.193.111.59/usr/uploads/2021/01/983950829.png#vwid=2547&vhei=664
[5]: http://42.193.111.59/usr/uploads/2021/01/1778542221.png#vwid=1750&vhei=180
[6]: http://42.193.111.59/usr/uploads/2021/01/2571251397.png#vwid=678&vhei=714
[7]: http://42.193.111.59/usr/uploads/2021/01/2574217079.png#vwid=791&vhei=729
[8]: http://42.193.111.59/usr/uploads/2021/01/3447896978.png#vwid=798&vhei=304
[9]: http://42.193.111.59/usr/uploads/2021/01/3447896978.png#vwid=798&vhei=304
[10]: http://42.193.111.59/usr/uploads/2021/01/1756267806.png#vwid=504&vhei=265
[11]: http://42.193.111.59/usr/uploads/2021/01/2953602614.png#vwid=687&vhei=641
[12]: http://42.193.111.59/usr/uploads/2021/01/3333480200.png#vwid=846&vhei=697
[13]: http://42.193.111.59/usr/uploads/2021/01/1328678923.png#vwid=853&vhei=517
[14]: http://42.193.111.59/usr/uploads/2021/01/3636502078.png#vwid=839&vhei=242
[15]: http://42.193.111.59/usr/uploads/2021/01/1747204090.png#vwid=534&vhei=210