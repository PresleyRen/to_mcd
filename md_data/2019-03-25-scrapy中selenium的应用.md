---
layout:     post
title:      scrapy中selenium的应用
subtitle:   
date:       2019-03-25
author:     P
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - python
---
引入

- 在通过scrapy框架进行某些网站数据爬取的时候，往往会碰到页面动态数据加载的情况发生，如果直接使用scrapy对其url发请求，是绝对获取不到那部分动态加载出来的数据值。但是通过观察我们会发现，通过浏览器进行url请求发送则会加载出对应的动态加载出的数据。那么如果我们想要在scrapy也获取动态加载出的数据，则必须使用selenium创建浏览器对象，然后通过该浏览器对象进行请求发送，获取动态加载的数据值。

今日详情

1.案例分析：

2.selenium在scrapy中使用的原理分析：

<img src="https://img-blog.csdn.net/20180416224202657?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4ODE3NzM5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="" data-cke-saved-src="https://img-blog.csdn.net/20180416224202657?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4ODE3NzM5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" />

当引擎将国内板块url对应的请求提交给下载器后，下载器进行网页数据的下载，然后将下载到的页面数据，封装到response中，提交给引擎，引擎将response在转交给Spiders。Spiders接受到的response对象中存储的页面数据里是没有动态加载的新闻数据的。要想获取动态加载的新闻数据，则需要在下载中间件中对下载器提交给引擎的response响应对象进行拦截，切对其内部存储的页面数据进行篡改，修改成携带了动态加载出的新闻数据，然后将被篡改的response对象最终交给Spiders进行解析操作。

3.selenium在scrapy中的使用流程：

- 重写爬虫文件的构造方法，在该方法中使用selenium实例化一个浏览器对象（因为浏览器对象只需要被实例化一次）
- 重写爬虫文件的closed(self,spider)方法，在其内部关闭浏览器对象。该方法是在爬虫结束时被调用
- 重写下载中间件的process_response方法，让该方法对响应对象进行拦截，并篡改response中存储的页面数据
- 在配置文件中开启下载中间件

4.代码展示：

- 爬虫文件：

{% raw %}
```
<code class="language-python hljs">class WangyiSpider(RedisSpider):
    name = 'wangyi'
    #allowed_domains = ['www.xxxx.com']
    start_urls = ['https://news.163.com']
    def __init__(self):
        #实例化一个浏览器对象(实例化一次)
        self.bro = webdriver.Chrome(executable_path='/Users/bobo/Desktop/chromedriver')

    #必须在整个爬虫结束后，关闭浏览器
    def closed(self,spider):
        print('爬虫结束')
        self.bro.quit()</code>
```
{% endraw %}

- 中间件文件：

{% raw %}
```
<code class="language-python hljs">from scrapy.http import HtmlResponse    
    #参数介绍：
    #拦截到响应对象（下载器传递给Spider的响应对象）
    #request：响应对象对应的请求对象
    #response：拦截到的响应对象
    #spider：爬虫文件中对应的爬虫类的实例
    def process_response(self, request, response, spider):
        #响应对象中存储页面数据的篡改
        if request.url in['http://news.163.com/domestic/','http://news.163.com/world/','http://news.163.com/air/','http://war.163.com/']:
            spider.bro.get(url=request.url)
            js = 'window.scrollTo(0,document.body.scrollHeight)'
            spider.bro.execute_script(js)
            time.sleep(2)  #一定要给与浏览器一定的缓冲加载数据的时间
            #页面数据就是包含了动态加载出来的新闻数据对应的页面数据
            page_text = spider.bro.page_source
            #篡改响应对象
            return HtmlResponse(url=spider.bro.current_url,body=page_text,encoding='utf-8',request=request)
        else:
            return response</code>
```
{% endraw %}

- 配置文件：

{% raw %}
```
<code class="language-python hljs">DOWNLOADER_MIDDLEWARES = {
    'wangyiPro.middlewares.WangyiproDownloaderMiddleware': 543,

}</code>
```
{% endraw %}
