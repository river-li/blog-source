在运行python写的程序时发现无法`import tkinter`,会报错缺少一个静态链接库

`libtk8.5.so`

解决办法：
```bash
pacman -S tk
```

