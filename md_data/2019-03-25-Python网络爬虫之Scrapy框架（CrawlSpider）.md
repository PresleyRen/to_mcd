---
layout:     post
title:      Python网络爬虫之Scrapy框架（CrawlSpider）
subtitle:   
date:       2019-03-25
author:     P
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - python
---
引入

提问：如果想要通过爬虫程序去爬取糗百全站数据新闻数据的话，有几种实现方法？

方法一：基于Scrapy框架中的Spider的递归爬取进行实现（Request模块递归回调parse方法）。

方法二：基于CrawlSpider的自动爬取进行实现（更加简洁和高效）。

今日概要

- CrawlSpider简介
<li>CrawlSpider使用
<ul>
- 基于CrawlSpider爬虫文件的创建
- 链接提取器
- 规则解析器

今日详情

一.简介

　　CrawlSpider其实是Spider的一个子类，除了继承到Spider的特性和功能外，还派生除了其自己独有的更加强大的特性和功能。其中最显著的功能就是LinkExtractors链接提取器。Spider是所有爬虫的基类，其设计原则只是为了爬取start_url列表中网页，而从爬取到的网页中提取出的url进行继续的爬取工作使用CrawlSpider更合适。

二.使用

　　1.创建scrapy工程：scrapy startproject projectName

　　2.创建爬虫文件：scrapy genspider -t crawl spiderName www.xxx.com

　　　　--此指令对比以前的指令多了 "-t crawl"，表示创建的爬虫文件是基于CrawlSpider这个类的，而不再是Spider这个基类。

　　3.观察生成的爬虫文件

{% raw %}
```
<code class="language-python hljs"># -*- coding: utf-8 -*-
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule


class ChoutidemoSpider(CrawlSpider):
    name = 'choutiDemo'
    #allowed_domains = ['www.chouti.com']
    start_urls = ['http://www.chouti.com/']

    rules = (
        Rule(LinkExtractor(allow=r'Items/'), callback='parse_item', follow=True),
    )

    def parse_item(self, response):
        i = {}
        #i['domain_id'] = response.xpath('//input[@id="sid"]/@value').extract()
        #i['name'] = response.xpath('//div[@id="name"]').extract()
        #i['description'] = response.xpath('//div[@id="description"]').extract()
        return i</code>
```
{% endraw %}

　　- 7行：表示该爬虫程序是基于CrawlSpider类的

　　- 12，13，14行：表示为提取Link规则

　　- 16行：解析方法

　　CrawlSpider类和Spider类的最大不同是CrawlSpider多了一个rules属性，其作用是定义提取动作。在rules中可以包含一个或多个Rule对象，在Rule对象中包含了LinkExtractor对象。

3.1 LinkExtractor：顾名思义，链接提取器。

　　　　**LinkExtractor(**

**　　　　　　　  **allow=r'Items/'，# 满足括号中正则表达式的值会被提取，如果为空，则全部匹配。

　　　　　　　　 deny=xxx,  # 满足正则表达式的则不会被提取。

　　　　　　　　 restrict_xpaths=xxx, # 满足xpath表达式的值会被提取

　　　　　　　　 restrict_css=xxx, # 满足css表达式的值会被提取

　　　　　　　　 deny_domains=xxx, # 不会被提取的链接的domains。　

{% raw %}
```
**　　  )**
```
{% endraw %}

　　　　- 作用：提取response中符合规则的链接。

　　　　

　　3.2 Rule : 规则解析器。根据链接提取器中提取到的链接，根据指定规则提取解析器链接网页中的内容。

　　　　 Rule(LinkExtractor(allow=r'Items/'), callback='parse_item', follow=True)

　　　　- 参数介绍：

　　　　　　参数1：指定链接提取器

　　　　　　参数2：指定规则解析器解析数据的规则（回调函数）

　　　　　　参数3：是否将链接提取器继续作用到链接提取器提取出的链接网页中。当callback为None,参数3的默认值为true。

　　3.3 rules=( ):指定不同规则解析器。一个Rule对象表示一种提取规则。

　　3.4 CrawlSpider整体爬取流程：

　　　　a)爬虫文件首先根据起始url，获取该url的网页内容

　　　　b)链接提取器会根据指定提取规则将步骤a中网页内容中的链接进行提取

　　　　c)规则解析器会根据指定解析规则将链接提取器中提取到的链接中的网页内容根据指定的规则进行解析

　　　　d)将解析数据封装到item中，然后提交给管道进行持久化存储

　　4.简单代码实战应用

{% raw %}
```
<code class="language-python hljs"># -*- coding: utf-8 -*-
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule


class CrawldemoSpider(CrawlSpider):
    name = 'qiubai'
    #allowed_domains = ['www.qiushibaike.com']
    start_urls = ['https://www.qiushibaike.com/pic/']

    #连接提取器：会去起始url响应回来的页面中提取指定的url
    link = LinkExtractor(allow=r'/pic/page/\d+\?') #s=为随机数
    link1 = LinkExtractor(allow=r'/pic/$')#爬取第一页
    #rules元组中存放的是不同的规则解析器（封装好了某种解析规则)
    rules = (
        #规则解析器：可以将连接提取器提取到的所有连接表示的页面进行指定规则（回调函数）的解析
        Rule(link, callback='parse_item', follow=True),
        Rule(link1, callback='parse_item', follow=True),
    )

    def parse_item(self, response):
        print(response)
        


</code>
```
{% endraw %}

　　4.2 爬虫文件：

{% raw %}
```
<code class="language-python hljs"># -*- coding: utf-8 -*-
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from qiubaiBycrawl.items import QiubaibycrawlItem
import re
class QiubaitestSpider(CrawlSpider):
    name = 'qiubaiTest'
    #起始url
    start_urls = ['http://www.qiushibaike.com/']

    #定义链接提取器，且指定其提取规则
    page_link = LinkExtractor(allow=r'/8hr/page/\d+/')
    
    rules = (
        #定义规则解析器，且指定解析规则通过callback回调函数
        Rule(page_link, callback='parse_item', follow=True),
    )

    #自定义规则解析器的解析规则函数
    def parse_item(self, response):
        div_list = response.xpath('//div[@id="content-left"]/div')
        
        for div in div_list:
            #定义item
            item = QiubaibycrawlItem()
            #根据xpath表达式提取糗百中段子的作者
            item['author'] = div.xpath('./div/a[2]/h2/text()').extract_first().strip('\n')
            #根据xpath表达式提取糗百中段子的内容
            item['content'] = div.xpath('.//div[@class="content"]/span/text()').extract_first().strip('\n')

            yield item #将item提交至管道</code>
```
{% endraw %}

　　4.2 item文件：

{% raw %}
```
<code class="language-python hljs"># -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# https://doc.scrapy.org/en/latest/topics/items.html

import scrapy


class QiubaibycrawlItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    author = scrapy.Field() #作者
    content = scrapy.Field() #内容</code>
```
{% endraw %}

　　4.3 管道文件：

{% raw %}
```
<code class="language-python hljs"># -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://doc.scrapy.org/en/latest/topics/item-pipeline.html

class QiubaibycrawlPipeline(object):
    
    def __init__(self):
        self.fp = None
        
    def open_spider(self,spider):
        print('开始爬虫')
        self.fp = open('./data.txt','w')
        
    def process_item(self, item, spider):
        #将爬虫文件提交的item写入文件进行持久化存储
        self.fp.write(item['author']+':'+item['content']+'\n')
        return item
    
    def close_spider(self,spider):
        print('结束爬虫')
        self.fp.close()</code>
```
{% endraw %}
