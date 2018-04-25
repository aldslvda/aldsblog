title: 使用scrapy搭建一个简单的新闻爬虫
date: 2017-11-24 01:01:43
tags:
- Python
- scrapy
- crawler
categories:
- 技术分享
photos:	 
- "https://github.com/aldslvda/blog-images/blob/master/scrapylogo.png?raw=true"
toc: true
comment: true
---


# 使用scrapy搭建一个简单的新闻爬虫 #

前段时间面了几家公司的爬虫工程师岗位。所有的公司都会问的一个问题就是: 你使用过哪种爬虫框架？熟练度如何？是否能根据需求修改框架源码？有过分布式爬取的经验吗？    
每次我遇到这种问题都感到很尴尬 —— 现在主流的爬虫框架大概有Scrapy, PySpider(Python); Nutch, Heritrix(Java) —— 这些我一个都没用过, 目前在线上运行的爬虫全是 requests + json/html/xml 解析。![1](https://github.com/aldslvda/blog-images/blob/master/acfun_emoji/11.png?raw=true)    
至于分布式爬虫, 由于我目前接触的项目爬取量比较小(200+新闻app，300+新闻媒体微信公众号，300+微博的每日增量爬取), 每天的爬取量一台1核/1G/1M的阿里云足够胜任了, 暂时还用不到分布式爬取......![1](https://github.com/aldslvda/blog-images/blob/master/acfun_emoji/01.png?raw=true)

另外分布式爬取大多依赖上面提到的框架。

所以我做了一个很艰难的决定！把之前写过的爬虫！用scrapy重新实现一遍！![1](https://github.com/aldslvda/blog-images/blob/master/acfun_emoji/1010.png?raw=true)

......   
......   
......   

好吧，的确有点艰难......

所以先从其中一个开始吧![1](https://github.com/aldslvda/blog-images/blob/master/acfun_emoji/1015.png?raw=true)。

下面会讲到如何使用scrapy 编写一个网易新闻app的爬虫。

**\*** 我选择了一本参考书籍 《learning scrapy》, 但是这本书爬取的示例网站需要翻墙，所以我只看了框架相关的部分，实际网页解析之类的事情，在修改之前的爬虫的过程中解决。

## 1. 环境配置 ##
我开发使用电脑的是Mac OSX, 但其实在实际的Python开发中, OSX使用的命令和ubuntu/debian大同小异。    

- 安装scrapy

```bash
sudo pip install scrapy
```

- 创建工程文件夹

```bash
scrapy startproject crawler
tree crawler
.
├── crawler
│   ├── __init__.py
│   ├── items.py
│   ├── middlewares.py
│   ├── pipelines.py
│   ├── settings.py
│   └── spiders
│       └── __init__.py
└── scrapy.cfg
```
scrapy startproject crawler 这行命令创建了一个新的工程文件夹，文件夹的结构使用tree命令展示出来了。     
这些文件的主要作用:   

(1) scrapy.cfg 项目的配置文件   
(2) crawler/ :Python代码存放的位置   
(3) crawler/items.py: items文件，定义一个(或多个)item的属性   
(4) crawler/pipelines: 项目的管道文件。
> 在scrapy的官方文档中，pipeline的作用是：   
   1. 清洗html数据；   
	2. 验证已经爬取的数据(检查item是否有特定属性)；   
	3. 去重；   
	4. 将爬取的item存进数据库。   

(5) crawler/settings.py 配置文件   
(6) crawler/spiders/ :爬虫文件的目录   

## 2. 爬虫的编写 ##

### 2.1 定义Item ###
简单来说，item的作用是装载抓取到的数据，是一种类似字典的容器。它的属性都会定义为scrapy.item.Field对象。

由于我们要写的是一个爬取新闻的app, 首先要明确的是我们需要的数据是什么，对于一个爬虫来说, 重要的是能否**取我所需**, 而不是尽我所能爬取对应网站的所有数据。
那么对于一个新闻爬虫，我们爬到的每一条新闻都需要一些什么样的数据呢？   

-    category 新闻类型(按照包含的媒体 分为图片/视频)    
-    source\_type  新闻来源的类型(app, 微博, 微信, 网站, 电子报 etc.)   
-    source\_name  新闻来源的名称
-    news_sign     一条新闻的唯一标识(可以用于去重)
-    views         浏览量
-    title         标题
-    url           新闻链接
-    publish_time  发布时间
-    text          新闻文本
-    images        新闻图片的列表

以上就是我们需要的一些数据，我们按照上面的列表定义一个item

```python
from scrapy.item import Item, Field

class NewsCrawlerItem(Item):
    category = Field()
    source_type = Field()
    source_name = Field()

    news_sign = Field()
    views = Field()
    title = Field()
    url = Field()
    publish_time = Field()
    text = Field()
    images = Field()
```    

### 2.2 编写爬虫 ###
Spider是爬虫的核心部分，用于从一个(或一系列)网站爬取数据。
要建立一个Spider，你必须为scrapy.spider.BaseSpider创建一个子类，并确定三个主要的、强制的属性：
- name 爬虫名称，必须是唯一的
- start_urls 起始页面
- parse() 解析爬取到的数据所用的方法, 接受的唯一参数是scrapy.Request方法请求得到的Response对象

#### 2.2.1 关于二维爬取 ####
一个典型的爬虫在两个方向移动：
- 水平方向：从一个索引页到另一个索引页
- 垂直方向：从索引页面到下一级页面(可能是下一级的索引，或者内容页面)

这个例子中水平方向是网易新闻的一页新闻到另一页，它返回的是一个个的新闻列表；   
垂直方向是从新闻列表逐个进入新闻的内容界面

#### 2.2.2 关于从网页(或者HTTP请求的Response Body)中提取数据 ####
- html 网页的数据可以用 beautifusoup/xpath/正则 提取
- json 直接解析成Python的字典即可
- xml  python也有对应的库用于解析   

本文的爬虫会用到json和html解析，这里html使用xpath解析。

#### 2.2.3 一个网易新闻app的Spider ####

```python
#coding:utf-8
import datetime
import socket
import json
import scrapy
import urlparse

from scrapy.loader.processors import MapCompose, Join
from scrapy.loader import ItemLoader
from scrapy.http import Request

from news_crawler.items import NewsCrawlerItem

class BasicSpider(scrapy.Spider):
    name = "netease"
    allowed_domains = ["163.com"]
    # Start on the first index page
    chls = [
        "T1370583240249",  "T1348649145984",  "T1348647909107",  "T1348648037603",  
        "T1368497029546",  "T1348648141035",  "T1474271789612",  "T1467284926140",  
        "T1492136373327",  "T1348648517839",  "T1348648650048",  "T1498701411149",  
        "T1348648756099",  "T1473054348939",  "T1356600029035",  "T1348649079062",  
        "T1348649503389",  "T1348649176279",  "T1348649475931",  "T1411113472760",  
        "T1486979691117",  "T1348649580692",  "T1348649654285",  "T1348649776727",  
        "T1350383429665",  "T1421997195219",  "T1456394562871",  "T1348654060988",  
        "T1348654085632",  "T1491816738487",  "T1348654105308",  "T1348654151579",  
        "T1348654204705",  "T1414389941036",  "T1401272877187",  "T1385429690972",  
        "T1348654225495",  "T1397116135282",  "T1444270454635",  "T1481105123675",  
        "T1503456682313",  "T1464592736048",  "T1504171773862",  "T1348650593803",  
        "T1348650839000",  "T1414142214384",  "T1441074311424",  "T1482470888760",  
        "T1499853820829",  "T1509504918215",  "T1502955728035",  "T1509448512433",  
        "T1419315959525",  "T1419316284722",  "T1419316384474",  "T1419316531256",  
        "T1427878984398",  "T1433137697241",  "T1449126525962",  "T1456112189138",  
        "T1493374039495",  "T1456112438822",  "T1468031118349",  "T1488432440430",  
        "T1488432474929",  "T1504689350701"
    ]
    start_urls = (
        'http://c.m.163.com/nc/article/list/%s/0-20.html'%chlid for chlid in chls
    )
    def parse(self, response):
        # Get the next index URLs and yield Requests
        url_spl = response.url.split('/')
        chl_url = '/'.join(url_spl[:-1])
        chl_id = url_spl[-2]
        items = json.loads(response.body)[chl_id]
        # Get item URLs and yield Requests
        for item in items:
            if 'url_3w' in item and item['url_3w']:
                news_url = item['url_3w']
                if '_mobile' not in news_url and '3g.163.com' not in news_url:
                    news_url = news_url.replace('.html', '_mobile.html')
                yield Request(news_url, callback=self.parse_item, dont_filter=True)
        if len(items) == 20:
            page_num = int(url_spl[-1].split('-')[0]) /20 + 1
            yield Request(chl_url+'/%d-%d.html'%(page_num*20, page_num*20+20), callback=self.parse, dont_filter=True)



    def parse_item(self, response):
        """ This function parses a netease page.
        @returns item
        @scrapes category source_type source_name news_sign views title url publish_time text images
        """

        # Create the loader using the response
        
        item = NewsCrawlerItem()
        item['category'] = 'image'
        item['source_type'] = 'app'
        item['source_name'] = self.name
        item['news_sign'] = ''
        item['views'] = 0
        item['url'] = response.url
        item['title'] = response.xpath('//h1/text()').extract()[0]
        # Load fields using XPath expressions
        if '3g.163.com' in response.url:
            item['publish_time'] = response.xpath('//*[@property="article:published_time"]/@content').extract()[0][:19].replace('T','')
            item['text'] = ''.join(response.xpath('//*[@class="content"]//p/text()').extract())
            item['images'] = response.xpath('//*[@class="content"]//img/@src').extract()
            item['news_sign'] = 'netease'+response.xpath('//article/@id').extract()[0]
        else:
            item['publish_time'] = response.xpath('//*[@id="articleBody"]/div[1]/span[1]/text()').extract()[0]
            item['text'] = ''.join(response.xpath('//*[@class="article-body"]//p/text()').extract())
            item['images'] = response.xpath('//*[@class="article-body"]//img/@src').extract()
            item['news_sign'] = 'netease'+response.xpath('//*[@id="docId"]/@value').extract()[0]
        return item
```

#### 2.2.4 爬虫代码代码的解析 ####
上面提到过parse接受response作为参数。
这里重点说一下parse()方法中的两个生成器yeild,分别代表了垂直爬取和水平爬取。
首先说一下Request对象:

```python
class Request(object_ref):

    def __init__(self, url, callback=None, method='GET', headers=None, body=None,
                 cookies=None, meta=None, encoding='utf-8', priority=0,
                 dont_filter=False, errback=None, flags=None):
```

这里我们使用到了3个参数,url,callback,dont\_filter。
我们第一次使用yeild是请求新闻的内容页面，这里的回调函数是parse\_item()，用来提取单个新闻的信息;    
第二次使用yeild是请求“下一页”的数据，这里的回调函数是parse(),当然从scrapy的官方文档来看，这里不传callback的话，默认的回调函数也是parse(),这样下一页的数据也会再一次通过parse()解析，实现水平方向上的爬取。
dont\_filter是告诉Request不要对url进行过滤(去重)

下面的parse\_item()就是html的解析了，这里不赘述。



## 3 爬取到的数据存储 ##
这里我们用mongoDb对爬取到的数据进行存储。
### 3.1 配置MongoDB信息 ###
在settings.py中设置MongoDB的信息

```python
# MongoDB settings
MONGODB_SERVER = "$mongo_server"
MONGODB_PORT = 27017
MONGODB_DB = "news"
MONGODB_COLLECTION = "news_item"

```
这里server的ip隐去了
### 3.2 编写MongoDBPipeline ###

```python
from pymongo import MongoClient

from scrapy.conf import settings
from scrapy.exceptions import DropItem
from scrapy import log


class MongoDBPipeline(object):

    def __init__(self):
        client = MongoClient(
            settings['MONGODB_SERVER'],
            settings['MONGODB_PORT']
        )
        news_db = client[settings['MONGODB_DB']]
        self.collection = news_db[settings['MONGODB_COLLECTION']]

    def process_item(self, item, spider):
        valid = True
        if not item['news_sign']:
            valid = False
            raise DropItem("Missing item!")
        if valid:
            self.collection.insert(dict(item))
            log.msg("news added to MongoDB database!",
                    level=log.INFO, spider=spider)
        return item

```

这个pipeline的作用包括上面提到的验证已经爬取的数据和将爬取的item存进数据库。去重暂时还没有做处理。

### 3.3 配置pipeline ###
 在settings.py中添加下面的配置
 
 ```python
 ITEM_PIPELINES = {
    'news_crawler.pipelines.mongodb_ppl.MongoDBPipeline': 300,
}
 
 ```
 
 可以看到这是一个字典，键为pipeline的路径，值为优先级(值越小越优先)。
 
 
## 4.总结 ##

按照上面的步骤走下来，大概就能编写出一个网易新闻的爬虫。这个爬虫写出来，大概就迈出了重构以前爬虫代码的第一步了![1](https://github.com/aldslvda/blog-images/blob/master/acfun_emoji/1010.png?raw=true)

后面碰到什么问题也会更新上来的，敬请期待![1](https://github.com/aldslvda/blog-images/blob/master/acfun_emoji/25.png?raw=true)
