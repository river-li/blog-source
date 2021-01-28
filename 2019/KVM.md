这次准备在Linux上面装虚拟机，虽然有现成的软件比如Virtualbox和VMWare

但是在i3的环境下运行的不太好

这次主要是想用KVM和Qemu，这两个的性能还好一些

（最主要的愿望当然还是打游戏）

<!--more--->

## Installation

这次由于机缘巧合，电脑上的发行版从Arch转到了Manjaro

虽然整体上差别不大，在安装KVM时需要安装这些包

```bash
sudo pacman -S libvirt
sudo pacman -S qemu
sudo pacman -S ovmf
sudo pacman -S virt-manager
```

安装完成这些包就需要装具体的虚拟机了

首先来装个MacOS吧

```bash
git clone https://github.com/foxlet/macOS-Simple-KVM.git
```
下载下这个repo

之后需要运行这个repo里的脚本来获取MacOS镜像

```bash
./jumpstart.sh --[high-sierra|mojave|catalina]
```
这个过程最好有代理，下载会快一点

之后创建QEMU的硬盘介质

```bash
qemu-img create -f qcow2 MyDisk.qcow2 64G
```

然后这个repo的脚本有一点点需要修改的地方

首先是需要在basic.sh的脚本最后加上以下内容:

```bash
    -drive id = SystemDisk，if = none，file = MyDisk.qcow2 
    -device ide-hd，bus = sata.4，drive = SystemDisk 
```

然后是前面的MAC地址需要修改一下，这个可以用以下命令生成:
```bash
openssl rand -hex 6 | sed 's/\(..\)/\1:/g; s/:$//'
```
将结果复制过去就可以了

最后运行这个脚本就可以启动MACOS
