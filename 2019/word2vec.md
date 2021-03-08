word2vector是将词转换为词向量的方法，能够把自然语言转换为向量

使用数学方法进行描述


<!--more-->


## 理论

常用的词向量表示方法有

- one-hot representation
- distributed representation

前者是用一个很长的向量表示一个词，一个向量只有一个维度为1

每个词语的维数就是语料库字典的长度



后者的基本思想是：相同语境下出现的词语义也应该是相近的

使用一个固定长度的稠密词向量标识



word2vector是通过神经网络机器学习的算法来训练N-gram语言模型，并在训练过程中求出对应的词向量的方法



## 编程

word2vector可以在tensorflow中调用

也可以用gensim中的包

这里用gensim

```python
from gensim.models import word2vec
sentences = word2vec.Text8Corpus('1.txt')
model = word2vec.Word2Vec(sentences,size=200)
```

可以使用`model.similarity('word1','word2')`

得到两个词的相关度值

`model.most_similarity('word1',topn=20)`

可以得到与word1相关度最高的20个词



`model.doesnt_match(list)`

这个方法会返回出list中最不合群的词

`model.save(filename)`

`word2vec.Word2Vec.load(filename)`

以上两个方法可以将训练的模型保存为文件，或是直接从文件读取模型



### 实际例子

首先需要在网上获取到中文语料库，数据来源是搜狐的新闻

![1](https://static.hack1s.fun/images/2021/02/24/1.png)

这里一共有1990个文件，编号从10到1999，每个文件中为一段新闻

例如10.txt：

![2](https://static.hack1s.fun/images/2021/02/24/2.png)

对于获取到的语料库首先应该用jieba进行分词，将每个词用空格隔开

关于[jieba](http://river-li.me/)分词的用法和分词的理论模型可以点击链接查看；

整个脚本如下:

```python
import jieba
prefix = './data/word/'
affix='.txt'
with open('result_dict.txt','a') as f:
    for i in range(10,2000):
        s=''
        with open(prefix+str(i)+affix,'r',encoding='gb18030') as fp:
            l=fp.readlines()
            for j in l:
                s=s+j.replace('\u3000','')
            result=jieba.lcut(s)
            for r in result:
                f.write(r)
                f.write(' ')
```

这里将所有的文本文件进行分词后写入到`result_dict.txt`文件中，并且要在每个词后面写入一个空格

运行完毕后效果如图

![3](https://static.hack1s.fun/images/2021/02/24/3.png)

之后使用gensim库中的word2vec来训练生成词向量

为了了解生成词向量时的进度，导入logging这个库进行

```python
from gensim.models import word2vec
import logging
logging.basicConfig(format='%(asctime)s:%(levelname)s:%(message)s',level=logging.INFO)


sentences = word2vec.Text8Corpus('./result_dict,txt')
model = word2vec.Word2Vec(sentences,size=200)
#size是神经网络中隐藏层中神经元的个数
#model.save('word.model')
```

训练的过程如下

![4](https://static.hack1s.fun/images/2021/02/24/4.png)

完成

整个训练过程可以看作是一个三层的神经网络的训练过程，有一个隐藏层，其神经元个数就是生成Word2Vec对象时的参数size大小

下面测试几个自带的方法；

查看与"北京"这个词意思最接近的词：

![5](https://static.hack1s.fun/images/2021/02/24/5.png)

可以发现有的是地名有的是职位；

这是由于这个模型的基本思想就是，意思相近的词出现在相似语境下的概率更大；

因此与北京出现的位置相近，或经常与它同时出现的词就被认为是他的同义词了。



比较两个词的词向量距离：

![6](https://static.hack1s.fun/images/2021/02/24/6.png)

可以看到本报和记者的距离比本报和警察的距离近很多

这是由于前者经常同时出现，被判定为含义接近的缘故
