添加源的时候一般是

```bash
sudo add-apt-repository ppa:user/ppa-name
```

对应删除的时候加一个`-r`就可以

```bash
sudo add-apt-repository -r ppa:user/ppa-name
```

或者也可以直接删除`/etc/apt/source.list.d/`下面的内容

