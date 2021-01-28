用于记录一些有用的vim快捷键

会根据我自己的掌握进度定期更新

vim主要有三种模式：normal, insert, visual

<!--more--->



## Normal Mode

### 移动

hjkl 左上下右移动

w\e 向后移动一个词

b 向前移动一个词

L\H\M 光标移动到当前页面最底端\顶端\中间

% （光标本身在括号上时）移动到另一端括号

$ 移动到该行末尾

0 移动到当前行首

^ 移动到行首第一个非空字符

\+ 移动到下一行的开始第一个字符

\- 移动到上一行开始第一个字符

\<C-d\> 向下半屏

\<C-u\> 向上半屏

下面三个分别对应三种移动方式, section、paragraph、sentence

- {[[ ]]
- { }
- ( )



### 操作

i 在当前字符进入插入模式

a 在当前字符后一个字符进入插入模式

x 删除当前字符

s 删除当前字符并进入插入模式



I 光标移动到当前行行首并进入插入模式

A 光标移动到当前行行尾并进入插入模式

o 在当前行下面新建一个空行并进入插入模式

O 在当前行上面新建一个空行并进入插入模式

p 粘贴到当前行下方

P 粘贴到当前行上方

u 撤回操作(undo)

\<C-r\> 取消撤回(redo)



yy 复制当前行

dd 删除当前行



\<C-n> \<C-p> 自动补全

. 重复上一个操作

f/F/t/T 行内搜索，只能接受一个字符为参数

多行操作：

[数字]+{y/p/d]



## 高级用法

### 代码折叠

代码折叠需要设置折叠方式

```bash
set foldmethod=manual
```

其他方式还有marker、indent等

在manual模式下，折叠代码时使用

zf 创建一个折叠，后面可以接行数，也可以按照光标移动方式接参数

zd 删除这个折叠

一般情况下，例如写C的代码，可以用`zf%`折叠整个块结构



### 寄存器

vim 有一个特性，删除以及复制的内容会保存历史，按顺序九个记录依次保存在数字寄存器中，如果需要粘贴寄存器的内容可以

`"1p`含义为粘贴一号寄存器的内容，等价于`p`

如果要搜索寄存器中的内容，可以`"1pu.u.u`这样会逐渐递增寄存器标号

同时，26个字母也是可以使用的寄存器，如果要保存在26个字母中，只需要

`"ayy`即可复制保存在a寄存器

如果使用大写字母，则会将其以append方式附加在寄存器中

`"Ayy`不覆盖a寄存器中的内容，在其后面继续添加内容

### 书签

vim还有一个书签功能，可以设置26个书签

`ma`即将目前所在位置标记为a

之后可以快速回到这个位置，只需使用`'a `或`a





## Instruction

:set nu 显示行号

:set nonu 不显示行号

:set relativenumber 设置显示相对行号

:set ts=4 设置tab的宽度为4

:open \<file> 打开文件

:x 保存退出

:qa! 不保存退出

:[addr]s/源字符串/目的字符串[options]



vi 可以说是ex 编辑器的升级版，ex编辑器是一个没有图形化的编辑器，纯命令行的交互，而vi、vim也都完全支持了ex的命令

这作为命令模式保留在了vi中

例如替换的命令`:s/Hello/hello`即可将Hello替换为hello

删除行的命令`:13,23d`

移动`:220,231m2`将220行到231行的内容移到第二行下面

`:/pattern/=`显示pattern第一次出现的行号

ex模式中`.`代表当前行,`$`代表最后一行,`%`代表每一行

`:20r file` 读取file，并写在当前这个文件的20行处

`:e filename` 在编辑这个文件时打开另一个文件，编辑另一个文件结束后返回当前文件

`:e!` 回到先前编辑的文件

当命令行启动跟着多个参数时，`:n`切换到下一个文件

之后可以用`<C-^>`切换buffer

在编辑多个文件时也可以使用寄存器保存复制和删除的内容，在多个文件间粘贴

### 匹配和替换

源字符串和目的字符串可以用正则表达式

addr:

- %全局
- 1,20 一到二十行
- .,$ 当前行到文件尾
- 空着默认只有当前行

options:

- g 全局
- c 每一个替换时进行确认
- p 结果逐行显示

搜索中使用的正则表达式和python中语法相同：

`^开始； $结束；*重复0次到多次；[]一个字符；（）一个表达式`





## Vimrc

vimrc是vim的配置文件，可以设置每次启动vim时执行的命令

例如上面instruction的一些内容可以直接写在vimrc中使得每次启动自动执行

不过vimrc最重要的一个功能还是按键的映射，可以将按键映射为不同的功能和指令

首先我们设置一下LEADER键

```bash
let mapleader=" "
```

可以设置leader键为空格

之后可以通过map设置一些用空格触发的快捷键

## 插件

vim的插件属于更高级，更个性化的用法，在熟悉了前面的基础操作后逐步增加插件

### vim-plug

vim-plug是vim的一个插件管理插件；

安装方法:

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

安装完成后需要在`.vimrc`中添加这样的内容：

```bash
call plug#begin('~/.vim/plugged')
	Plug '...'
call plug#end()
```

在需要安装内容时将`...`替换为需要安装的插件的github上repo链接即可

只需要链接的后半部分，例如：

`github.com/garbas/vim-snipmate.git`只需要写`garbas/vim-snipmate`即可



设置好vimrc后在启动vim时使用`:PlugInstall`就会自动安装

需要更新插件时`:PlugUpdate`，更新`vim-plug`本身`:PlugUpgrade`



### NerdTree

NerdTree可以作为侧边栏出现文件树，方便打开同一个工程内的其他文件

repo位置`scrooloose/nerdtree`

在vimrc中加入

```bash
map <LEADER>f :NERDTreeToggle <CR>
```

可以将打开关闭树的功能与LEADER加f键绑定
