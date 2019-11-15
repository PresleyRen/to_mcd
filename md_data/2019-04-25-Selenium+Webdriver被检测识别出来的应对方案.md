---
layout:     post
title:      Selenium+Webdriver被检测识别出来的应对方案
subtitle:   
date:       2019-04-25
author:     P
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - python
---
在写爬虫，面对很多js 加载的页面，很多人束手无策，更多的人喜欢用Senlenium+ Webdriver，古语有云：道高一尺魔高一丈。已淘宝为首，众多网站都针对 Selenium的js监测机制， 比如：window.navigator.webdriver，navigator.languages，navigator.plugins.length

正常情况下我们用浏览器访问淘宝等网站的 window.navigator.webdriver的值为 undefined。 





<img class="wp-image-86" src="http://www.cnseu.net/wp-content/uploads/2019/02/image.png" alt="" />

当我们用selenium 的时候， window.navigator.webdriver的值为 true。





<img class="wp-image-87" src="http://www.cnseu.net/wp-content/uploads/2019/02/image-1.png" alt="" />

那么如何解决这个问题呢？  

第一种：使用mitmproxy用中间人的方式截取服务器发送来的js，修改js里面函数的参值方式发送给服务器。相当于在browser和server之间做一层中介的拦截。不过此方法要对js非常熟悉的人才好实施。

第二种方法依旧通过selenium，不过是在服务器在第一次发送js并在本地验证的时候，做好第一次的伪装，从而实现第一次登陆有效。。方法简单，适合小白。

之前我写过一次用 pyppeteer 加 asyncio 绕过selenium检测的方案，对于新手来说比较麻烦，现在我有了更好的解决方案。

只需要设置Chromedriver的启动参数即可解决问题。

在启动Chromedriver之前，为Chrome开启实验性功能参数`excludeSwitches`，它的值为`['enable-automation']`，完整代码如下：

```
<code>
from selenium.webdriver import Chrome
from selenium.webdriver import ChromeOptions

option = ChromeOptions()
option.add_experimental_option('excludeSwitches', ['enable-automation'])
driver = Chrome(options=option)</code>
```

此时启动的Chrome窗口，在右上角会弹出一个提示，不用管它，不要点击`停用`按钮。

再次在开发者工具的Console选项卡中查询`window.navigator.webdriver`，可以发现这个值已经自动变成`undefined`了。并且无论你打开新的网页，开启新的窗口还是点击链接进入其他页面，都不会让它变成`true`。运行效果如下图所示。

<img class="wp-image-88" src="http://www.cnseu.net/wp-content/uploads/2019/02/image-2.png" alt="" />
