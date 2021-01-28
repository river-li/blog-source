# Cross-Version Binary Code Similarity Detection with DNN

## Abstract

二进制相似度检测有很多的应用，包括对patch的分析、代码抄袭的分析、恶意代码检测等，但是传统的方法是依靠专业技术人员的知识从二进制中提取特征的，这要么成本很高，要么效果很差，并且很少有能够对跨版本的文件进行分析的。（跨版本的文件不仅在句法结构syntactic上不同，语义上semantic差异也很大）

文中主要是设计了一个深度学习的系统用于跨版本的二进制相似度分析，主要采用了三个特征：intra-function、inter-function、inter-module

<!--more--->

## Introduction

BCSD有很多应用，甚至可以通过分析打补丁前后的文件用于分析1Day漏洞以及总结漏洞模式；更进一步的，将BCSD应用在已知的BUG和目标程序上还可以实现不同架构机器上的漏洞挖掘

但是这个技术也面临着一些挑战：

1. 同一编译器不同的优化方式得到的代码不同——cross-optimization
2. 不同编译器或使用不同算法得到的代码不同——cross-compiler
3. 同一源码在不同架构下得到的二进制不同——cross-architecture
4. 同一文件不同时期（由于patch）的二进制不同——cross-version

现有的方法一定程度可以解决前三个问题，但cross-version的代码表现都不是很好

cross-version的二进制不仅在syntactic上有所不同，在semantic上也有一些差异；

主要是现有的方法大多使用了人工提取特征，这有可能会引入一些偏差，另外有些特征的例如CFG，在代码发生些微改变时可能会剧变；这些方法大多数都只关注到了syntactic层面，很少提取到semantic层面，然而在二进制分析中，相似的文件在语义，即表达的思想、实现的功能上是高度相似的，具体的二进制函数代码只是其表现形式。

这篇文章主要想解决三个问题：

1. 如何能从二进制文件中尽可能不含偏见的提取特征
2. 如何有效的利用semantic上的特征来提高BCSD的性能
3. 如何构建一个可行的适用于cross-version文件的BCSD系统

这个系统使用的特征都是从二进制本身提取的，没有利用到相关的需要专业知识提取的特征，首先直接使用DNN从raw bytes提取函数中的特征：将二进制比特视为矩阵输入CNN，卷积得到向量作为特征

之后为了确保功能相近的函数向量距离接近，又将这个CNN用于孪生神经网络

第二，相似的函数具有相似的调用图，但是反之则不成立；因此作者对每个函数的调用图进行分析提取了inter-function特征，这里只考虑了每个节点的入度和出度，并没有具体分析，这是为了更好的性能所作的取舍

第三，相似的函数具有相似的导入函数，这一特点即使在不同架构也成立，同样反之不成立，分析导入函数得到特征称之为inter-module

## Approach

主要是介绍了三个特征的具体得到的方式，以及最终计算相似度的标准

### Intra-function

结构：8层convolutional，8层batch normalization，4层 max pooling，2层全连接

输入：100*100*1 的张量

输出：64维的向量

输入的直接是10000字节的函数raw bytes，不足的部分用0填充

防止过拟合的措施和AlexNet类似，使用了layer stacking和batch normalization

孪生神经网络部分则是成对输入，将两个函数分别输入给两个CNN，同时还输入了一个标签决定是否是相似的，这两个网络共享参数，根据两个网络的结果计算一个损失值

损失函数这里是他们自己重新定义的：

$$D1(I_q,I_t)=||f(I_q;\theta)-f(I_t;\theta)||$$
$$L(\theta) = Average_{(I_q,I_t)}{y \dot D1(I_q,I_t)+(1-y) \dot max(0,m-D1(I_q,I_t))}$$

这里的m是作为预先设置的超参数存在的

孪生神经网络的主要目的是训练CNN，得到合适的参数$\theta$是的损失最小

### Inter-function

这个特征是从CFG提取出来的，其中每个节点的入度和出度作为一个二维向量当成特征

$$g(I_q) = (in(I_q),out(I_q)) $$

这个特征距离的计算方法就是直接用了欧式距离

$$ D2(I_q,I_t) = ||g(I_q)-g(I_t)|| $$

### Inter-module

这部分的特征是从引入函数提取出来的

首先定义一个哈希函数，能够将一个集合映射为一个向量

$$ h(set,superset) = <x_1,x_2,...,x_N> $$

这个向量作为一个特征

一个二进制文件B的引入函数集合记作$imp(B)$

B的某个函数I的引入函数记作$imp(I)$

距离计算方法：

$$ D3(I_q,I_t) = ||h(imp(I_q),imp(B_q)\cap imp(B_t)) - h(imp(I_t),imp(B_q)\cap imp(B_t))|| $$

### Overall

最终两个函数之间的相似度计算公式：

$$
D(I_q,I_t)=D1(I_q,I_t)+(1- \epsilon^{D2(I_q,I_t)})+D3(I_q,I_t)
$$

这里的$\epsilon$也是一个超参数

## Thinking

文章太水了，没啥说的
