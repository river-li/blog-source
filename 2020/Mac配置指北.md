最近换了一台Mac，很开心，但是Mac还是有很多地方用的不习惯，在这里记录一下各个软件完整的配置过程


<!--more-->


## Basic

首先是对Mac的基本设置，我主要设置了一下触控板，还有键位，以及安装了包管理软件brew

### Touchpad

都说Mac的触控板好用，虽然看着这么大一块确实应该会挺舒服，但是在用之前我并没觉得可能会多好用，因为三指、四指的功能我都可以在Linux上面实现，（可以参照我[之前的文章](https://river-li.me/2019/07/06/pei-zhi-yi-ge-mac-yi-yang-de-hong-mo-ban/)）而且最近Windows也开始对触控板的功能进行优化了。

但是还是没想到，用过之后感觉：这大概是笔记本上能做到的极致了，Linux怎么配置也赶不上的。

Mac本身的触控板功能就很强了，不过还是有一些需要自己设置的地方

比如拖拽，可以设置拖拽锁定、或者三指拖拽。虽然锁定更符合原本的Windows和Linux使用习惯，但是我还是在这里设置成了三指拖拽，适应了一段时间之后发现这个功能真的是触控板上最好用的功能了。

还有一些动作需要自己设置打开，默认是关闭的，主要是多指的复杂操作。



### Keys

这是一个改键位的软件[karabiner](https://pqrs.org/osx/karabiner/)

Mac的键盘布局还是传统的qwer键盘，并且没办法定制成colemak之类的键盘；因此为了比较大限度的提高键盘利用效率，还是需要对键盘上的键进行一些修改。

看之前的博客也知道我比较喜欢使用Vim，这样做就需要频繁的用到ESC。虽然MBP16恢复了ESC键，但是按起来还是没那么容易，因此直接将ESC、Caps这两个键进行了调换

![k-element](https://static.hack1s.fun/images/2021/02/08/k-element.jpg)

软件本身还支持很多高级的改键功能，但是目前还没有这方面的需求，就只使用了基础的改键



### Homebrew

homebrew 是Mac上的一款包管理程序，安装这个的原因是网上大多数第三方的程序都推荐使用这个，因此我也装了这个

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装brew的时候发现速度变得很慢，默认的repo源在github上，改成国内的源可以提升速度，执行下面的几个命令

```bash
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
cd "$(brew --repo)/Library/Taps/homwbrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
```

之后速度就变快了，下载软件时可以类似这样

```bash
brew install wget
# brew install <package>
```

但是，brew只能安装终端内的程序，想要安装图形界面程序需要借助一个扩展`brew cask`

安装cask首先需要执行这个命令

```bash
brew tap caskroom/cask
```

整体上这个工具需要依赖MacOS上的XCode，由于我本身也想做一些IOS开发之类的东西，也直接在App Store 上下载了XCode



在安装第三方的图形界面程序时可以使用cask进行搜索和下载

```bash
brew cask install atom
```



### Touchbar

新版的MacbookPro全线配置了Touchbar，这个说鸡肋也鸡肋但是有时候确实也觉得有点意思。

为了最大程度的增加一些价值，还是装了一些东西来

拓展的一个程序叫做[pock](https://pock.dev/)，同时还有一款类似的软件[TouchSwitcher](https://hazeover.com/touchswitcher.html)

对比了一下还是Pock的自定义程度比较高，用起来也比较好用

但是其实这些工具都有一个缺点，就是在使用的时候一些原本touchbar功能比较好用的程序也用不了了

所以最好还是能有一个可以切换的开关，比如pock可以双击control切换系统栏和自定义栏，很方便

![pock](https://static.hack1s.fun/images/2021/02/08/pock.jpg)



## Terminal

Mac OS是一个基于BSD的系统，和linux很相似的终端同时还有很多常用的软件；

### iterm2

Mac自带的终端不是太好用，其实也还好，但是可以自定义的功能还是比较少

iterm2是一个功能比较丰富的、可自定义的免费终端，基本可以说是最好的终端了

iterm可以直接在[官网](https://www.iterm2.com)下载

下载下来，安装之后进行配置

#### Color

首先要换一个好看的配色，按 `command+,`

![iterm-color](https://static.hack1s.fun/images/2021/02/08/iterm-color.jpg)

在profile -> colors 这里右下角可以看到一个选条，选择`Visit Online Gallery`

在网上可以找到自己喜欢的配色，下载之后双击安装即可

我这里安装了一个`Snazzy`的配色



#### Font

虽然Mac默认的字体还算好看，但是还是看习惯了Linux的开源字体，直接换一个字体来

```bash
brew tap caskroom/fonts
brew cask install font-hack-nerd-font
```

之后在iterm的preference处修改字体即可



#### Hotkey

快捷键可以针对不同的profile进行设置，我仿照linux下常用的\<C-A  T\>

设置了开启终端的快捷键

![iterm-hotkey](https://static.hack1s.fun/images/2021/02/08/iterm-hotkey.jpg)



### zsh

在终端中除了终端本身就是shell了，我这台mac自带的shell是bash，希望配置好zsh

#### oh-my-zsh

首先安装了oh-my-zsh,这部分很简单，执行官网的命令就ok了，但是我这里出现的问题是联网的时候访问github速度非常慢，实时下载运行可能会出错，因此直接下载下来了脚本再执行

```bash
wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh
```

下载下来再执行就好了



#### powerlevel10k

安装完zsh之后对其的主题进行配置，直接到[powerlevel10k](https://github.com/romkatv/powerlevel10k)按照教程进行安装

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
```

之后在zshrc中修改这么一个变量

```bash
ZSH_THEME=powerlevel10k/powerlevel10k
```

完成之后重启一个shell

执行`p10k configure`

之后按照提示选择自己的喜欢的终端风格即可



#### plugins

我们[之前](https://river-li.me/2018/12/09/zsh/)也讲过zsh有很多厉害的插件，因此换了新电脑肯定不能不装

主要安装了这么几款

- git
- extract
- zsh-syntax-highlighting
- autojump
- zsh-autosuggestions

比较值得注意的地方在于，语法高亮的插件必须放在最后一个

安装方法大体上都一样，这里就省略了



### vim

Vim 是我很喜欢用的一款终端里的编辑器，主要是学会了一些高级的用法之后实在是方便的很，具体的配置过程就还是略去不写了，可以参照之前的[文章](https://river-li.me/2018/12/11/vim-tai-qiang-liao/)



## Development

这肯定是买电脑的主要原因之一，学习iOS和MacOS的开发

但是同时还会经常有一些别的东西需要写，都整理一下

总共安装了这些

- Pycharm
- Webstorm
- VS Code
- Virtualbox
- X Code
- Parallels Desktop



前面的一些都很常规，最后两项是Mac独有的软件，目前还没有体验X Code

但是Parallels Desktop确实是比较好用的虚拟机，尤其是增强功能方面，安装的过程也非常快，直接一键就可以安装完成

但是另一方面，在做靶机或者vulnhub的时候还是经常用到virtualbox和vmware这样的软件因此也不能代替


## Tools

这一类都是一些小工具，但是也都是很实用有趣的那种

有一些可以配置的很强大，我也稍微配置了一些

### 截图

是的，这个软件名字就叫做截图，在App Store安装的，不知道是Mac上没有截图工具还是怎么样，我就直接下载了这个软件，可以设置截图的快捷键也可以直接录屏


![jietu](https://static.hack1s.fun/images/2021/02/08/jietu.jpg)


### RunCat

这是一个在扩展栏的状态显示小工具，可以显示当前电脑的CPU、内存占用等信息

我主要是用来看内存和网速的

同样可以直接在App Store找到

![runcat](https://static.hack1s.fun/images/2021/02/08/runcat.jpg)



### CheatSheet

这个程序是一个快捷键的列表

由于刚刚用Mac，很多快捷键都不太熟悉，很多时候都不知道快捷键是啥

而且键盘布局也和windows的键盘有点不同，所以有点不习惯

在任何一个程序下长按`command`键就可以弹出一个快捷键的清单，其实效果有些像iPad外接键盘的效果

下载时在App Store没有找到，官网好像也挂了

我直接google找到了一个[下载](https://cheatsheet-mac.en.softonic.com/mac)链接

![cheatsheet](https://static.hack1s.fun/images/2021/02/08/cheatsheet.jpg)



### V2rayU

Emmm...这个就不用介绍了吧？功能大家都懂得



### Mate

这是一个翻译的软件，设置快捷键后可以直接翻译剪贴板里的单词或者句子，有时候还是挺好用的，App Store有售

![mate](https://static.hack1s.fun/images/2021/02/08/mate.jpg)



### Lastpass

全平台的免费密码管理程序

想要安装的话最好安装Safari的扩展版本

但是这个版本在国内的苹果商店找不到，我最后也是网上搜到的程序



### Dozer

上面安装了这么多程序，加上平常用的QQ微信什么的，顶部的状态栏应该已经很满了吧？这时候就需要用到这个工具了，可以一键隐藏其他的图标

直接在[官网](dozermac.com)可以下载到

对比一下效果

![dozer1](https://static.hack1s.fun/images/2021/02/08/dozer1.jpg)

![dozer2](https://static.hack1s.fun/images/2021/02/08/dozer2.jpg)



### Alfred

最后要介绍的就是两个比较复杂的工具了

一是这个Alfred，苹果商店虽然能下载到但是版本非常的老，最好去[官网](https://www.alfredapp.com/)

其实我觉得苹果自带的spotlight就挺好用的了，只是不支持自定义的功能

#### General

首先修改一下快捷键，调整成按起来比较方便按出来的

在Windows下的Wox我都是习惯ctrl+space的

但是Mac的键盘左下角小拇指按到的键是Fn，按到ctrl的话不得不说是十分的难受

因此我直接改成了双击command



另外就是需要修改一下location

这里最好选择为自己梯子的物理位置，这样访问Google会更快一点

高级功能需要购买专业版才可以自己定制，不过专业版的价格还是有点贵

25欧元，贵的不得了



#### Features

这部分里面可以设置的内容很多

##### File Search

首先是文件搜索

通过关键词find 可以查找文件，in可以查找文件内容，open可以直接打开

tags功能我觉得比较鸡肋，就直接关闭了



这部分主要设置开启了Fuzzy，并且设置了一下快捷键

![file](https://static.hack1s.fun/images/2021/02/08/file.jpg)



##### Action

这部分的功能是可以设置两个键，在搜索到选项的文件时按相应的键可以直接将其送到Action的页面，在那个页面可以对文件选择进一步的操作

![action](https://static.hack1s.fun/images/2021/02/08/action.jpg)



##### Web Search

Web的功能可以说是Alfred使用到的最多的功能了，因此这部分需要好好设置一下

首先关闭所有的默认选项，因为大多数都用不到

我只保留了Google的几个服务，另外添加了bilibili和baidu



#### Workflow

链接: https://pan.baidu.com/s/1LYpb_YpL_TQGy6IFpDgcQQ 提取码: mmnv

这里面有一些现成的workflow，都还比较有用

另外我还找到了一个v2ex的workflow，但是安装上之后刷新不出来消息，目前还不知道具体原因

![workflow](https://static.hack1s.fun/images/2021/02/08/workflow.jpg)



### HammerSpoon

这个软件可以支持用lua编写脚本自定义，功能很多也很复杂，目前还没玩透，准备学一学lua之后专门看一下

