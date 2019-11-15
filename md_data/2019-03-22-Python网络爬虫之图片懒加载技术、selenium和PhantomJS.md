---
layout:     post
title:      Python网络爬虫之图片懒加载技术、selenium和PhantomJS
subtitle:   
date:       2019-03-22
author:     P
header-img: img/post-bg-iWatch.jpg
catalog: true
tags:
    - python
---
引入

今日概要

- 图片懒加载
- selenium
- phantomJs
- 谷歌无头浏览器

知识点回顾

- 验证码处理流程

今日详情

动态数据加载处理

一.图片懒加载

<li>什么是图片懒加载？
<ul>
<li>案例分析：抓取站长素材http://sc.chinaz.com/中的图片数据

<pre class="cke_widget_element" data-cke-widget-data="%7B%22lang%22%3A%22python%22%2C%22code%22%3A%22%23!%2Fusr%2Fbin%2Fenv%20python%5Cn%23%20-*-%20coding%3Autf-8%20-*-%5Cnimport%20requests%5Cnfrom%20lxml%20import%20etree%5Cn%5Cnif%20__name__%20%3D%3D%20%5C%22__main__%5C%22%3A%5Cn%20%20%20%20%20url%20%3D%20'http%3A%2F%2Fsc.chinaz.com%2Ftupian%2Fgudianmeinvtupian.html'%5Cn%20%20%20%20%20headers%20%3D%20%7B%5Cn%20%20%20%20%20%20%20%20%20'User-Agent'%3A%20'Mozilla%2F5.0%20(Macintosh%3B%20Intel%20Mac%20OS%20X%2010_12_0)%20AppleWebKit%2F537.36%20(KHTML%2C%20like%20Gecko)%20Chrome%2F69.0.3497.100%20Safari%2F537.36'%2C%5Cn%20%20%20%20%20%7D%5Cn%20%20%20%20%20%23%E8%8E%B7%E5%8F%96%E9%A1%B5%E9%9D%A2%E6%96%87%E6%9C%AC%E6%95%B0%E6%8D%AE%5Cn%20%20%20%20%20response%20%3D%20requests.get(url%3Durl%2Cheaders%3Dheaders)%5Cn%20%20%20%20%20response.encoding%20%3D%20'utf-8'%5Cn%20%20%20%20%20page_text%20%3D%20response.text%5Cn%20%20%20%20%20%23%E8%A7%A3%E6%9E%90%E9%A1%B5%E9%9D%A2%E6%95%B0%E6%8D%AE%EF%BC%88%E8%8E%B7%E5%8F%96%E9%A1%B5%E9%9D%A2%E4%B8%AD%E7%9A%84%E5%9B%BE%E7%89%87%E9%93%BE%E6%8E%A5%EF%BC%89%5Cn%20%20%20%20%20%23%E5%88%9B%E5%BB%BAetree%E5%AF%B9%E8%B1%A1%5Cn%20%20%20%20%20tree%20%3D%20etree.HTML(page_text)%5Cn%20%20%20%20%20div_list%20%3D%20tree.xpath('%2F%2Fdiv%5B%40id%3D%5C%22container%5C%22%5D%2Fdiv')%5Cn%20%20%20%20%20%23%E8%A7%A3%E6%9E%90%E8%8E%B7%E5%8F%96%E5%9B%BE%E7%89%87%E5%9C%B0%E5%9D%80%E5%92%8C%E5%9B%BE%E7%89%87%E7%9A%84%E5%90%8D%E7%A7%B0%5Cn%20%20%20%20%20for%20div%20in%20div_list%3A%5Cn%20%20%20%20%20%20%20%20%20image_url%20%3D%20div.xpath('.%2F%2Fimg%2F%40src')%5Cn%20%20%20%20%20%20%20%20%20image_name%20%3D%20div.xpath('.%2F%2Fimg%2F%40alt')%5Cn%20%20%20%20%20%20%20%20%20print(image_url)%20%23%E6%89%93%E5%8D%B0%E5%9B%BE%E7%89%87%E9%93%BE%E6%8E%A5%5Cn%20%20%20%20%20%20%20%20%20print(image_name)%23%E6%89%93%E5%8D%B0%E5%9B%BE%E7%89%87%E5%90%8D%E7%A7%B0%22%2C%22classes%22%3Anull%7D" data-cke-widget-upcasted="1" data-cke-widget-keep-attr="0" data-widget="codeSnippet"><code class="language-python hljs">#!/usr/bin/env python
# -*- coding:utf-8 -*-
import requests
from lxml import etree

if __name__ == "__main__":
     url = 'http://sc.chinaz.com/tupian/gudianmeinvtupian.html'
     headers = {
         'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
     }
     #获取页面文本数据
     response = requests.get(url=url,headers=headers)
     response.encoding = 'utf-8'
     page_text = response.text
     #解析页面数据（获取页面中的图片链接）
     #创建etree对象
     tree = etree.HTML(page_text)
     div_list = tree.xpath('//div[@id="container"]/div')
     #解析获取图片地址和图片的名称
     for div in div_list:
         image_url = div.xpath('.//img/@src')
         image_name = div.xpath('.//img/@alt')
         print(image_url) #打印图片链接
         print(image_name)#打印图片名称</code></pre>
<img class="cke_reset cke_widget_mask" src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" alt="" /><img class="cke_reset cke_widget_drag_handler" title="点击并拖拽以移动" src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" alt="" width="15" height="15" data-cke-widget-drag-handler="1" />
 
</li>
<li>
- 运行结果观察发现，我们可以获取图片的名称，但是链接获取的为空，检查后发现xpath表达式也没有问题，究其原因出在了哪里呢？
</li>
<li>
图片懒加载概念：
<ul>
<li>
图片懒加载是一种网页优化技术。图片作为一种网络资源，在被请求时也与普通静态资源一样，将占用网络资源，而一次性将整个页面的所有图片加载完，将大大增加页面的首屏加载时间。为了解决这种问题，通过前后端配合，使图片仅在浏览器当前视窗内出现时才加载该图片，达到减少首屏图片请求数的技术就被称为图片懒加载。
</li>

网站一般如何实现图片懒加载技术呢？

<li>
在网页源码中，在img标签中首先会使用一个伪属性（通常使用src2，original......）去存放真正的图片链接而并非是直接存放在src属性中。当图片出现到页面的可视化区域中，会动态将伪属性替换成src属性，完成图片的加载。
</li>

站长素材案例后续分析：通过细致观察页面的结构后发现，网页中图片的链接是存储在了src2这个伪属性中

```
<code class="language-python hljs">#!/usr/bin/env python
# -*- coding:utf-8 -*-
import requests
from lxml import etree

if __name__ == "__main__":
     url = 'http://sc.chinaz.com/tupian/gudianmeinvtupian.html'
     headers = {
         'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36',
     }
     #获取页面文本数据
     response = requests.get(url=url,headers=headers)
     response.encoding = 'utf-8'
     page_text = response.text
     #解析页面数据（获取页面中的图片链接）
     #创建etree对象
     tree = etree.HTML(page_text)
     div_list = tree.xpath('//div[@id="container"]/div')
     #解析获取图片地址和图片的名称
     for div in div_list:
         image_url = div.xpath('.//img/@src'2) #src2伪属性
         image_name = div.xpath('.//img/@alt')
         print(image_url) #打印图片链接
         print(image_name)#打印图片名称</code>
```

二.selenium

<li>什么是selenium？
<ul>
<li>
是Python的一个第三方库，对外提供的接口可以操作浏览器，然后让浏览器完成自动化的操作。　　
</li>

环境搭建

<li>
安装selenum：pip install selenium
</li>
<li>
获取某一款浏览器的驱动程序（以谷歌浏览器为例）
<ul>
<li>
谷歌浏览器驱动下载地址：`http://chromedriver.storage.googleapis.com/index.html`
</li>
<li>
下载的驱动程序必须和浏览器的版本统一，大家可以根据`http://blog.csdn.net/huilan_same/article/details/51896672中提供的版本映射表进行对应`
</li>

```
<code class="language-python hljs">from selenium import webdriver
from time import sleep

# 后面是你的浏览器驱动位置，记得前面加r'','r'是防止字符转义的
driver = webdriver.Chrome(r'驱动程序路径')
# 用get打开百度页面
driver.get("http://www.baidu.com")
# 查找页面的设置选项，并进行点击
driver.find_elements_by_link_text('设置')[0].click()
sleep(2)
# # 打开设置后找到搜索设置选项，设置为每页显示50条
driver.find_elements_by_link_text('搜索设置')[0].click()
sleep(2)

# 选中每页显示50条
m = driver.find_element_by_id('nr')
sleep(2)
m.find_element_by_xpath('//*[@id="nr"]/option[3]').click()
m.find_element_by_xpath('.//option[3]').click()
sleep(2)

# 点击保存设置
driver.find_elements_by_class_name("prefpanelgo")[0].click()
sleep(2)

# 处理弹出的警告页面   确定accept() 和 取消dismiss()
driver.switch_to_alert().accept()
sleep(2)
# 找到百度的输入框，并输入 美女
driver.find_element_by_id('kw').send_keys('美女')
sleep(2)
# 点击搜索按钮
driver.find_element_by_id('su').click()
sleep(2)
# 在打开的页面中找到Selenium - 开源中国社区，并打开这个页面
driver.find_elements_by_link_text('美女_百度图片')[0].click()
sleep(3)

# 关闭浏览器
driver.quit()</code>
```

代码介绍：

```
<code class="hljs coffeescript">#导包
from selenium import webdriver  
#创建浏览器对象，通过该对象可以操作浏览器
browser = webdriver.Chrome('驱动路径')
#使用浏览器发起指定请求
browser.get(url)

#使用下面的方法，查找指定的元素进行操作即可
    find_element_by_id            根据id找节点
    find_elements_by_name         根据name找
    find_elements_by_xpath        根据xpath查找
    find_elements_by_tag_name     根据标签名找
    find_elements_by_class_name   根据class名字查找</code>
```

三.phantomJs

- PhantomJS是一款无界面的浏览器，其自动化操作流程和上述操作谷歌浏览器是一致的。由于是无界面的，为了能够展示自动化操作流程，PhantomJS为用户提供了一个截屏的功能，使用save_screenshot函数实现。
<li>代码演示：

<pre class="cke_widget_element" data-cke-widget-data="%7B%22code%22%3A%22from%20selenium%20import%20webdriver%5Cnimport%20time%5Cn%5Cn%23%20phantomjs%E8%B7%AF%E5%BE%84%5Cnpath%20%3D%20r'PhantomJS%E9%A9%B1%E5%8A%A8%E8%B7%AF%E5%BE%84'%5Cnbrowser%20%3D%20webdriver.PhantomJS(path)%5Cn%5Cn%23%20%E6%89%93%E5%BC%80%E7%99%BE%E5%BA%A6%5Cnurl%20%3D%20'http%3A%2F%2Fwww.baidu.com%2F'%5Cnbrowser.get(url)%5Cn%5Cntime.sleep(3)%5Cn%5Cnbrowser.save_screenshot(r'phantomjs%5C%5Cbaidu.png')%5Cn%5Cn%23%20%E6%9F%A5%E6%89%BEinput%E8%BE%93%E5%85%A5%E6%A1%86%5Cnmy_input%20%3D%20browser.find_element_by_id('kw')%5Cn%23%20%E5%BE%80%E6%A1%86%E9%87%8C%E9%9D%A2%E5%86%99%E6%96%87%E5%AD%97%5Cnmy_input.send_keys('%E7%BE%8E%E5%A5%B3')%5Cntime.sleep(3)%5Cn%23%E6%88%AA%E5%B1%8F%5Cnbrowser.save_screenshot(r'phantomjs%5C%5Cmeinv.png')%5Cn%5Cn%23%20%E6%9F%A5%E6%89%BE%E6%90%9C%E7%B4%A2%E6%8C%89%E9%92%AE%5Cnbutton%20%3D%20browser.find_elements_by_class_name('s_btn')%5B0%5D%5Cnbutton.click()%5Cn%5Cntime.sleep(3)%5Cn%5Cnbrowser.save_screenshot(r'phantomjs%5C%5Cshow.png')%5Cn%5Cntime.sleep(3)%5Cn%5Cnbrowser.quit()%22%2C%22classes%22%3Anull%7D" data-cke-widget-upcasted="1" data-cke-widget-keep-attr="0" data-widget="codeSnippet"><code class="hljs python">from selenium import webdriver
import time

# phantomjs路径
path = r'PhantomJS驱动路径'
browser = webdriver.PhantomJS(path)

# 打开百度
url = 'http://www.baidu.com/'
browser.get(url)

time.sleep(3)

browser.save_screenshot(r'phantomjs\baidu.png')

# 查找input输入框
my_input = browser.find_element_by_id('kw')
# 往框里面写文字
my_input.send_keys('美女')
time.sleep(3)
#截屏
browser.save_screenshot(r'phantomjs\meinv.png')

# 查找搜索按钮
button = browser.find_elements_by_class_name('s_btn')[0]
button.click()

time.sleep(3)

browser.save_screenshot(r'phantomjs\show.png')

time.sleep(3)

browser.quit()</code></pre>
<img class="cke_reset cke_widget_mask" src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" alt="" /><img class="cke_reset cke_widget_drag_handler" title="点击并拖拽以移动" src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" alt="" width="15" height="15" data-cke-widget-drag-handler="1" />
 
</li>
<li>
重点：selenium+phantomjs 就是爬虫终极解决方案:有些网站上的内容信息是通过动态加载js形成的，所以使用普通爬虫程序无法回去动态加载的js内容。例如豆瓣电影中的电影信息是通过下拉操作动态加载更多的电影信息。
<ul>
<li>
综合操作：需求是尽可能多的爬取豆瓣网中的电影信息

<pre class="cke_widget_element" data-cke-widget-data="%7B%22code%22%3A%22from%20selenium%20import%20webdriver%5Cnfrom%20time%20import%20sleep%5Cnimport%20time%5Cn%5Cnif%20__name__%20%3D%3D%20'__main__'%3A%5Cn%20%20%20%20url%20%3D%20'https%3A%2F%2Fmovie.douban.com%2Ftyperank%3Ftype_name%3D%25E6%2581%2590%25E6%2580%2596%26type%3D20%26interval_id%3D100%3A90%26action%3D'%5Cn%20%20%20%20%23%20%E5%8F%91%E8%B5%B7%E8%AF%B7%E6%B1%82%E5%89%8D%EF%BC%8C%E5%8F%AF%E4%BB%A5%E8%AE%A9url%E8%A1%A8%E7%A4%BA%E7%9A%84%E9%A1%B5%E9%9D%A2%E5%8A%A8%E6%80%81%E5%8A%A0%E8%BD%BD%E5%87%BA%E6%9B%B4%E5%A4%9A%E7%9A%84%E6%95%B0%E6%8D%AE%5Cn%20%20%20%20path%20%3D%20r'C%3A%5C%5CUsers%5C%5CAdministrator%5C%5CDesktop%5C%5C%E7%88%AC%E8%99%AB%E6%8E%88%E8%AF%BE%5C%5Cday05%5C%5Cziliao%5C%5Cphantomjs-2.1.1-windows%5C%5Cbin%5C%5Cphantomjs.exe'%5Cn%20%20%20%20%23%20%E5%88%9B%E5%BB%BA%E6%97%A0%E7%95%8C%E9%9D%A2%E7%9A%84%E6%B5%8F%E8%A7%88%E5%99%A8%E5%AF%B9%E8%B1%A1%5Cn%20%20%20%20bro%20%3D%20webdriver.PhantomJS(path)%5Cn%20%20%20%20%23%20%E5%8F%91%E8%B5%B7url%E8%AF%B7%E6%B1%82%5Cn%20%20%20%20bro.get(url)%5Cn%20%20%20%20time.sleep(3)%5Cn%20%20%20%20%23%20%E6%88%AA%E5%9B%BE%5Cn%20%20%20%20bro.save_screenshot('1.png')%5Cn%5Cn%20%20%20%20%23%20%E6%89%A7%E8%A1%8Cjs%E4%BB%A3%E7%A0%81%EF%BC%88%E8%AE%A9%E6%BB%9A%E5%8A%A8%E6%9D%A1%E5%90%91%E4%B8%8B%E5%81%8F%E7%A7%BBn%E4%B8%AA%E5%83%8F%E7%B4%A0%EF%BC%88%E4%BD%9C%E7%94%A8%EF%BC%9A%E5%8A%A8%E6%80%81%E5%8A%A0%E8%BD%BD%E4%BA%86%E6%9B%B4%E5%A4%9A%E7%9A%84%E7%94%B5%E5%BD%B1%E4%BF%A1%E6%81%AF%EF%BC%89%EF%BC%89%5Cn%20%20%20%20js%20%3D%20'window.scrollTo(0%2Cdocument.body.scrollHeight)'%5Cn%20%20%20%20bro.execute_script(js)%20%20%23%20%E8%AF%A5%E5%87%BD%E6%95%B0%E5%8F%AF%E4%BB%A5%E6%89%A7%E8%A1%8C%E4%B8%80%E7%BB%84%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%BD%A2%E5%BC%8F%E7%9A%84js%E4%BB%A3%E7%A0%81%5Cn%20%20%20%20time.sleep(2)%5Cn%5Cn%20%20%20%20bro.execute_script(js)%20%20%23%20%E8%AF%A5%E5%87%BD%E6%95%B0%E5%8F%AF%E4%BB%A5%E6%89%A7%E8%A1%8C%E4%B8%80%E7%BB%84%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%BD%A2%E5%BC%8F%E7%9A%84js%E4%BB%A3%E7%A0%81%5Cn%20%20%20%20time.sleep(2)%5Cn%20%20%20%20bro.save_screenshot('2.png')%20%5Cn%20%20%20%20time.sleep(2)%20%5Cn%20%20%20%20%23%20%E4%BD%BF%E7%94%A8%E7%88%AC%E8%99%AB%E7%A8%8B%E5%BA%8F%E7%88%AC%E5%8E%BB%E5%BD%93%E5%89%8Durl%E4%B8%AD%E7%9A%84%E5%86%85%E5%AE%B9%20%5Cn%20%20%20%20html_source%20%3D%20bro.page_source%20%23%20%E8%AF%A5%E5%B1%9E%E6%80%A7%E5%8F%AF%E4%BB%A5%E8%8E%B7%E5%8F%96%E5%BD%93%E5%89%8D%E6%B5%8F%E8%A7%88%E5%99%A8%E7%9A%84%E5%BD%93%E5%89%8D%E9%A1%B5%E7%9A%84%E6%BA%90%E7%A0%81%EF%BC%88html%EF%BC%89%20%5Cn%20%20%20%20with%20open('.%2Fsource.html'%2C%20'w'%2C%20encoding%3D'utf-8')%20as%20fp%3A%20%5Cn%20%20%20%20%20%20%20%20fp.write(html_source)%20%5Cn%20%20%20%20bro.quit()%22%2C%22classes%22%3Anull%7D" data-cke-widget-upcasted="1" data-cke-widget-keep-attr="0" data-widget="codeSnippet"><code class="hljs python">from selenium import webdriver
from time import sleep
import time

if __name__ == '__main__':
    url = 'https://movie.douban.com/typerank?type_name=%E6%81%90%E6%80%96&type=20&interval_id=100:90&action='
    # 发起请求前，可以让url表示的页面动态加载出更多的数据
    path = r'C:\Users\Administrator\Desktop\爬虫授课\day05\ziliao\phantomjs-2.1.1-windows\bin\phantomjs.exe'
    # 创建无界面的浏览器对象
    bro = webdriver.PhantomJS(path)
    # 发起url请求
    bro.get(url)
    time.sleep(3)
    # 截图
    bro.save_screenshot('1.png')

    # 执行js代码（让滚动条向下偏移n个像素（作用：动态加载了更多的电影信息））
    js = 'window.scrollTo(0,document.body.scrollHeight)'
    bro.execute_script(js)  # 该函数可以执行一组字符串形式的js代码
    time.sleep(2)

    bro.execute_script(js)  # 该函数可以执行一组字符串形式的js代码
    time.sleep(2)
    bro.save_screenshot('2.png') 
    time.sleep(2) 
    # 使用爬虫程序爬去当前url中的内容 
    html_source = bro.page_source # 该属性可以获取当前浏览器的当前页的源码（html） 
    with open('./source.html', 'w', encoding='utf-8') as fp: 
        fp.write(html_source) 
    bro.quit()</code></pre>
<img class="cke_reset cke_widget_mask" src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" alt="" /><img class="cke_reset cke_widget_drag_handler" title="点击并拖拽以移动" src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" alt="" width="15" height="15" data-cke-widget-drag-handler="1" />
 
</li>

四.谷歌无头浏览器

- 由于PhantomJs最近已经停止了更新和维护，所以推荐大家可以使用谷歌的无头浏览器，是一款无界面的谷歌浏览器。
<li>代码展示：

<pre class="cke_widget_element" data-cke-widget-data="%7B%22code%22%3A%22from%20selenium%20import%20webdriver%5Cnfrom%20selenium.webdriver.chrome.options%20import%20Options%5Cnimport%20time%5Cn%20%5Cn%23%20%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%8F%82%E6%95%B0%E5%AF%B9%E8%B1%A1%EF%BC%8C%E7%94%A8%E6%9D%A5%E6%8E%A7%E5%88%B6chrome%E4%BB%A5%E6%97%A0%E7%95%8C%E9%9D%A2%E6%A8%A1%E5%BC%8F%E6%89%93%E5%BC%80%5Cnchrome_options%20%3D%20Options()%5Cnchrome_options.add_argument('--headless')%5Cnchrome_options.add_argument('--disable-gpu')%5Cn%23%20%E9%A9%B1%E5%8A%A8%E8%B7%AF%E5%BE%84%5Cnpath%20%3D%20r'C%3A%5C%5CUsers%5C%5CZBLi%5C%5CDesktop%5C%5C1801%5C%5Cday05%5C%5Cziliao%5C%5Cchromedriver.exe'%5Cn%20%5Cn%23%20%E5%88%9B%E5%BB%BA%E6%B5%8F%E8%A7%88%E5%99%A8%E5%AF%B9%E8%B1%A1%5Cnbrowser%20%3D%20webdriver.Chrome(executable_path%3Dpath%2C%20chrome_options%3Dchrome_options)%5Cn%20%5Cn%23%20%E4%B8%8A%E7%BD%91%5Cnurl%20%3D%20'http%3A%2F%2Fwww.baidu.com%2F'%5Cnbrowser.get(url)%5Cntime.sleep(3)%5Cn%20%5Cnbrowser.save_screenshot('baidu.png')%5Cn%20%5Cnbrowser.quit()%22%2C%22classes%22%3Anull%7D" data-cke-widget-upcasted="1" data-cke-widget-keep-attr="0" data-widget="codeSnippet"><code class="hljs python">from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time
 
# 创建一个参数对象，用来控制chrome以无界面模式打开
chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--disable-gpu')
# 驱动路径
path = r'C:\Users\ZBLi\Desktop\1801\day05\ziliao\chromedriver.exe'
 
# 创建浏览器对象
browser = webdriver.Chrome(executable_path=path, chrome_options=chrome_options)
 
# 上网
url = 'http://www.baidu.com/'
browser.get(url)
time.sleep(3)
 
browser.save_screenshot('baidu.png')
 
browser.quit()</code></pre>
<img class="cke_reset cke_widget_mask" src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" alt="" /><img class="cke_reset cke_widget_drag_handler" title="点击并拖拽以移动" src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" alt="" width="15" height="15" data-cke-widget-drag-handler="1" />
</li>

作业

- 爬取网易新闻国内板块下的新闻标题和新闻内容
