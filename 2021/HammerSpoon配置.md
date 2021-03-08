## Intro

HammerSpoon能实现的功能蛮多的，按照目前看到的一些资料，试着自己写一下配置

目前电脑上开着的乱七八糟后台程序挺多的，有一点占用内存

想把这些的功能都用hammerspoon替代掉

目前电脑上始终在后台开着的程序有：

- cheetsheet偶尔用来查快捷键
- alfred频繁用来开APP、查看剪贴板，偶尔查单词、OCR 
- clash/ssr/v2ray用来上网
- uPic用来上传typora图片到图床
- Karabiner用来改键，其实就只改了`ESC`和`Tab`，还有就是针对蓝牙键盘把`option`和`command`位置换了一下
- BetterTouchTool用来自定义触摸板和TouchBar
- iShot用来截图



还有一些偶尔发生的特殊场景才需要打开的APP

- 偶尔用Klib导出Kindle的笔记
- 看完书、电影后登录豆瓣作标记
- 偶尔用Calibre转换电子书格式
- 插入特殊格式的硬盘后用Mounty重新挂载



主要想用HammerSpoon实现的功能包括：

- 类似i3的窗口管理方式
- 根据Wi-Fi的SSID自动挂载远程的硬盘
- 替代掉cheetsheet
- 替换掉改键的Karabiner
- 替换掉截图的iShot



根据最近查资料看到的一些效果可以做出的一些尝试：

- 结合Spotlight尝试替换掉Alfred
- 自动备份某个git repo的内容到nas
- 第二天天气异常时提醒、提醒日程等
- 结合Huginn在一些博客、RSS微信公众号更新时提醒
- 结合一些文本聊天机器人定制一个语音助手
- 结合1focus实现一个带专注效果的番茄时钟
- 通过USB在插上Kindle时自动导出笔记
- 隔一段时间提示喝水
- 根据用的是显示器还是Macbook自己的屏幕设置Virtualbox的缩放倍率
- 随机数生成器，偶尔用来生成密码
- 网易云里把在播放的歌添加到播放列表



一点一点试着实现一下吧，比较核心的就是窗口管理，先实现这部分，剩下的就锦上添花了



## 窗口管理

i3的窗口管理其实切分一下功能主要是这样几部分：

1. `<Mod>+h/j/k/l`切换焦点
2. `<Mod>+1/2/3`切换虚拟桌面
3. `<Mod>+Shift+1/2/3`将窗口移动到虚拟桌面
4. 新建窗口时自动将原本的窗口缩放并分屏



首先最容易实现的是焦点的切换

类似i3里面的切换方式`<Mod>+h/j/k/l`来切换焦点到左下上右边的窗口

可以直接使用`hs.window:focusWindowEast`这样的API

直接绑定在对应的hotkey上就可以了

```lua
hs.hotkey.bind(
{"alt"},
"L",
function()
local window = hs.window.focusedWindow()
window:focusWindowEast()
end
)
```

类似这样的方式重复四遍，键位换一下就可以了；

