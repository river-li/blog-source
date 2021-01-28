tmux是一个终端分屏的工具
在ssh这样的远程访问时也可以使用
因此在纯图形化下很实用
但是这也是一个十分复杂的工具，快捷键多的让人头皮发麻
<!--more--->


## 配置默认启动zsh

在`~/.tmux.conf`内加入如下内容

```bash
set -g default-shell /bin/zsh
set -g default-command /bin/zsh
```

这样启动tmux时会默认启动zsh而不是bash



## 配置使用vim快捷键

tmux的移动很麻烦，我也不知道是不是emacs的移动方式

不过很难记，就想要直接配置成vim的快捷键

在`~/.tmux.conf`加入这些内容：

```bash
set-window-option -g mode-keys vi
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
```

