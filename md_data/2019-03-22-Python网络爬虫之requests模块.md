---
layout:     post
title:      Python网络爬虫之requests模块
subtitle:   
date:       2019-03-22
author:     P
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - python
---
今日内容

- session处理cookie
- proxies参数设置请求代理ip
- 基于线程池的数据爬取

知识点回顾

- xpath的解析流程
- bs4的解析流程
- 常用xpath表达式
- 常用bs4解析方法

引入

有些时候，我们在使用爬虫程序去爬取一些用户相关信息的数据（爬取张三人人网个人主页数据）时，如果使用之前requests模块常规操作时，往往达不到我们想要的目的，例如：

{% raw %}
```
<code class="language-python hljs">#!/usr/bin/env python
# -*- coding:utf-8 -*-
import requests
if __name__ == "__main__":

    #张三人人网个人信息页面的url
    url = 'http://www.renren.com/289676607/profile'

   #伪装UA
    headers={
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
    }
    #发送请求，获取响应对象
    response = requests.get(url=url,headers=headers)
    #将响应内容写入文件
    with open('./renren.html','w',encoding='utf-8') as fp:
        fp.write(response.text)</code>
```
{% endraw %}

一.基于requests模块的cookie操作

　　　　- cookie概念：当用户通过浏览器首次访问一个域名时，访问的web服务器会给客户端发送数据，以保持web服务器与客户端之间的状态保持，这些数据就是cookie。

　　　　- cookie作用：我们在浏览器中，经常涉及到数据的交换，比如你登录邮箱，登录一个页面。我们经常会在此时设置30天内记住我，或者自动登录选项。那么它们是怎么记录信息的呢，答案就是今天的主角cookie了，Cookie是由HTTP服务器设置的，保存在浏览器中，但HTTP协议是一种无状态协议，在数据交换完毕后，服务器端和客户端的链接就会关闭，每次交换数据都需要建立新的链接。就像我们去超市买东西，没有积分卡的情况下，我们买完东西之后，超市没有我们的任何消费信息，但我们办了积分卡之后，超市就有了我们的消费信息。cookie就像是积分卡，可以保存积分，商品就是我们的信息，超市的系统就像服务器后台，http协议就是交易的过程。

- 经过cookie的相关介绍，其实你已经知道了为什么上述案例中爬取到的不是张三个人信息页，而是登录页面。那应该如何抓取到张三的个人信息页呢？

　　思路：

　　　　1.我们需要使用爬虫程序对人人网的登录时的请求进行一次抓取，获取请求中的cookie数据

　　　　2.在使用个人信息页的url进行请求时，该请求需要携带 1 中的cookie，只有携带了cookie后，服务器才可识别这次请求的用户信息，方可响应回指定的用户信息页数据

{% raw %}
```
<code class="language-python hljs">#!/usr/bin/env python
# -*- coding:utf-8 -*-
import requests
if __name__ == "__main__":

    #登录请求的url（通过抓包工具获取）
    post_url = 'http://www.renren.com/ajaxLogin/login?1=1&uniqueTimestamp=201873958471'
    #创建一个session对象，该对象会自动将请求中的cookie进行存储和携带
    session = requests.session()
   #伪装UA
    headers={
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
    }
    formdata = {
        'email': '17701256561',
        'icode': '',
        'origURL': 'http://www.renren.com/home',
        'domain': 'renren.com',
        'key_id': '1',
        'captcha_type': 'web_login',
        'password': '7b456e6c3eb6615b2e122a2942ef3845da1f91e3de075179079a3b84952508e4',
        'rkey': '44fd96c219c593f3c9612360c80310a3',
        'f': 'https%3A%2F%2Fwww.baidu.com%2Flink%3Furl%3Dm7m_NSUp5Ri_ZrK5eNIpn_dMs48UAcvT-N_kmysWgYW%26wd%3D%26eqid%3Dba95daf5000065ce000000035b120219',
    }
    #使用session发送请求，目的是为了将session保存该次请求中的cookie
    session.post(url=post_url,data=formdata,headers=headers)

    get_url = 'http://www.renren.com/960481378/profile'
    #再次使用session进行请求的发送，该次请求中已经携带了cookie
    response = session.get(url=get_url,headers=headers)
    #设置响应内容的编码格式
    response.encoding = 'utf-8'
    #将响应内容写入文件
    with open('./renren.html','w') as fp:
        fp.write(response.text)</code>
```
{% endraw %}

二.基于requests模块的代理操作

<li>什么是代理
<ul>
<li>
代理就是第三方代替本体处理相关事务。例如：生活中的代理：代购，中介，微商......
</li>

爬虫中为什么需要使用代理

<li>
一些网站会有相应的反爬虫措施，例如很多网站会检测某一段时间某个IP的访问次数，如果访问频率太快以至于看起来不像正常访客，它可能就会会禁止这个IP的访问。所以我们需要设置一些代理IP，每隔一段时间换一个代理IP，就算IP被禁止，依然可以换个IP继续爬取。
</li>

代理的分类：

<li>
正向代理：代理客户端获取数据。正向代理是为了保护客户端防止被追究责任。
</li>
<li>
反向代理：代理服务器提供数据。反向代理是为了保护服务器或负责负载均衡。
</li>

免费代理ip提供网站

<li>
http://www.goubanjia.com/
</li>
<li>
西祠代理
</li>
<li>
快代理
</li>

代码

{% raw %}
```
<code class="language-python hljs">#!/usr/bin/env python
# -*- coding:utf-8 -*-
import requests
import random
if __name__ == "__main__":
    #不同浏览器的UA
    header_list = [
        # 遨游
        {"user-agent": "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Maxthon 2.0)"},
        # 火狐
        {"user-agent": "Mozilla/5.0 (Windows NT 6.1; rv:2.0.1) Gecko/20100101 Firefox/4.0.1"},
        # 谷歌
        {
            "user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_0) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.56 Safari/535.11"}
    ]
    #不同的代理IP
    proxy_list = [
        {"http": "112.115.57.20:3128"},
        {'http': '121.41.171.223:3128'}
    ]
    #随机获取UA和代理IP
    header = random.choice(header_list)
    proxy = random.choice(proxy_list)

    url = 'http://www.baidu.com/s?ie=UTF-8&wd=ip'
    #参数3：设置代理
    response = requests.get(url=url,headers=header,proxies=proxy)
    response.encoding = 'utf-8'
    
    with open('daili.html', 'wb') as fp:
        fp.write(response.content)
    #切换成原来的IP
    requests.get(url, proxies={"http": ""})</code>
```
{% endraw %}

三.基于multiprocessing.dummy线程池的数据爬取

<li>需求：爬取梨视频的视频信息，并计算其爬取数据的耗时
<ul>
<li>普通爬取

<pre class="cke_widget_element" data-cke-widget-data="%7B%22lang%22%3A%22python%22%2C%22code%22%3A%22%25%25time%5Cnimport%20requests%5Cnimport%20random%5Cnfrom%20lxml%20import%20etree%5Cnimport%20re%5Cnfrom%20fake_useragent%20import%20UserAgent%5Cn%23%E5%AE%89%E8%A3%85fake-useragent%E5%BA%93%3Apip%20install%20fake-useragent%5Cnurl%20%3D%20'http%3A%2F%2Fwww.pearvideo.com%2Fcategory_1'%5Cn%23%E9%9A%8F%E6%9C%BA%E4%BA%A7%E7%94%9FUA%2C%E5%A6%82%E6%9E%9C%E6%8A%A5%E9%94%99%E5%88%99%E5%8F%AF%E4%BB%A5%E6%B7%BB%E5%8A%A0%E5%A6%82%E4%B8%8B%E5%8F%82%E6%95%B0%EF%BC%9A%5Cn%23ua%20%3D%20UserAgent(verify_ssl%3DFalse%2Cuse_cache_server%3DFalse).random%5Cn%23%E7%A6%81%E7%94%A8%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%93%E5%AD%98%EF%BC%9A%5Cn%23ua%20%3D%20UserAgent(use_cache_server%3DFalse)%5Cn%23%E4%B8%8D%E7%BC%93%E5%AD%98%E6%95%B0%E6%8D%AE%EF%BC%9A%5Cn%23ua%20%3D%20UserAgent(cache%3DFalse)%5Cn%23%E5%BF%BD%E7%95%A5ssl%E9%AA%8C%E8%AF%81%EF%BC%9A%5Cn%23ua%20%3D%20UserAgent(verify_ssl%3DFalse)%5Cn%5Cnua%20%3D%20UserAgent().random%5Cnheaders%20%3D%20%7B%5Cn%20%20%20%20'User-Agent'%3Aua%5Cn%7D%5Cn%23%E8%8E%B7%E5%8F%96%E9%A6%96%E9%A1%B5%E9%A1%B5%E9%9D%A2%E6%95%B0%E6%8D%AE%5Cnpage_text%20%3D%20requests.get(url%3Durl%2Cheaders%3Dheaders).text%5Cn%23%E5%AF%B9%E8%8E%B7%E5%8F%96%E7%9A%84%E9%A6%96%E9%A1%B5%E9%A1%B5%E9%9D%A2%E6%95%B0%E6%8D%AE%E4%B8%AD%E7%9A%84%E7%9B%B8%E5%85%B3%E8%A7%86%E9%A2%91%E8%AF%A6%E6%83%85%E9%93%BE%E6%8E%A5%E8%BF%9B%E8%A1%8C%E8%A7%A3%E6%9E%90%5Cntree%20%3D%20etree.HTML(page_text)%5Cnli_list%20%3D%20tree.xpath('%2F%2Fdiv%5B%40id%3D%5C%22listvideoList%5C%22%5D%2Ful%2Fli')%5Cndetail_urls%20%3D%20%5B%5D%5Cnfor%20li%20in%20li_list%3A%5Cn%20%20%20%20detail_url%20%3D%20'http%3A%2F%2Fwww.pearvideo.com%2F'%2Bli.xpath('.%2Fdiv%2Fa%2F%40href')%5B0%5D%5Cn%20%20%20%20title%20%3D%20li.xpath('.%2F%2Fdiv%5B%40class%3D%5C%22vervideo-title%5C%22%5D%2Ftext()')%5B0%5D%5Cn%20%20%20%20detail_urls.append(detail_url)%5Cnfor%20url%20in%20detail_urls%3A%5Cn%20%20%20%20page_text%20%3D%20requests.get(url%3Durl%2Cheaders%3Dheaders).text%5Cn%20%20%20%20vedio_url%20%3D%20re.findall('srcUrl%3D%5C%22(.*%3F)%5C%22'%2Cpage_text%2Cre.S)%5B0%5D%5Cn%20%20%20%20%5Cn%20%20%20%20data%20%3D%20requests.get(url%3Dvedio_url%2Cheaders%3Dheaders).content%5Cn%20%20%20%20fileName%20%3D%20str(random.randint(1%2C10000))%2B'.mp4'%20%23%E9%9A%8F%E6%9C%BA%E7%94%9F%E6%88%90%E8%A7%86%E9%A2%91%E6%96%87%E4%BB%B6%E5%90%8D%E7%A7%B0%5Cn%20%20%20%20with%20open(fileName%2C'wb')%20as%20fp%3A%5Cn%20%20%20%20%20%20%20%20fp.write(data)%5Cn%20%20%20%20%20%20%20%20print(fileName%2B'%20is%20over')%22%2C%22classes%22%3Anull%7D" data-cke-widget-upcasted="1" data-cke-widget-keep-attr="0" data-widget="codeSnippet"><code class="language-python hljs">%%time
import requests
import random
from lxml import etree
import re
from fake_useragent import UserAgent
#安装fake-useragent库:pip install fake-useragent
url = 'http://www.pearvideo.com/category_1'
#随机产生UA,如果报错则可以添加如下参数：
#ua = UserAgent(verify_ssl=False,use_cache_server=False).random
#禁用服务器缓存：
#ua = UserAgent(use_cache_server=False)
#不缓存数据：
#ua = UserAgent(cache=False)
#忽略ssl验证：
#ua = UserAgent(verify_ssl=False)

ua = UserAgent().random
headers = {
    'User-Agent':ua
}
#获取首页页面数据
page_text = requests.get(url=url,headers=headers).text
#对获取的首页页面数据中的相关视频详情链接进行解析
tree = etree.HTML(page_text)
li_list = tree.xpath('//div[@id="listvideoList"]/ul/li')
detail_urls = []
for li in li_list:
    detail_url = 'http://www.pearvideo.com/'+li.xpath('./div/a/@href')[0]
    title = li.xpath('.//div[@class="vervideo-title"]/text()')[0]
    detail_urls.append(detail_url)
for url in detail_urls:
    page_text = requests.get(url=url,headers=headers).text
    vedio_url = re.findall('srcUrl="(.*?)"',page_text,re.S)[0]
    
    data = requests.get(url=vedio_url,headers=headers).content
    fileName = str(random.randint(1,10000))+'.mp4' #随机生成视频文件名称
    with open(fileName,'wb') as fp:
        fp.write(data)
        print(fileName+' is over')</code></pre>
<img class="cke_reset cke_widget_mask" src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" alt="" /><img class="cke_reset cke_widget_drag_handler" title="点击并拖拽以移动" src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" alt="" width="15" height="15" data-cke-widget-drag-handler="1" />
 
</li>
<li>基于线程池的爬取

<pre class="cke_widget_element" data-cke-widget-data="%7B%22lang%22%3A%22python%22%2C%22code%22%3A%22%25%25time%5Cnimport%20requests%5Cnimport%20random%5Cnfrom%20lxml%20import%20etree%5Cnimport%20re%5Cnfrom%20fake_useragent%20import%20UserAgent%5Cn%23%E5%AE%89%E8%A3%85fake-useragent%E5%BA%93%3Apip%20install%20fake-useragent%5Cn%23%E5%AF%BC%E5%85%A5%E7%BA%BF%E7%A8%8B%E6%B1%A0%E6%A8%A1%E5%9D%97%5Cnfrom%20multiprocessing.dummy%20import%20Pool%5Cn%23%E5%AE%9E%E4%BE%8B%E5%8C%96%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AF%B9%E8%B1%A1%5Cnpool%20%3D%20Pool()%5Cnurl%20%3D%20'http%3A%2F%2Fwww.pearvideo.com%2Fcategory_1'%5Cn%23%E9%9A%8F%E6%9C%BA%E4%BA%A7%E7%94%9FUA%5Cnua%20%3D%20UserAgent().random%5Cnheaders%20%3D%20%7B%5Cn%20%20%20%20'User-Agent'%3Aua%5Cn%7D%5Cn%23%E8%8E%B7%E5%8F%96%E9%A6%96%E9%A1%B5%E9%A1%B5%E9%9D%A2%E6%95%B0%E6%8D%AE%5Cnpage_text%20%3D%20requests.get(url%3Durl%2Cheaders%3Dheaders).text%5Cn%23%E5%AF%B9%E8%8E%B7%E5%8F%96%E7%9A%84%E9%A6%96%E9%A1%B5%E9%A1%B5%E9%9D%A2%E6%95%B0%E6%8D%AE%E4%B8%AD%E7%9A%84%E7%9B%B8%E5%85%B3%E8%A7%86%E9%A2%91%E8%AF%A6%E6%83%85%E9%93%BE%E6%8E%A5%E8%BF%9B%E8%A1%8C%E8%A7%A3%E6%9E%90%5Cntree%20%3D%20etree.HTML(page_text)%5Cnli_list%20%3D%20tree.xpath('%2F%2Fdiv%5B%40id%3D%5C%22listvideoList%5C%22%5D%2Ful%2Fli')%5Cn%5Cndetail_urls%20%3D%20%5B%5D%23%E5%AD%98%E5%82%A8%E4%BA%8C%E7%BA%A7%E9%A1%B5%E9%9D%A2%E7%9A%84url%5Cnfor%20li%20in%20li_list%3A%5Cn%20%20%20%20detail_url%20%3D%20'http%3A%2F%2Fwww.pearvideo.com%2F'%2Bli.xpath('.%2Fdiv%2Fa%2F%40href')%5B0%5D%5Cn%20%20%20%20title%20%3D%20li.xpath('.%2F%2Fdiv%5B%40class%3D%5C%22vervideo-title%5C%22%5D%2Ftext()')%5B0%5D%5Cn%20%20%20%20detail_urls.append(detail_url)%5Cn%20%20%20%20%5Cnvedio_urls%20%3D%20%5B%5D%23%E5%AD%98%E5%82%A8%E8%A7%86%E9%A2%91%E7%9A%84url%5Cnfor%20url%20in%20detail_urls%3A%5Cn%20%20%20%20page_text%20%3D%20requests.get(url%3Durl%2Cheaders%3Dheaders).text%5Cn%20%20%20%20vedio_url%20%3D%20re.findall('srcUrl%3D%5C%22(.*%3F)%5C%22'%2Cpage_text%2Cre.S)%5B0%5D%5Cn%20%20%20%20vedio_urls.append(vedio_url)%20%5Cn%23%E4%BD%BF%E7%94%A8%E7%BA%BF%E7%A8%8B%E6%B1%A0%E8%BF%9B%E8%A1%8C%E8%A7%86%E9%A2%91%E6%95%B0%E6%8D%AE%E4%B8%8B%E8%BD%BD%20%20%20%20%5Cnfunc_request%20%3D%20lambda%20link%3Arequests.get(url%3Dlink%2Cheaders%3Dheaders).content%5Cnvideo_data_list%20%3D%20pool.map(func_request%2Cvedio_urls)%5Cn%23%E4%BD%BF%E7%94%A8%E7%BA%BF%E7%A8%8B%E6%B1%A0%E8%BF%9B%E8%A1%8C%E8%A7%86%E9%A2%91%E6%95%B0%E6%8D%AE%E4%BF%9D%E5%AD%98%5Cnfunc_saveData%20%3D%20lambda%20data%3Asave(data)%5Cnpool.map(func_saveData%2Cvideo_data_list)%5Cndef%20save(data)%3A%5Cn%20%20%20%20fileName%20%3D%20str(random.randint(1%2C10000))%2B'.mp4'%5Cn%20%20%20%20with%20open(fileName%2C'wb')%20as%20fp%3A%5Cn%20%20%20%20%20%20%20%20fp.write(data)%5Cn%20%20%20%20%20%20%20%20print(fileName%2B'%E5%B7%B2%E5%AD%98%E5%82%A8')%5Cn%20%20%20%20%20%20%20%20%5Cnpool.close()%5Cnpool.join()%22%2C%22classes%22%3Anull%7D" data-cke-widget-upcasted="1" data-cke-widget-keep-attr="0" data-widget="codeSnippet"><code class="language-python hljs">%%time
import requests
import random
from lxml import etree
import re
from fake_useragent import UserAgent
#安装fake-useragent库:pip install fake-useragent
#导入线程池模块
from multiprocessing.dummy import Pool
#实例化线程池对象
pool = Pool()
url = 'http://www.pearvideo.com/category_1'
#随机产生UA
ua = UserAgent().random
headers = {
    'User-Agent':ua
}
#获取首页页面数据
page_text = requests.get(url=url,headers=headers).text
#对获取的首页页面数据中的相关视频详情链接进行解析
tree = etree.HTML(page_text)
li_list = tree.xpath('//div[@id="listvideoList"]/ul/li')

detail_urls = []#存储二级页面的url
for li in li_list:
    detail_url = 'http://www.pearvideo.com/'+li.xpath('./div/a/@href')[0]
    title = li.xpath('.//div[@class="vervideo-title"]/text()')[0]
    detail_urls.append(detail_url)
    
vedio_urls = []#存储视频的url
for url in detail_urls:
    page_text = requests.get(url=url,headers=headers).text
    vedio_url = re.findall('srcUrl="(.*?)"',page_text,re.S)[0]
    vedio_urls.append(vedio_url) 
#使用线程池进行视频数据下载    
func_request = lambda link:requests.get(url=link,headers=headers).content
video_data_list = pool.map(func_request,vedio_urls)
#使用线程池进行视频数据保存
func_saveData = lambda data:save(data)
pool.map(func_saveData,video_data_list)
def save(data):
    fileName = str(random.randint(1,10000))+'.mp4'
    with open(fileName,'wb') as fp:
        fp.write(data)
        print(fileName+'已存储')
        
pool.close()
pool.join()</code></pre>
<img class="cke_reset cke_widget_mask" src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" alt="" /><img class="cke_reset cke_widget_drag_handler" title="点击并拖拽以移动" src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" alt="" width="15" height="15" data-cke-widget-drag-handler="1" />
 
</li>
