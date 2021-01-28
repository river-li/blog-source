又到了喜闻乐见的安装显卡驱动的时候了，之前不知道多少次因为这个开不开机

不过这次，似乎有点进步，顺利搞定

<!--more--->

### 步骤

1. arch的源要开启multilib，具体位置在`/etc/pacman.conf`

2. 安装相关的包:

   ```bash
    pacman -S bumblebee bbswitch nvidia opencl-nvidia lib32-nvidia-utils lib32-opencl-nvidia mesa lib32-mesa-libgl xf86-video-intel
   ```

3. 添加用户组：

   ```bash
   sudo gpasswd -a [username] bumblebee		//方括号内替换为自己用户名
   ```

4. 启动服务

   ```bash
   sudo systemctl enable bumblebeed.service
   ```

5. 修改配置文件,路径 `/etc/bumblebee/bumblebee.conf`

   ```yaml
   Driver=nvidia
   [driver-nvidia]
   PMMethod=bbswitch
   ```

6. 重启



至此安装成功

如果成功安装没有出错，使用命令`nvidia-smi`可以检测是否启动显卡

需要用显卡运行软件时使用`optirun`命令

---
打脸后续
重启之后又出问题了，不过由于这次bbswitch是后面单独安装的，因此怀疑导致问题的是这个包
用启动盘chroot后卸载了bbswitch后就没问题了