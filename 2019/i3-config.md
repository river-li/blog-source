最近在配置i3,对着官网的配置文件过了一遍，写一下看到的内容

<!--more--->

## i3bar

### 显示模式

bar 一共有三种模式(mode)

- hide 只有按下mod键时才会显示
- dock 默认，常驻
- invisible 不显示

显示的位置可以在底部也可以在顶部(position)

- top
- bottom



### 鼠标交互

鼠标交互主要分为左键、中键、右键点击以及滚轮上下滚动

分别是button[1/2/3/4/5]

例如

```
bar {
    # disable clicking on workspace buttons
    bindsym button1 nop
    # Take a screenshot by right clicking on the bar
    bindsym --release button3 exec --no-startup-id import /tmp/latest-screenshot.png
    # execute custom script when scrolling downwards
    bindsym button5 exec ~/.i3/scripts/custom_wheel_down
}
```

这里设置的是右键会执行后面的命令，下滑滚轮会执行最下面的脚本



### 分隔符

`separator_symbol “”`

在引号之中输入分隔符即可



### 工作区

工作区可以设置是否显示,workspace_button设置为no则会不显示

另外两个选项用于设置对于`[n]:[NAME]`这样的工作区是显示数字还是名字

```
workspace_button yes
strip_workspace_numbers yes
strip_workspace_name no
```





