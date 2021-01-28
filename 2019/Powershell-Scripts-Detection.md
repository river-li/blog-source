# Semantic-Aware Attack Detection for PowerShell Scripts

## Abstract

近年来PowerShell脚本在APT、勒索软件、钓鱼邮件等攻击中出现的频率越来越高，然而PowerShell脚本的语言从设计时就是动态的并能够以不同级别组建脚本，基于state-of-the-art的静态分析PowerShell攻击检测天然的易受攻击混淆。

文中为了解决该问题设计了第一个高效轻量的反混淆方式。文中设计了一种新奇的基于子树的反混淆方法用于检测混淆和PowerShell抽象语法树子树层面的基于模拟的恢复技术。

基于上面的反混淆方法，可以进一步设计第一个于一层面的PowerShell脚本攻击检测系统。文中改变了传统的面向对象的关联挖掘算法并针对PowerShell脚本攻击识别新提出了31中语义上的签名。

<!--more--->

## Introduction

由于powershell在攻击中具有无文件、Windows预装等优势，越来越多的攻击都开始使用Powershell脚本。

为了检测相应的攻击，先前曾有人提出过state-of-the-art攻击检测系统，这个系统是使用静态分析匹配字符串级别的签名实现的。虽然静态分析确实更高效，覆盖代码范围更广，但是PowerShell本身的动态性特点使得运行过程中可以不同层面的产生脚本片段，这就使得静态分析的方式容易被绕过。

论文中首先有一个核心观点：***企图绕过检测系统的脚本片段在运行时需要能恢复为原本的整个脚本***，因此，针对这些存在混淆绕过技术的脚本，只要能够定位到脚本的片段就能够模拟每一段的执行并逐步重建完整的脚本。

然而问题在于如何精准的识别标识出这些片段，文中称之为*recoverable script pieces*, 如果对这些片段的检测不够精准可能最终得到的结果就是代码片段运行后的结果或是中间结果而非原脚本。

为了解决这一问题，文中提出了一个新颖的方法：基于子树的反混淆方案，这一方案的检测运行在PowerShell的抽象语法树的子树层面，而这是PowerShell脚本混淆方法的基本单元。

由于一个几KB的脚本的AST就会有几千个子树，为了实现高效的反混淆文中设计了一个机器学习的分类器来首先对子树进行初步分类判别是否是混淆了的。针对混淆过的脚本在子树中使用自底向上的方式来标识可恢复的脚本片段并模拟恢复其逻辑。

进一步构建系统的过程文中用到了OOA算法，这个算法可以自动提取出最常出现的命令和函数集合，成为OOA rules，并将其作为语义特征进行匹配，作者对大量脚本处理后新定义了31个rules作为PowerShell攻击的Signature

## Background

Powershell脚本在攻击中常用下载、执行payload、建立反弹shell、收集信息等

### "Living off the land"和无文件攻击

前者指尽量少释放文件，而使用系统自身的攻击实现的攻击

后者则是不会在硬盘上留下文件痕迹的攻击

PowerShell作为以上两种攻击的好处在于：

1. Windows自带
2. 能够轻易的访问绝大多数Windows组件并可以直接以管理员权限执行
3. 可以直接在内存中执行而不需要在硬盘上留下大量的文件，这样可以绕过许多传统的检测文件释放的检测系统

### PowerShell脚本的混淆技术

传统编程语言的混淆技术主要是逻辑结构上的，然而在PowerShell中数据和代码没有明确的界限，因此大多数混淆技术可以总结为两个步骤：

- 计算字符串，转换为代码
- 重新组合代码执行

一些常见的技术有以下内容

#### 随机化

随机化主要是对代码作一些随机改变，这些对代码的执行不起影响。

主要有：

- 空格随机化
- 大小写随机化
- 变量函数名随机化
- 随机插入会被PowerShell忽略的字符

还有一些如使用别名而不是完整的命令等

这方面的混淆只影响阅读，而对文件语法语义没有影响



#### 字符串操作

这主要是在执行时通过字符串的一些操作得到字符串

如：

> $StrReorder = "{1}{0}{2}"-f 'w-o','Ne','ject'
>
> $Strjoint = "Net.W" + "ebClient"
>
> $Url = "{9}...{26}"-f'ellE'...'s1'



#### 编码

这是实际应用最常见的混淆方式，编码后的代码只能反映很少一部分原本代码的信息



## Overview

{% asset_img overview.png overview.png %}

### Deobfuscation phase

这一过程中使用到了基于子树的方法来利用powershell的特征

将AST的子树作为混淆的最小单位，对这些子树施行复原最终构建去混淆的脚本，去混淆的脚本之后会用在训练和检测中



### Training and detection phase

反混淆之后，PowerShell脚本中的语义会被暴露出来并可以以此设计实现语义上的攻击检测系统。采用经典的Objective-oriented Association mining algorithm在数据集上，自动提取OOA rules用于特征匹配



### Application scenarios

文中的方法主要还是基于静态分析，因此相比较于动态分析具有更高的代码覆盖率，更低的开销，并且不需要对系统或解释器进行修改。

相比于现存的静态分析方案，文中的方案对混淆更具有适应力，更可解释

文中系统可以应用的场景有：

- 实时
- 大尺度自动化恶意代码检测



## PowerShell Deobfuscation

基于静态而非动态的原因：

- 动态方法需要对系统或解释器进行额外修改
- 动态方法已知的具有更低的代码覆盖度

混淆后的脚本必须能生产出背后隐藏的原脚本，这样解释器才能够正确的执行

脚本片段主要可以分成两部分，hidden original pieces和recovery algorithms

更重要的一个特点是这些片段会返回字符串类型的恢复后的代码片段，因此称前面的那些片段为recoverable pieces，后面的为recovered pieces

recoverable pieces在AST中相关的子树为recoverable subtress

只要能找到这些片段，就可以直接用嵌入的恢复算法恢复出原本的代码片段

然而在实际应用中，recoverable和脚本的其他部分并没有明显的界限尤其是代码呗多次混淆的情况下。为了解决这一问题论文中提出了基于AST子树定位可恢复代码片段，并重建原代码的方法

### Subtree-based Deobfuscation Approach Overview

{% asset_img subtree-workflow.png subtree-workflow.png %}

总体上来说分五步骤

1. 将Powershell脚本解释为AST，提取子树
2. 使用分类器找到被混淆了的片段子树，但并不是所有满足分类器的子树都可还原的
3. 使用一个模拟器来恢复原本的代码片段
4. 反混淆的片段被解释为语法树，并替换原本的子树
5. 重构整个语法树得到原本的完整代码片段

在整个过程结束后还需要进行处理去除混淆添加的多余结构

在第二步中，recoverable pieces可能是混淆片段的一部分，这种情况下如果直接尝试恢复片段，得到的结果只是一个中间过程；反过来，混淆的片段也可能只是recoverable pieces的一部分，这种情况下与第一种情况类似，直接尝试恢复也无法得到原片段。

文中用栈自底向上遍历所有可以节点，这可以避免恢复级别过高的节点。为了避免恢复的level过低，需要对输出结果进行模拟，如果输出结果不是字符串就意味着这棵子树不是recoverable的，之后就需要继续遍历以达到一个更高的level

### Extract Suspicious Subtrees

文中的系统使用了微软的API`System.Management.Automation.Language`，用来提取PowerShell的语法树。

PowerShell的语法树一共有71种节点，调用的api会返回一个语法树和一个`ScriptBlockAst`类型的根节点。一个典型的几KB大小的脚本会有几千个语法树结点，这意味着会有几千个子树，检测全部的子树会耗费很多时间

向上级节点传递节点片段只有两种方式，直接通过管道或是间接通过变量传递，因此我们只需要检测这两种类型的子树

- PipelineAst类型的子树节点
- AssignmentStatementAst节点的第二个子树

这两种类型的子树为可疑的子树，下图红色为Pipeline类型蓝色为AssignmentStatementAst类型

{% asset_img pipeline.png pipeline.png %}

{% asset_img assignment.png assignment.png %}

这样可以显著的减少需要检测的类型

基于这种思想，将AST以广度优先搜索，并将可以节点按照顺序压栈待处理



### Subtree-based Obfuscation Detection

上一步得到的可疑子树在这里输入一个二分类的分类器进行判别，即使混淆可以很好的隐藏语义信息，语法树中仍会有一些特征残留。现有的对Javascript和PowerShell的混淆检测正确率很高，因此直接使用现有的方法进行应用。

分成以下步骤：

#### 特征选择

文中参照了

> DanielBohannon.2019. PowerShell Obfuscation Detection Framework. Contribute to danielbohannon/Revoke-Obfuscation development by creating an account on GitHub. https://github.com/danielbohannon/Revoke-Obfuscation original-date: 2017-07-11T01:20:48Z
>
> MehranJodavi,MahdiAbadi,andElhamParhizkar.2015. JSObfusDetector:A binaryPSO-basedone-classclassifierensembletodetectobfuscatedJavaScript code.In 2015 The International Symposium on Artificial Intelligence and Signal Processing (AISP).IEEE,Mashhad,Iran,322–327. https://doi.org/10.1109/AISP. 2015.7123508 

两篇文章选择了四种特征

- Entropy of script pieces (代码片段熵) 
- Lengths of tokens, 取最大和最小token长度作为特征
- Distribution of AST types: 混淆过程中特定种类节点的数目会变化，文中将这个数量作为一个向量当作特征
- Depth of AST：所有的混淆的方法都对AST的深度有很大影响

上面四种总计76种特征(?)

实现时文中使用了梯度下降的逻辑回归作为分类器

计算代码片段熵的公式：

$$H = - \sum{P_ilog_2^{P_i}}$$

其中$P_i$代表第i个字符的出现频率

### Emulation-based Recovery

这一步中建立了一个powershell的session，并最终给执行这些混淆的代码片段，如果是一个recoverable的代码片段，最终过程的返回值是一个字符串类型，如果不是字符串则说明检测结果错误或者当前片段不是一个recoverable的片段，检测中无论碰到哪种情况都对这个子树进行标记并继续查找下一个子树。

由于遍历顺序是自底向上的，总会有一个高层次的子树是recoverable的且包含了当前这个子树。



### AST Update

上一步结束后需要将AST中的混淆部分进行替换，这里主要有两个步骤，首先替换recoveable subtrees为recovered subtrees；另外就是代码片段中改变的部分也需要更新。

实现时存储需要更新的子树根节点，从下向上传递。



### POST processing

 {% asset_img f7.png f7.png %}

在重新构建代码后，我们可以得到一个和原本代码具有相同语义的代码，但是语法上两者仍有差别，这些差异主要由混淆过程导致。

混淆过程中为了方便解释器理解代码引入了token，例如`("DownloadFile").invoke($url)`中`invoke`告诉解释器`DownloadFile`应该被当作一个成员函数而`$url`是其参数

同时混淆过程中还会引入一些额外的括号，因此在Post processing过程中这些语法层面上的内容会被修正



## Sematic-Aware Powershell Attack Detection 

系统总体工作流程如下：

{% asset_img sad-workflow.png sad-workflow.png %}

语义分析上的攻击检测对多态变量名具有更好的效果，同时系统的鲁棒性比较好

二进制程序分析中经常使用各种图、控制流图、依赖图等而不是API集合来表现语义特征，这是由于API集合只包含低层次的语义，比较容易误判

然而在PowerShell中API集合具有很高层次的语义，这是由于Powershell的API可以很直接的访问系统功能，同时考虑到使用集合标识特征比图具有更好的性能，文中使用了API集合作为特征。

整体的步骤可以分为training和detection两个部分

### Training Phase

首先提取前面工作得到的节点并对其归一化，主要包括：

- 转换为小写
- 删除武关子夫
- 检查别名

#### OOA

{% asset_img ooa.png ooa.png %}

提取OOA rules的方法需要两个步骤：首先使用FP-growth算法生成频率模式，之后选择计算符合要求的模式即分数大于设置的阈值的模式。特别的support标识恶意的可能性，而confidence标识正常的可能性

$$ support(I,Obj) = \frac{count(I \cup \{Obj\},DB)}{|DB|}$$

$$ support(I,Obj) = \frac{count(I \cup \{Obj\},DB)}{count(I,DB)}$$

这里I表示指令的集合，函数$count(I \cup \{Obj\},DB)$返回在DB中$I \cup \{Obj\}$出现的次数
