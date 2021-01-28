## 前言

​        在下载实验课的课件时发现，linux下zip文件解压出来的文件都是乱码，于是查询了一些相关的资料，解决方法留存以备用。

<!--more-->

## 解决方法
​       windows下zip文件编码默认是GBK，而 linux下默认的unzip是指定UTF-8的编码，所以只要指定解压时的编码就可以，unzip有一个-O的参数，但不是所有版本都支持，因此给unzip打补丁就可以解决了。

- arch下直接安装unzip的补丁版```sudo pacman -S unzip-iconv```安装这个版本的unzip后，需要解压时unzip -O cp936就可以
- debian下可以使用unrar（默认安装），简单又方便，解压时使用unrar就可以;也可以安装unzip的补丁，[使用方法](https://link.zhihu.com/?target=https%3A//github.com/ikohara/dpkg-unzip-iconv)
