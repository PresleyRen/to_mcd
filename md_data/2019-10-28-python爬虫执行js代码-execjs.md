---
layout:     post
title:      python爬虫执行js代码-execjs
subtitle:   
date:       2019-10-28
author:     P
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - python
---
## 一.安装模块

`pip install PyExecJS`

`execjs会自动使用当前电脑上的运行时环境（建议用nodejs，与Phantomjs）`

## 二.简单的使用

```
<code class="hljs">import execjs

js_obj = execjs.compile('js字符串')
js_obj.call('js字符串中方法',参数)</code>
```

## 三.js字符串中模拟浏览器环境

即导入`document`与`window`对象

### 一.安装依赖

`npm install jsdom`

### 二.导入包

```
`js_obj = execjs.compile('js字符串',cwd='node_modules')`
```

### 三.js字符串中添加抬头

```
<code class="hljs">const jsdom = require("jsdom");
const { JSDOM } = jsdom;
const dom = new JSDOM(`<!DOCTYPE html><p>Hello world</p>`);
window = dom.window;
document = window.document;
XMLHttpRequest = window.XMLHttpRequest;</code>
```

### 四.其它方案

```
`使用PhantomJS之前，需要下载它的[驱动](http://phantomjs.org/download.html)，然后放下Python代码统一目录下。对之前的Python代码也进行修改：`
```

```
import execjs
 
import os
os.environ["EXECJS_RUNTIME"] = "PhantomJS"
node = execjs.get()
file = 'eleme.js'
ctx = node.compile(open(file).read())
js_encode = 'getParam()'
params = ctx.eval(js_encode)
print(params)
```

模拟浏览器用的selenium和chrome的webDriver（效率高点，可以自打开一个浏览器（都调用一个webdriver对象）），代码如下：

```
from selenium import webdriver
 
browser = webdriver.Chrome(executable_path='chromedriver.exe')
with open('eleme.js', 'r') as f:
    js = f.read()
print(browser.execute_script(js))
```

```
` `
```
