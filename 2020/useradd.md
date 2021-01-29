linux的useradd指令可以添加一个新的用户，在添加用户时也可以将其添加到指定的组

在渗透过程中可能需要创建一个新的特权级用户，记录一下创建的方法



通过命令`groups`可以查看当前用户所属的组别

如果希望添加用户到某一个组，可以使用

```bash
usermod -a -G groupA user
```

这样可以将用户`user`添加为`groupA`的一员

这里的`-a`选项是必须的，如果没有这个参数，即

```bash
usermod -G groupA user
```

这样的命令会将用户`user`加入到`groupA`中，但是同时会从当前其所在的组中将其删除



创建一个新的用户，并将其添加到特权级的指令

```bash
useradd -m -g users -G wheel -s /bin/bash username
```

这样就添加了一个名为`username`的特权级用户

