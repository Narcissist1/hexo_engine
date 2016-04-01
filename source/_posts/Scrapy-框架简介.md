---
title: Scrapy 框架简介
date: 2016-03-31 15:29:07
tags:
	- Scrapy
categories: 
	- 技术相关
---

# Scrapy 是？

Scrapy 是一个用于爬取网络内容的应用框架，可以提取结构化数据由于数据挖掘或者信息处理。

# Scrapy 安装

以下是 Scrapy 的一些依赖，安装 Scrapy 之前首先完成它们的安装配置：
	
	Python2.7
	pip 和 setuptools 包
	lxml. 
	OpenSSL
	
<!-- more -->
你可以通过 pip 来进行安装。

	pip install scrapy

PS: Mac OS X上需要安装 Xcode development tools.

	xcode-select --install


# Scrapy Tutorial

这个教程将会涵盖以下内容：

1. 创建一个 Scrapy 项目
2. 定义你需要抓去的 Item
3. 写一个爬虫去爬取一个网站并且提取出你的 Item
4. 写一个 Item Pipeline 去存取数据

我们爬取的目标是这个网站[Open directory project (dmoz)](http://www.dmoz.org/)

## 创建 Scrapy 项目

运行以下命令：

	scrapy startproject tutorial

这个命令用于创建 scrapy 项目，它会生成一个名为 tutorial 的目录包含以下内容：

	tutorial/
    scrapy.cfg            # deploy configuration file

    tutorial/             # project's Python module, you'll import your code from here
        __init__.py

        items.py          # project items file

        pipelines.py      # project pipelines file

        settings.py       # project settings file

        spiders/          # a directory where you'll later put your spiders
            __init__.py
            ...


## 定义 Item

Item 相当于一个包含抓取来的数据的一个容器，它们和 Python 中的字典很像，你可以像操作 Python 字典一样使用它。

通过创建 `scrapy.Item` 类和相应的 `srapy.Field` 属性来定义它， 这有点像一个 ORM，（django ORM 或者 SQLAlchemy）。

在本教程中我们希望抓取 title, 链接和描述，所以，修改 tutorial 目录下的  item.py 文件：
	
	import scrapy

	class DmozItem(scrapy.Item):
    	title = scrapy.Field()
    	link = scrapy.Field()
    	desc = scrapy.Field()
    	
## 第一个 Spider

Spider 是一些你定义的类。用于让 Scrapy 去从一个 domain 或者 URL 来爬取你需要的数据。

创建一个 Spider 需要继承 `scrapy.Spider` 类，以下是类中定义的属性：

* `name`: Spider 的名字，每个 Spider 的名字都是不同的，你不能定义两个拥有同样名字的 Spider
* `start_urls`: 一个包含开始爬取 URL 的列表
* `parse()`: spider 的一个方法，每一个 response 都会调用这个方法，它负责解析 response 中的数据，提取更多的 URL

本教程中，在`tutorial/spiders`目录下保存`dmoz_spider.py` 作为第一个 spider 内容如下：
	
	import scrapy

	class DmozSpider(scrapy.Spider):
    	name = "dmoz"
    	allowed_domains = ["dmoz.org"]
    	start_urls = [
        	"http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
       		"http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
    	]

    def parse(self, response):
        filename = response.url.split("/")[-2] + '.html'
        with open(filename, 'wb') as f:
            f.write(response.body)

### 爬取

返回项目跟目录，运行：

	scrapy crawl dmoz

这个命令会启动名字为`dmoz`的爬虫，就是我们刚刚写的。会有如下输出

	2014-01-23 18:13:07-0400 [scrapy] INFO: Scrapy started (bot: tutorial)
	2014-01-23 18:13:07-0400 [scrapy] INFO: Optional features available: ...
	2014-01-23 18:13:07-0400 [scrapy] INFO: Overridden settings: {}
	2014-01-23 18:13:07-0400 [scrapy] INFO: Enabled extensions: ...
	2014-01-23 18:13:07-0400 [scrapy] INFO: Enabled downloader middlewares: ...
	2014-01-23 18:13:07-0400 [scrapy] INFO: Enabled spider middlewares: ...
	2014-01-23 18:13:07-0400 [scrapy] INFO: Enabled item pipelines: ...
	2014-01-23 18:13:07-0400 [scrapy] INFO: Spider opened
	2014-01-23 18:13:08-0400 [scrapy] DEBUG: Crawled (200) <GET http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/> (referer: None)	
	2014-01-23 18:13:09-0400 [scrapy] DEBUG: Crawled (200) <GET http://www.dmoz.org/Computers/Programming/Languages/Python/Books/> (referer: None)
	2014-01-23 18:13:09-0400 [scrapy] INFO: Closing spider (finished)

### 提取Items

#### Selector

Scrapy Selector 使用两种机制 [CSS](https://www.w3.org/TR/selectors/) 和 [XPath](https://www.w3.org/TR/xpath/) 进行数据解析。
更多信息参考 []Selector documentation](http://doc.scrapy.org/en/1.0/topics/selectors.html#topics-selectors)

#### 在 Shell 里面尝试 Selector

运行以下命令：

	scrapy shell "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/"

然后就可以尝试查看 response 的各种属性了

	In [1]: response.xpath('//title')
	Out[1]: [<Selector xpath='//title' data=u'<title>Open Directory - Computers: Progr'>]

	In [2]: response.xpath('//title').extract()
	Out[2]: [u'<title>Open Directory - Computers: Programming: Languages: Python: Books</title>']

	In [3]: response.xpath('//title/text()')
	Out[3]: [<Selector xpath='//title/text()' data=u'Open Directory - Computers: Programming:'>]

	In [4]: response.xpath('//title/text()').extract()
	Out[4]: [u'Open Directory - Computers: Programming: Languages: Python: Books']

	In [5]: response.xpath('//title/text()').re('(\w+):')
	Out[5]: [u'Computers', u'Programming', u'Languages', u'Python']
	
#### 提取数据

现在我们来提取一些真正的信息吧

通过查看`response.body` 我们发现需要的信息在<ul>标签里面，

在我们的 spider 里面加入以下代码：
	
	import scrapy
	
	class DmozSpider(scrapy.Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
    ]

    def parse(self, response):
        for sel in response.xpath('//ul/li'):
            title = sel.xpath('a/text()').extract()
            link = sel.xpath('a/@href').extract()
            desc = sel.xpath('text()').extract()
            print title, link, desc


### 使用我们的Item

Item objects 是一个 Python 字典，你可以像使用字典那样使用它。

	>>> item = DmozItem()
	>>> item['title'] = 'Example title'
	>>> item['title']
	'Example title'

我们修改 spider 的代码：

	import scrapy
	
	from tutorial.items import DmozItem
	
	class DmozSpider(scrapy.Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
    ]

    def parse(self, response):
        for sel in response.xpath('//ul/li'):
            item = DmozItem()
            item['title'] = sel.xpath('a/text()').extract()
            item['link'] = sel.xpath('a/@href').extract()
            item['desc'] = sel.xpath('text()').extract()
            yield item

现在运行代码会得到`DmozItem` objects:

	[scrapy] DEBUG: Scraped from <200 http://www.dmoz.org/Computers/Programming/Languages/Python/Books/>
     {'desc': [u' - By David Mertz; Addison Wesley. Book in progress, full text, ASCII format. Asks for feedback. [author website, Gnosis Software, Inc.\n],
      'link': [u'http://gnosis.cx/TPiP/'],
      'title': [u'Text Processing in Python']}
	[scrapy] DEBUG: Scraped from <200 http://www.dmoz.org/Computers/Programming/Languages/Python/Books/>
     {'desc': [u' - By Sean McGrath; Prentice Hall PTR, 2000, ISBN 0130211192, has CD-ROM. Methods to build XML applications fast, Python tutorial, DOM and SAX, new Pyxie open source XML processing library. [Prentice Hall PTR]\n'],
      'link': [u'http://www.informit.com/store/product.aspx?isbn=0130211192'],
      'title': [u'XML Processing with Python']}

## Following links

仅仅提取两个页面的内容显然是不够的，如果我们需要提取链接在这两个页面上的链接以及它们的全部内容呢。
这里修改我们的代码：

	import scrapy
	
	from tutorial.items import DmozItem
	
	class DmozSpider(scrapy.Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/",
    ]

    def parse(self, response):
        for href in response.css("ul.directory.dir-col > li > a::attr('href')"):
            url = response.urljoin(href.extract())
            yield scrapy.Request(url, callback=self.parse_dir_contents)

    def parse_dir_contents(self, response):
        for sel in response.xpath('//ul/li'):
            item = DmozItem()
            item['title'] = sel.xpath('a/text()').extract()
            item['link'] = sel.xpath('a/@href').extract()
            item['desc'] = sel.xpath('text()').extract()
            yield item

现在 *parse()* 方法仅仅提取我们需要的URL，再用 *response.urljoin* 方法合成全局路径用以之后请求。在请求这些 URL 时，注册了一个 *parse_dir_contents()* callback 函数， 作为实际的内容提取。

Scrapy 的 following links 机制：当你在一个 callback 函数中 yield 一个 Request 请求时，Scrapy 会安排发送 request 并且注册另一个 callback 函数用于当前 request 结束时执行。

通过这个机制我们可以编写更为复杂的爬虫。

## 存储爬取的数据

最简单的方式是用 [Feed exports](http://doc.scrapy.org/en/1.0/topics/feed-exports.html#topics-feed-exports)，通过下面的命令：

	scrapy crawl dmoz -o items.json
	
这会生成一个名为`item.json`的文件，存储了我们抓取的数据，存储格式为JSON。
小的项目可以使用这种存储方式，如果需要处理更加复杂的情况，则可以编写一个 [Item Pipeline](http://doc.scrapy.org/en/1.0/topics/item-pipeline.html#topics-item-pipeline)


	
