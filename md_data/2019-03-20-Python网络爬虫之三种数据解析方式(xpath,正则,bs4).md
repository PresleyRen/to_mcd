---
layout:     post
title:      Python网络爬虫之三种数据解析方式(xpath,正则,bs4)
subtitle:   
date:       2019-03-20
author:     P
header-img: img/post-bg-ioses.jpg
catalog: true
tags:
    - python
---
### 引入

### 回顾requests实现数据爬取的流程

1. 指定url
1. 基于requests模块发起请求
1. 获取响应对象中的数据
1. 进行持久化存储

其实，在上述流程中还需要较为重要的一步，就是在持久化存储之前需要进行指定数据解析。因为大多数情况下的需求，我们都会指定去使用聚焦爬虫，也就是爬取页面中指定部分的数据值，而不是整个页面的数据。因此，本次课程中会给大家详细介绍讲解三种聚焦爬虫中的数据解析方式。至此，我们的数据爬取的流程可以修改为：

1. 指定url
1. 基于requests模块发起请求
1. 获取响应中的数据
1. **数据解析**
1. 进行持久化存储

今日概要

- 正则解析
- xpath解析
- bs4解析

知识点回顾

- requests模块的使用流程
- requests模块请求方法参数的作用
- 抓包工具抓取ajax的数据包

### 一.正解解析

- 常用正则表达式回顾：

{% raw %}
```
<code class="language-python hljs">   单字符：
        . : 除换行以外所有字符
        [] ：[aoe] [a-w] 匹配集合中任意一个字符
        \d ：数字  [0-9]
        \D : 非数字
        \w ：数字、字母、下划线、中文
        \W : 非\w
        \s ：所有的空白字符包,括空格、制表符、换页符等等。等价于 [ \f\n\r\t\v]。
        \S : 非空白
    数量修饰：
        * : 任意多次  >=0
        + : 至少1次   >=1
        ? : 可有可无  0次或者1次
        {m} ：固定m次 hello{3,}
        {m,} ：至少m次
        {m,n} ：m-n次
    边界：
        $ : 以某某结尾 
        ^ : 以某某开头
    分组：
        (ab)  
    贪婪模式： .*
    非贪婪（惰性）模式： .*?

    re.I : 忽略大小写
    re.M ：多行匹配
    re.S ：单行匹配

    re.sub(正则表达式, 替换内容, 字符串)</code>
```
{% endraw %}

- 回顾练习：

{% raw %}
```
<code class="language-python hljs">import re
#提取出python
key="javapythonc++php"
re.findall('python',key)[0]
#####################################################################
#提取出hello world
key="<html><h1>hello world<h1></html>"
re.findall('<h1>(.*)<h1>',key)[0]
#####################################################################
#提取170
string = '我喜欢身高为170的女孩'
re.findall('\d+',string)
#####################################################################
#提取出http://和https://
key='http://www.baidu.com and https://boob.com'
re.findall('https?://',key)
#####################################################################
#提取出hello
key='lalala<hTml>hello</HtMl>hahah' #输出<hTml>hello</HtMl>
re.findall('<[Hh][Tt][mM][lL]>(.*)</[Hh][Tt][mM][lL]>',key)
#####################################################################
#提取出hit. 
key='bobo@hit.edu.com'#想要匹配到hit.
re.findall('h.*?\.',key)
#####################################################################
#匹配sas和saas
key='saas and sas and saaas'
re.findall('sa{1,2}s',key)
#####################################################################
#匹配出i开头的行
string = '''fall in love with you
i love you very much
i love she
i love her'''

re.findall('^.*',string,re.M)
#####################################################################
#匹配全部行
string1 = """静夜思
窗前明月光
疑是地上霜
举头望明月
低头思故乡
"""

re.findall('.*',string1,re.S)</code>
```
{% endraw %}

<li>项目需求：爬取糗事百科指定页面的糗图，并将其保存到指定文件夹中
<pre><code class="language-python hljs">#!/usr/bin/env python
# -*- coding:utf-8 -*-
import requests
import re
import os
if __name__ == "__main__":
     url = 'https://www.qiushibaike.com/pic/%s/'
     headers={
         'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
     }
     #指定起始也结束页码
     page_start = int(input('enter start page:'))
     page_end = int(input('enter end page:'))

     #创建文件夹
     if not os.path.exists('images'):
         os.mkdir('images')
     #循环解析且下载指定页码中的图片数据
     for page in range(page_start,page_end+1):
         print('正在下载第%d页图片'%page)
         new_url = format(url % page)
         response = requests.get(url=new_url,headers=headers)

         #解析response中的图片链接
         e = '.*?<img src="(.*?)".*?>.*?'
         pa = re.compile(e,re.S)
         image_urls = pa.findall(response.text)
          #循环下载该页码下所有的图片数据
         for image_url in image_urls:
             image_url = 'https:' + image_url
             image_name = image_url.split('/')[-1]
             image_path = 'images/'+image_name

             image_data = requests.get(url=image_url,headers=headers).content
             with open(image_path,'wb') as fp:
                 fp.write(image_data)</code></pre>
 
</li>

### 二.Xpath解析

- 测试页面数据

{% raw %}
```
<code class="language-html hljs xml"><html lang="en">
<head>
	<meta charset="UTF-8" />
	<title>测试bs4</title>
</head>
<body>
	
		<p>百里守约</p>
	
	
		<p>李清照</p>
		<p>王安石</p>
		<p>苏轼</p>
		<p>柳宗元</p>
		<a href="http://www.song.com/" title="赵匡胤" target="_self">
			<span>this is span
		宋朝是最强大的王朝，不是军队的强大，而是经济很强大，国民都很有钱</a>
		<a href="" class="du">总为浮云能蔽日,长安不见使人愁</a>
		<img src="http://www.baidu.com/meinv.jpg" alt="" />
	
	
		<ul>
			<li><a href="http://www.baidu.com" title="qing">清明时节雨纷纷,路上行人欲断魂,借问酒家何处有,牧童遥指杏花村</a></li>
			<li><a href="http://www.163.com" title="qin">秦时明月汉时关,万里长征人未还,但使龙城飞将在,不教胡马度阴山</a></li>
			<li><a href="http://www.126.com" alt="qi">岐王宅里寻常见,崔九堂前几度闻,正是江南好风景,落花时节又逢君</a></li>
			<li><a href="http://www.sina.com" class="du">杜甫</a></li>
			<li><a href="http://www.dudu.com" class="du">杜牧</a></li>
			<li><b>杜小月</b></li>
			<li><i>度蜜月</i></li>
			<li><a href="http://www.haha.com" id="feng">凤凰台上凤凰游,凤去台空江自流,吴宫花草埋幽径,晋代衣冠成古丘</a></li>
		</ul>
	
</body>
</html></code>
```
{% endraw %}

- 常用xpath表达式回顾

{% raw %}
```
<code class="language-python hljs">属性定位：
    #找到class属性值为song的div标签
    //div[@class="song"] 
层级&索引定位：
    #找到class属性值为tang的div的直系子标签ul下的第二个子标签li下的直系子标签a
    //div[@class="tang"]/ul/li[2]/a
逻辑运算：
    #找到href属性值为空且class属性值为du的a标签
    //a[@href="" and @class="du"]
模糊匹配：
    //div[contains(@class, "ng")]
    //div[starts-with(@class, "ta")]
取文本：
    # /表示获取某个标签下的文本内容
    # //表示获取某个标签下的文本内容和所有子标签下的文本内容
    //div[@class="song"]/p[1]/text()
    //div[@class="tang"]//text()
取属性：
    //div[@class="tang"]//li[2]/a/@href</code>
```
{% endraw %}

- 代码中使用xpath表达式进行数据解析：

{% raw %}
```
<code class="language-python hljs">1.下载：pip install lxml
2.导包：from lxml import etree

3.将html文档或者xml文档转换成一个etree对象，然后调用对象中的方法查找指定的节点

　　2.1 本地文件：tree = etree.parse(文件名)
                tree.xpath("xpath表达式")

　　2.2 网络数据：tree = etree.HTML(网页内容字符串)
                tree.xpath("xpath表达式")</code>
```
{% endraw %}

<li>安装xpath插件在浏览器中对xpath表达式进行验证：可以在插件中直接执行xpath表达式
<ul>
<li>
将xpath插件拖动到谷歌浏览器拓展程序（更多工具）中，安装成功
</li>
<li>
 
启动和关闭插件 ctrl + shift + x
</li>

项目需求：获取好段子中段子的内容和作者   http://www.haoduanzi.com

{% raw %}
```
<code class="language-python hljs">from lxml import etree
import requests

url='http://www.haoduanzi.com/category-10_2.html'
headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36',
    }
url_content=requests.get(url,headers=headers).text
#使用xpath对url_conten进行解析
#使用xpath解析从网络上获取的数据
tree=etree.HTML(url_content)
#解析获取当页所有段子的标题
title_list=tree.xpath('//div[@class="log cate10 auth1"]/h3/a/text()')

ele_div_list=tree.xpath('//div[@class="log cate10 auth1"]')

text_list=[] #最终会存储12个段子的文本内容
for ele in ele_div_list:
    #段子的文本内容（是存放在list列表中）
    text_list=ele.xpath('./div[@class="cont"]//text()')
    #list列表中的文本内容全部提取到一个字符串中
    text_str=str(text_list)
    #字符串形式的文本内容防止到all_text列表中
    text_list.append(text_str)
print(title_list)
print(text_list)</code>
```
{% endraw %}

### 【重点】下载煎蛋网中的图片数据：[http://jandan.net/ooxx](http://jandan.net/ooxx)

{% raw %}
```
<code class="language-python hljs">import requests
from lxml import etree
from fake_useragent import UserAgent
import base64
import urllib.request
url = 'http://jandan.net/ooxx'
ua = UserAgent(verify_ssl=False,use_cache_server=False).random
headers = {
    'User-Agent':ua
}
page_text = requests.get(url=url,headers=headers).text

#查看页面源码：发现所有图片的src值都是一样的。
#简单观察会发现每张图片加载都是通过jandan_load_img(this)这个js函数实现的。
#在该函数后面还有一个class值为img-hash的标签，里面存储的是一组hash值，该值就是加密后的img地址
#加密就是通过js函数实现的，所以分析js函数，获知加密方式，然后进行解密。
#通过抓包工具抓取起始url的数据包，在数据包中全局搜索js函数名（jandan_load_img），然后分析该函数实现加密的方式。
#在该js函数中发现有一个方法调用，该方法就是加密方式，对该方法进行搜索
#搜索到的方法中会发现base64和md5等字样，md5是不可逆的所以优先考虑使用base64解密
#print(page_text)

tree = etree.HTML(page_text)
#在抓包工具的数据包响应对象对应的页面中进行xpath的编写，而不是在浏览器页面中。
#获取了加密的图片url数据
imgCode_list = tree.xpath('//span[@class="img-hash"]/text()')
imgUrl_list = []
for url in imgCode_list:
    #base64.b64decode(url)为byte类型，需要转成str
    img_url = 'http:'+base64.b64decode(url).decode()
    imgUrl_list.append(img_url)

for url in imgUrl_list:
    filePath = url.split('/')[-1]
    urllib.request.urlretrieve(url=url,filename=filePath)
    print(filePath+'下载成功')

</code>
```
{% endraw %}

### 三.BeautifulSoup解析

- 环境安装

{% raw %}
```
<code class="language-python hljs">- 需要将pip源设置为国内源，阿里源、豆瓣源、网易源等
   - windows
    （1）打开文件资源管理器(文件夹地址栏中)
    （2）地址栏上面输入 %appdata%
    （3）在这里面新建一个文件夹  pip
    （4）在pip文件夹里面新建一个文件叫做  pip.ini ,内容写如下即可
        [global]
        timeout = 6000
        index-url = https://mirrors.aliyun.com/pypi/simple/
        trusted-host = mirrors.aliyun.com
   - linux
    （1）cd ~
    （2）mkdir ~/.pip
    （3）vi ~/.pip/pip.conf
    （4）编辑内容，和windows一模一样
- 需要安装：pip install bs4
     bs4在使用时候需要一个第三方库，把这个库也安装一下
     pip install lxml</code>
```
{% endraw %}

- 基础使用

{% raw %}
```
<code class="language-python hljs">使用流程：       
    - 导包：from bs4 import BeautifulSoup
    - 使用方式：可以将一个html文档，转化为BeautifulSoup对象，然后通过对象的方法或者属性去查找指定的节点内容
        （1）转化本地文件：
             - soup = BeautifulSoup(open('本地文件'), 'lxml')
        （2）转化网络文件：
             - soup = BeautifulSoup('字符串类型或者字节类型', 'lxml')
        （3）打印soup对象显示内容为html文件中的内容

基础巩固：
    （1）根据标签名查找
        - soup.a   只能找到第一个符合要求的标签
    （2）获取属性
        - soup.a.attrs  获取a所有的属性和属性值，返回一个字典
        - soup.a.attrs['href']   获取href属性
        - soup.a['href']   也可简写为这种形式
    （3）获取内容
        - soup.a.string
        - soup.a.text
        - soup.a.get_text()
       【注意】如果标签还有标签，那么string获取到的结果为None，而其它两个，可以获取文本内容
    （4）find：找到第一个符合要求的标签
        - soup.find('a')  找到第一个符合要求的
        - soup.find('a', title="xxx")
        - soup.find('a', alt="xxx")
        - soup.find('a', class_="xxx")
        - soup.find('a', id="xxx")
    （5）find_all：找到所有符合要求的标签
        - soup.find_all('a')
        - soup.find_all(['a','b']) 找到所有的a和b标签
        - soup.find_all('a', limit=2)  限制前两个
    （6）根据选择器选择指定的内容
               select:soup.select('#feng')
        - 常见的选择器：标签选择器(a)、类选择器(.)、id选择器(#)、层级选择器
            - 层级选择器：
                div .dudu #lala .meme .xixi  下面好多级
                div > p > a > .lala          只能是下面一级
        【注意】select选择器返回永远是列表，需要通过下标提取指定的对象</code>
```
{% endraw %}

<li>需求：使用bs4实现将诗词名句网站中三国演义小说的每一章的内容爬去到本地磁盘进行存储   http://www.shicimingju.com/book/sanguoyanyi.html
<pre><code class="language-python hljs">#!/usr/bin/env python
# -*- coding:utf-8 -*-
import requests
from bs4 import BeautifulSoup

headers={
         'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
     }
def parse_content(url):
    #获取标题正文页数据
    page_text = requests.get(url,headers=headers).text
    soup = BeautifulSoup(page_text,'lxml')
    #解析获得标签
    ele = soup.find('div',class_='chapter_content')
    content = ele.text #获取标签中的数据值
    return content

if __name__ == "__main__":
     url = 'http://www.shicimingju.com/book/sanguoyanyi.html'
     reponse = requests.get(url=url,headers=headers)
     page_text = reponse.text

     #创建soup对象
     soup = BeautifulSoup(page_text,'lxml')
     #解析数据
     a_eles = soup.select('.book-mulu > ul > li > a')
     print(a_eles)
     cap = 1
     for ele in a_eles:
         print('开始下载第%d章节'%cap)
         cap+=1
         title = ele.string
         content_url = 'http://www.shicimingju.com'+ele['href']
         content = parse_content(content_url)

         with open('./sanguo.txt','w') as fp:
             fp.write(title+":"+content+'\n\n\n\n\n')
             print('结束下载第%d章节'%cap)</code></pre>
</li>
