VIm编辑Python文件时的函数定义跳转

需要使用ptag对项目文件处理生成一个tag

```bash
wget http://svn.python.org/projects/python/trunk/Tools/scripts/ptags.py
chmod +x ptags.py
```

在项目的根目录运行

```bash
find  . -name \*.py -print | xargs ./ptags.py
```

会在根目录生成一个tags文件



之后在vimrc中加一个`set tags=tags`就可以了

默认在遇到定义的时候可以用`<C-]>`跳转到定义

之后可以用`<C-t>`跳回来



在vimrc里面加入

```
map J <C-]>
```

这样可以直接在定义按shift+j跳转，感觉舒服一点

