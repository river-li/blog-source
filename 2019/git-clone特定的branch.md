想要找一个好看的建立模板，找到github上的一个repo:https://github.com/billryan/resume.git

但是中文的简历模板不在master的分支里而是单独的一个分支，稍查了一下用法:

```bash
git clone --single-branch --branch zh_CN https://github.com/billryan/resume.git
```

加上这个参数`--single-branch`，就可以clone特定的branch了

