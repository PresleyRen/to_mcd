---
layout:     post
title:      scrapy框架之递归解析和post请求
subtitle:   
date:       2019-03-22
author:     P
header-img: img/post-bg-digital-native.jpg
catalog: true
tags:
    - python
---
今日概要

- 递归爬取解析多页页面数据
- scrapy核心组件工作流程
- scrapy的post请求发送

今日详情

1.递归爬取解析多页页面数据

- 需求：将糗事百科所有页码的作者和段子内容数据进行爬取切持久化存储

- 需求分析：每一个页面对应一个url，则scrapy工程需要对每一个页码对应的url依次发起请求，然后通过对应的解析方法进行作者和段子内容的解析。

实现方案：

代码展示：

```
<code class="language-python hljs"># -*- coding: utf-8 -*-
import scrapy
from qiushibaike.items import QiushibaikeItem
# scrapy.http import Request
class QiushiSpider(scrapy.Spider):
    name = 'qiushi'
    allowed_domains = ['www.qiushibaike.com']
    start_urls = ['https://www.qiushibaike.com/text/']

    #爬取多页
    pageNum = 1 #起始页码
    url = 'https://www.qiushibaike.com/text/page/%s/' #每页的url

    def parse(self, response):
        div_list=response.xpath('//*[@id="content-left"]/div')
        for div in div_list:
            #//*[@id="qiushi_tag_120996995"]/div[1]/a[2]/h2
            author=div.xpath('.//div[@class="author clearfix"]//h2/text()').extract_first()
            author=author.strip('\n')
            content=div.xpath('.//div[@class="content"]/span/text()').extract_first()
            content=content.strip('\n')
            item=QiushibaikeItem()
            item['author']=author
            item['content']=content

            yield item #提交item到管道进行持久化

         #爬取所有页码数据
        if self.pageNum <= 13: #一共爬取13页（共13页）
            self.pageNum += 1
            url = format(self.url % self.pageNum)

            #递归爬取数据：callback参数的值为回调函数（将url请求后，得到的相应数据继续进行parse解析），递归调用parse函数
            yield scrapy.Request(url=url,callback=self.parse)</code>
```

2.五大核心组件工作流程：

[<img src="http://xiaobaibook.com/media/images/2018/10/22/scrapy.png" alt="" data-cke-saved-src="http://xiaobaibook.com/media/images/2018/10/22/scrapy.png" />](http://xiaobaibook.com/media/images/2018/10/22/scrapy.png)

- **引擎(Scrapy)**用来处理整个系统的数据流处理, 触发事务(框架核心)
- **调度器(Scheduler)**用来接受引擎发过来的请求, 压入队列中, 并在引擎再次请求的时候返回. 可以想像成一个URL（抓取网页的网址或者说是链接）的优先队列, 由它来决定下一个要抓取的网址是什么, 同时去除重复的网址
- **下载器(Downloader)**用于下载网页内容, 并将网页内容返回给蜘蛛(Scrapy下载器是建立在twisted这个高效的异步模型上的)
- **爬虫(Spiders)**爬虫是主要干活的, 用于从特定的网页中提取自己需要的信息, 即所谓的实体(Item)。用户也可以从中提取出链接,让Scrapy继续抓取下一个页面
- **项目管道(Pipeline)**负责处理爬虫从网页中抽取的实体，主要的功能是持久化实体、验证实体的有效性、清除不需要的信息。当页面被爬虫解析后，将被发送到项目管道，并经过几个特定的次序处理数据。




3.post请求发送

- 问题：在之前代码中，我们从来没有手动的对start_urls列表中存储的起始url进行过请求的发送，但是起始url的确是进行了请求的发送，那这是如何实现的呢？

- 解答：其实是因为爬虫文件中的爬虫类继承到了Spider父类中的start_requests（self）这个方法，该方法就可以对start_urls列表中的url发起请求：

```
<code class="language-python hljs">  def start_requests(self):
        for u in self.start_urls:
           yield scrapy.Request(url=u,callback=self.parse)</code>
```

【注意】该方法默认的实现，是对起始的url发起get请求，如果想发起post请求，则需要子类重写该方法。

　　-方法： 重写start_requests方法，让其发起post请求：

```
<code class="language-python hljs">def start_requests(self):
        #请求的url
        post_url = 'http://fanyi.baidu.com/sug'
        # post请求参数
        formdata = {
            'kw': 'wolf',
        }
        # 发送post请求
        yield scrapy.FormRequest(url=post_url, formdata=formdata, callback=self.parse)</code>
```
