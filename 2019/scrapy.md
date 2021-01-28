scrapy是一个爬虫的框架,比较适合对整个网站进行爬取
里面也有很多提前设置好的内容,比如多线程、请求Header等
这里记录一下相关的概念以及用法


<!--more-->


## 基本概念

框架结构

- Scrapy
- Scheduler 调度器
- Downloader 下载器
- Spider 爬虫
- Pipeline 管道，主要负责处理爬虫从网页抽取的实体

数据类型

- Request类
- Response类
- Item类  spider从页面中提取的信息对象，由pipeline处理

scrapy运行时直接在命令行里运行，可以像一个可执行文件一样用

只需要编辑爬虫project中的源码再用scrapy运行即可

|     命令     |     说明     |                      格式                      |
| :----------: | :----------: | :--------------------------------------------: |
| startproject |   创建工程   |       scrapy startproject \<name\> [dir]       |
|  genspider   |   创建爬虫   | scrapy genspider [options] \<name\> \<domain\> |
|   settings   | 获得配置信息 |           scrapy settings [options]            |
|    crawl     |   运行爬虫   |            scrapy crawl \<spider\>             |
|     list     | 列出所有爬虫 |                  scrapy list                   |
|    shell     |   启动调试   |               scrapy shell [url]               |



## Xpath

Xpath可以使用路径表达式在XML中选择节点

| 表达式   | 描述                                             |
| -------- | ------------------------------------------------ |
| nodename | 选取该节点的所有子节点                           |
| /        | 从根节点选取                                     |
| //       | 从匹配选择的当前节点选择文档中的节点，不考虑位置 |
| .        | 选取当前节点                                     |
| ..       | 当前节点的父节点                                 |
| @        | 选取属性                                         |

如果想要单独用xpath的功能

```python
from lxml import etree
html = etree.HTML(response.content)
```

在匹配时可以将属性放在中括号中，也可以像操作数组一样选择特定序号的节点



## 实例

### 豆瓣电影top250

第一个例子，爬去豆瓣电影的排名前250

![startproject.png][1]

创建完成后可以看到整个文件的目录结构如图

```tree
. 
├── douban
│   ├── __init__.py 
│   ├── items.py
│   ├── middlewares.py
│   ├── pipelines.py
│   ├── __pycache__ 
│   ├── settings.py
│   └── spiders
│       ├── __init__.py
│       └── __pycache__
└── scrapy.cfg
```

用浏览器找到前250的网站，每一页是10个，一共25页

![0.png][2]

```html
<div class="item">
   <div class="pic">
         <em class="">1</em>
         <a href="https://movie.douban.com/subject/1292052/">
                  <img alt="肖申克的救赎" src="https://img3.doubanio.com/view/photo/s_ratio_poster/public/p480747492.jpg" class="" width="100">
         </a>
    </div>
    <div class="info">
        <div class="hd">
            <a href="https://movie.douban.com/subject/1292052/" class="">
                <span class="title">肖申克的救赎</span>
                <span class="title">&nbsp;/&nbsp;The Shawshank Redemption</span>
                <span class="other">&nbsp;/&nbsp;月黑高飞(港)  /  刺激1995(台)</span>
            </a>

            <span class="playable">[可播放]</span>
        </div>
        <div class="bd">
              <p class="">导演: 弗兰克·德拉邦特 Frank Darabont&nbsp;&nbsp;&nbsp;主演: 蒂姆·罗宾斯 Tim Robbins /...<br>1994&nbsp;/&nbsp;美国&nbsp;/&nbsp;犯罪 剧情</p>

              <div class="star">
                    <span class="rating5-t"></span>
                    <span class="rating_num" property="v:average">9.6</span>
                    <span property="v:best" content="10.0"></span>
                    <span>1429179人评价</span>
              </div>
              <p class="quote">
                  <span class="inq">希望让人自由。</span>
              </p>
         </div>
   </div>
</div>
```

检查元素可以看到每一个container的结构，在爬去时我们希望获取到

- 排名
- 标题
- 评分
- 描述
- 评价人数

例如：1、肖申克的救赎、9.6、希望让人自由、 1429179

下面编辑Items.py

![1.png][3]

这个文件就是用于设置爬虫项目中的Item对象结构的



下面先创建一个爬虫

![2.png][4]

```python
# -*- coding: utf-8 -*-
import scrapy


class MovieSpider(scrapy.Spider):    name = 'movie'
    allowed_domains = ['https://movie.douban.com/top250']
    start_urls = ['http://https://movie.douban.com/top250/']

    def parse(self, response):
        pass
```

这个movie.py中的parse方法是用于解析获取到的response的

编辑一下里面的内容

```python
# -*- coding: utf-8 -*-
import scrapy
from douban.items import DoubanItem


class MovieSpider(scrapy.Spider):
    name = 'movie'
    start_urls = ['https://movie.douban.com/top250/']

    def parse(self, response):
        item = DoubanItem()
        movies = response.xpath('//div[@class="item"]')

        next_url = response.xpath('//span[@class="next"]/a/@href').extract()
        if next_url:
            next_url = 'https://movie.douban.com/top250' + next_url[0]
            yield scrapy.Request(next_url)

        for movie in movies:
            item["ranking"] = movie.xpath('.//em/text()').extract()[0]
            item["name"] = movie.xpath('.//span[@class="title"]/text()').extract()[0]
            item["score"] = movie.xpath('.//div[@class="star"]/span[@class="rating_num"]/text()').extract()[0]
            item["description"] = movie.xpath('.//p[@class="quote"]/span[@class="inq"]/text()').extract()[0]
            item["score_num"] = movie.xpath('//div[@class="star"]//span/text()')[3].extract()[0]
            yield item
```

上面设置的next_url那一部分主要目的是实现自动翻页

之后还需要设置一下爬虫的user-agent

修改setting.py

`USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:66.0) Gecko/20100101 Firefox/66.0'`

运行爬虫

`scarpy crawl movie -o output.csv`

如果对获得的数据需要作进一步处理，则可以在pipelines.py中进一步修改process_item函数

这样会将结果保存在这个csv文件中

![3.png][5]

结果如图


[1]: http://42.193.111.59/usr/uploads/2021/01/1831815558.png#vwid=1271&vhei=199
[2]: http://42.193.111.59/usr/uploads/2021/01/1763255564.png#vwid=794&vhei=808
[3]: http://42.193.111.59/usr/uploads/2021/01/3954370186.png#vwid=502&vhei=419
[4]: http://42.193.111.59/usr/uploads/2021/01/1238348454.png#vwid=815&vhei=62
[5]: http://42.193.111.59/usr/uploads/2021/01/3964296014.png#vwid=1313&vhei=757