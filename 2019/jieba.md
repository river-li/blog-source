jieba是一个python的中文分词库，在数据处理中用处比较大

这次主要介绍分词的原理和库的用法

<!--more--->

## 理论

汉语的分词方法主要有三种：

- 基于字典、词库的匹配分词方法
- 基于词频度统计的分词方法
- 基于知识理解的分词方法

jieba分词使用的就是第二种，基于统计的原理；

采用了动态规划查找最大概率路径找出基于词频的最大切分组合；

对于未登录的词使用了基于汉字成词能力的HMM模型，和Viterbi算法；

基于前缀词典实现高效的词图扫描，生成句子中汉字所有可能成词情况所构成的有向无环图；

jieba分词有三种分词模式，

- 全模式
- 精确模式
- 搜索引擎模式

全模式中会将一个字串分成最小单位的词；

精确模式是默认的分词模式，是对句子的一个概率最佳分词；

搜索引擎模式会将长词再拆分成短词，以关键词的形式出现；



## 编程

### 分词方法

```python
cut(self,sentence,cut_all=False,HMM=True)
```

`cut_all`这个参数是标识是否开始全模式分词的参数

默认状态为精确模式，因此为False

```python
cut_for_search(self,sentence,HMM=True)
```

两种分词方法返回的结构都是一个可迭代的generator，可以通过for循环直接得到分词后的每一个词语（Unicode）

另外还有两个`lcut`和`lcut_for_search`直接返回列表



### 自定义字典

`load_userdict(file_name)`

由于jieba是基于统计的分词方法，因此用户可以自己加载自定义的字典

字典的格式要求：

- 一个词占一行
- 每一行分三个部分：词语、词频、词性
- 用空格隔开
- 词频和词性可以省略

调整词典的方法

```python
add_word(word,freq=None,tag=None)
del_word(word)
suggest_freq(segment,tune=True)
```

第三个方法作用是调整单个词语的词频

例如`台中`这个词，有可能被jieba的词典切开，但是这在某些句子中应该作为一个整体，则可以使用

```python
jieba.suggest_freq('台中',True)
```

使台中作为一个整体的频率提升，之后就不会被切分出来了



### 关键词提取

#### TF-IDF统计方法

字词的重要性随着它在文件中出现的次数成正比，但是会随着它在语料库中出现的频率成反比

例如,`的地得`在文件a中出现频率很高，但是这三个字在整体的语料库中频率也很高，因此其重要性比较低

```python
jieba.analyse.extract_tags(sentence,topK=20,withWeight=False,allowPOS=())
```

sentence是待提取的文本

topK是返回TF/IDF权重最大的关键词的数量

withWeight为True时会一并返回关键词的权重值

allowPOS为True时仅包括指定词性的词

```python
jieba.analyse.set_stop_words(filename)
```

这个方法可以将停止词的词典替换为自定义的词典



#### TextRank

这种算法也是一个文本排序的算法，来源于搜索引擎的排序算法`PageRank`

主要是使用矩阵迭代的代数知识对每个文本词语进行分析，最后得到数值最大则重要性越高

```python
jieba.analyse.textrank(sentence,topK=20,withWeight=False,allowPOS=('ns','n','vn','v'))
```

```python
jieba.analyse.TextRank()
```

新建一个TextRank实例



### 词性标注

```python
jieba.posseg.POSTokenizer(tokenizer=None)
```

新建一个自定义分词器，标注句子中每个词的词性，采用和ictclass兼容的标记法

例子：

```python
import jieba.posseg as pseg
words = pseg.cut("我爱北京天安门")
for word,flag in words:
    print(word)
    print(flag)
```

