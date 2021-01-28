## windows 的引导扇区被误删，修复方法

首先制作一个Windows的U盘启动盘，启动之后找到里面的修复计算机

在修复计算机的选项里存在一个命令行参数

使用diskpart找到引导扇区所在的分区，并命名

同时还需要找到安装Windows的C盘所在的分区并命名

之后使用bcdboot修复即可

完整的cmd命令如下:

```powershell
diskpart

> list disk

> select disk 0

> list part

> select part 2

> assign letter P:

> select disk 1

> list part

> select part 2

> assign letter C:

> exit

bcdboot C:\Windows \s P: \f UEFI

```

完成即可，上面的分区和硬盘是符合我自己情况的

我的电脑硬盘有两个,disk0是固态，里面的2号分区是引导分区

disk1是机械硬盘，windows 的C盘安装在里面的2号分区

bcdboot的作用是将C盘下的Windows目录中与启动相关的文件复制到目标分区中，因此上面命令中P盘是给引导扇区的命名盘符