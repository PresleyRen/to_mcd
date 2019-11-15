---
layout:     post
title:      基于scrapy-redis两种形式的分布式爬虫
subtitle:   
date:       2019-03-25
author:     P
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - python
---
redis分布式部署

　　　　- 不可以。原因有二。

　　　　　　其一：因为多台机器上部署的scrapy会各自拥有各自的调度器，这样就使得多台机器无法分配start_urls列表中的url。（多台机器无法共享同一个调度器）

　　　　　　其二：多台机器爬取到的数据无法通过同一个管道对数据进行统一的数据持久出存储。（多台机器无法共享同一个管道）

{% raw %}
```
<code class="hljs perl">- 注释该行：bind 127.0.0.1，表示可以让其他ip访问redis

- 将yes该为no：protected-mode no，表示可以让其他ip操作redis</code>
```
{% endraw %}

{% raw %}
```
- 添加一个新属性：redis_key = 'xxx'(调度器队列的名称)
```
{% endraw %}

{% raw %}
```
<code class="hljs makefile">ITEM_PIPELINES = {
    'scrapy_redis.pipelines.RedisPipeline': 400
}</code>
```
{% endraw %}

{% raw %}
```
<code class="hljs ini"># 使用scrapy-redis组件的去重队列
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
# 使用scrapy-redis组件自己的调度器
SCHEDULER = "scrapy_redis.scheduler.Scheduler"
# 是否允许暂停
SCHEDULER_PERSIST = True</code>
```
{% endraw %}

{% raw %}
```
<code class="hljs ini">REDIS_HOST = 'redis服务的ip地址'
REDIS_PORT = 6379
REDIS_ENCODING = utf-8
REDIS_PARAMS = {password:123456}</code>
```
{% endraw %}
