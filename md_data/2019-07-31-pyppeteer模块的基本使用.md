---
layout:     post
title:      pyppeteer模块的基本使用
subtitle:   
date:       2019-07-31
author:     P
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - python
---
## **引言**

## **Pyppeteer简介**

注意，本节讲解的模块叫做 Pyppeteer，不是 Puppeteer。Puppeteer 是 Google 基于 Node.js 开发的一个工具，有了它我们可以通过 JavaScript 来控制 Chrome 浏览器的一些操作，当然也可以用作网络爬虫上，其 API 极其完善，功能非常强大。 而 Pyppeteer 又是什么呢？它实际上是 Puppeteer 的 Python 版本的实现，但他不是 Google 开发的，是一位来自于日本的工程师依据 Puppeteer 的一些功能开发出来的非官方版本。

在 Pyppetter 中，实际上它背后也是有一个类似 Chrome 浏览器的 Chromium 浏览器在执行一些动作进行网页渲染，首先说下 Chrome 浏览器和 Chromium 浏览器的渊源。

```
 
```

1. `Chromium 是谷歌为了研发 Chrome 而启动的项目，是完全开源的。二者基于相同的源代码构建，Chrome 所有的新功能都会先在 Chromium 上实现，待验证稳定后才会移植，因此 Chromium 的版本更新频率更高，也会包含很多新的功能，但作为一款独立的浏览器，Chromium 的用户群体要小众得多。两款浏览器同根同源，它们有着同样的 Logo，但配色不同，Chrome 由蓝红绿黄四种颜色组成，而 Chromium 由不同深度的蓝色构成。`

<img src="https://book.apeland.cn/media/images/2019/04/28/snip20190428_1.png" alt="" />

## **环境安装**

- 由于 Pyppeteer 采用了 Python 的 async 机制，所以其运行要求的 Python 版本为 3.5 及以上
- pip install pyppeteer

## **快速上手**

- 爬取[http://quotes.toscrape.com/js/](http://quotes.toscrape.com/js/) 全部页面数据

```
 
```

1. `import asyncio`
1. `from pyppeteer import launch`
1. `from lxml import etree`
1.  
1. `async def main():`
1. `browser = await launch()`
1. `page = await browser.newPage()`
1. `await page.goto('http://quotes.toscrape.com/js/')`
1. `page_text = await page.content()`
1. `tree = etree.HTML(page_text)`
1. `div_list = tree.xpath('//div[@class="quote"]')`
1. `print(len(div_list))`
1. `await browser.close()`
1.  
1. `asyncio.get_event_loop().run_until_complete(main())`

运行结果：10解释：launch 方法会新建一个 Browser 对象，然后赋值给 browser，然后调用 newPage 方法相当于浏览器中新建了一个选项卡，同时新建了一个 Page 对象。然后 Page 对象调用了 goto 方法就相当于在浏览器中输入了这个 URL，浏览器跳转到了对应的页面进行加载，加载完成之后再调用 content 方法，返回当前浏览器页面的源代码。然后进一步地，我们用 pyquery 进行同样地解析，就可以得到 JavaScript 渲染的结果了。在这个过程中，我们没有配置 Chrome 浏览器，没有配置浏览器驱动，免去了一些繁琐的步骤，同样达到了 Selenium 的效果，还实现了异步抓取，爽歪歪！

## 详细用法

<li>开启浏览器
<ul>
<li>调用 launch 方法即可，相关参数介绍：
<ul>
- ignoreHTTPSErrors (bool): 是否要忽略 HTTPS 的错误，默认是 False。
- headless (bool): 是否启用 Headless 模式，即无界面模式，如果 devtools 这个参数是 True 的话，那么该参数就会被设置为 False，否则为 True，即默认是开启无界面模式的。
- executablePath (str): 可执行文件的路径，如果指定之后就不需要使用默认的 Chromium 了，可以指定为已有的 Chrome 或 Chromium。
- args (List[str]): 在执行过程中可以传入的额外参数。
- devtools (bool): 是否为每一个页面自动开启调试工具，默认是 False。如果这个参数设置为 True，那么 headless 参数就会无效，会被强制设置为 False。



```
 
```

1. `browser = await launch(headless=False, args=['--disable-infobars'])`

处理页面显示问题:访问淘宝首页

```
 
```

1. `import asyncio`
1. `from pyppeteer import launch`
1.  
1. `async def main():`
1. `browser = await launch(headless=False)`
1. `page = await browser.newPage()`
1. `await page.goto('https://www.taobao.com')`
1. `await asyncio.sleep(10)`
1.  
1. `asyncio.get_event_loop().run_until_complete(main())`

<img src="https://book.apeland.cn/media/images/2019/05/26/Snip20190524_16.png" alt="" />发现页面显示出现了问题，需要手动调用setViewport方法设置显示页面的长宽像素。设置如下：

```
 
```

1. `import asyncio`
1. `from pyppeteer import launch`
1.  
1. `width, height = 1366, 768`
1.  
1. `async def main():`
1. `browser = await launch(headless=False)`
1. `page = await browser.newPage()`
1. `await page.setViewport({'width': width, 'height': height})`
1. `await page.goto('https://www.taobao.com')`
1. `await asyncio.sleep(3)`
1.  
1. `asyncio.get_event_loop().run_until_complete(main())`

执行js程序：拖动滚轮。调用evaluate方法。

```
 
```

1. `import asyncio`
1. `from pyppeteer import launch`
1. `width, height = 1366, 768`
1. `async def main():`
1. `browser = await launch(headless=False)`
1. `page = await browser.newPage()`
1. `await page.setViewport({'width': width, 'height': height})`
1. `await page.goto('https://movie.douban.com/typerank?type_name=%E5%8A%A8%E4%BD%9C&type=5&interval_id=100:90&action=')`
1. `await asyncio.sleep(3)`
1. `#evaluate可以返回js程序的返回值`
1. `dimensions = await page.evaluate('window.scrollTo(0,document.body.scrollHeight)')`
1. `await asyncio.sleep(3)`
1. `print(dimensions)`
1. `await browser.close()`
1.  
1. `asyncio.get_event_loop().run_until_complete(main())`

规避webdriver检测：

```
 
```

1. `import asyncio`
1. `from pyppeteer import launch`
1. `async def main():`
1. `browser = await launch(headless=False, args=['--disable-infobars'])`
1. `page = await browser.newPage()`
1. `await page.goto('https://login.taobao.com/member/login.jhtml?redirectURL=https://www.taobao.com/')`
1. `await page.evaluate(`
1. `'''() =>{ Object.defineProperties(navigator,{ webdriver:{ get: () => false } }) }''')`
1. `await asyncio.sleep(10)`
1.  
1. `asyncio.get_event_loop().run_until_complete(main())`

```
 
```

1. `await self.page.setUserAgent('xxx')`

节点交互

```
 
```

1. `import asyncio`
1. `from pyppeteer import launch`
1. `async def main():`
1. `# headless参数设为False，则变成有头模式`
1. `browser = await launch(`
1. `headless=False`
1. `)`
1.  
1. `page = await browser.newPage()`
1. `# 设置页面视图大小`
1. `await page.setViewport(viewport={'width': 1280, 'height': 800})`
1.  
1. `await page.goto('https://www.baidu.com/')`
1. `#节点交互`
1. `await page.type('#kw','周杰伦',{'delay': 1000})`
1. `await asyncio.sleep(3)`
1. `await page.click('#su')`
1. `await asyncio.sleep(3)`
1. `#使用选择器选中标签进行点击`
1. `alist = await page.querySelectorAll('.s_tab_inner > a')`
1. `a = alist[3]`
1. `await a.click()`
1. `await asyncio.sleep(3)`
1. `await browser.close()`
1. `asyncio.get_event_loop().run_until_complete(main())`

## 综合练习

爬取头条和网易的新闻标题

```
 
```

1. `import asyncio`
1. `from pyppeteer import launch`
1. `from lxml import etree`
1. `async def main():`
1. `# headless参数设为False，则变成有头模式`
1. `browser = await launch(`
1. `headless=False`
1. `)`
1.  
1. `page1 = await browser.newPage()`
1.  
1. `# 设置页面视图大小`
1. `await page1.setViewport(viewport={'width': 1280, 'height': 800})`
1.  
1. `await page1.goto('https://www.toutiao.com/')`
1. `await asyncio.sleep(2)`
1. `# 打印页面文本`
1. `page_text = await page1.content()`
1.  
1. `page2 = await browser.newPage()`
1. `await page2.setViewport(viewport={'width': 1280, 'height': 800})`
1. `await page2.goto('https://news.163.com/domestic/')`
1. `await page2.evaluate('window.scrollTo(0,document.body.scrollHeight)')`
1. `page_text1 = await page2.content()`
1.  
1. `await browser.close()`
1.  
1. `return {'wangyi':page_text1,'toutiao':page_text}`
1.  
1. `def parse(task):`
1. `content_dic = task.result()`
1. `wangyi = content_dic['wangyi']`
1. `toutiao = content_dic['toutiao']`
1. `tree = etree.HTML(toutiao)`
1. `a_list = tree.xpath('//div[[@class](https://github.com/class)="title-box"]/a')`
1. `for a in a_list:`
1. `title = a.xpath('./text()')[0]`
1. `print('toutiao:',title)`
1. `tree = etree.HTML(wangyi)`
1. `div_list = tree.xpath('//div[[@class](https://github.com/class)="data_row news_article clearfix "]')`
1. `print(len(div_list))`
1. `for div in div_list:`
1. `title = div.xpath('.//div[[@class](https://github.com/class)="news_title"]/h3/a/text()')[0]`
1. `print('wangyi:',title)`
1.  
1. `tasks = []`
1. `task1 = asyncio.ensure_future(main())`
1. `task1.add_done_callback(parse)`
1. `tasks.append(task1)`
1. `asyncio.get_event_loop().run_until_complete(asyncio.wait(tasks))`

爬取结果：toutiao: 「央视快评」坚守初心 为国奉献toutiao: 南航一A380客机北京降落时遭冰雹风挡现裂痕 已平安降落无人受伤toutiao: 美国正开启第二战场：围猎中国高科技企业 |双线作战战略意图toutiao: 云南省陆良县：农民给供销社打白条toutiao: 媒体：90后副县长若非靠拼爹上位 需拿出业绩服众toutiao: 南航A380飞北京客机遭遇冰雹袭击，挡风玻璃全碎toutiao: 秘鲁北部发生7.8级地震toutiao: 1958年，由捷克斯洛伐克援建的北京电影洗印厂曾为全国行业的老大toutiao: 一箭60星，发射成功！马斯克卫星互联网计划启动69wangyi: 中美经贸摩擦背后：有人在干，有人在骗wangyi: 华为回应个别标准组织撤销资格：产品服务不受影响wangyi: 隔空约架?中方主播刘欣23年前就赢得国际演讲比赛wangyi: 从钱学森到任正非 中国教育有多少底气应对全球化wangyi: 2个月内二度履新 35岁清华博士任安徽省直单位领导wangyi: 南阳水氢发动机汽车引热议 官方回应四大疑问wangyi: 31岁北大博士跻身县委常委 主笔6万字全县发展规划wangyi: 干部退休15年后投案自首 省委巡视办：头一次碰到wangyi: 台湾被标注＂中国台湾省＂ 台外事部门要求更正被拒wangyi: 190天3次现场办公!南阳领导为何钟爱青年汽车项目
