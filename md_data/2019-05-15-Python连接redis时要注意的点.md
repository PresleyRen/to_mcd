---
layout:     post
title:      Python连接redis时要注意的点
subtitle:   
date:       2019-05-15
author:     P
header-img: img/post-bg-ios10.jpg
catalog: true
tags:
    - python
---
一、一般连接redis情况

　　

{% raw %}
```
1 from redis import Redis
2 # 实例化redis对象
3 rdb = Redis(host='localhost', port=6379, db=0)
4 rdb.set('name', 'root')5 name = rdb.get('name')6 print(name)
```
{% endraw %}

　　这种情况连接数据库，对数据的存取都是字节类型，存取时还得转码一下，一般不推荐这种方法

二、连接池连接redis

　　

{% raw %}
```
1 from redis import ConnectionPool, Redis
2 pool = ConnectionPool(host='localhost', port=6379, db=0)
3 rdb = Redis(connection_pool=pool)
4 rdb.get('name')
```
{% endraw %}

　　这种连接池连接redis时也会有上述情况出现，所以一般也不推荐

三、redis连接的推荐方式

　　为了避免上述情况，redis在实例化的时候给了一个参数叫decode_response，默认值是False，如果我们把这个值改为True，则避免了转码流程，直接对原数据进行操作

{% raw %}
```
1 from redis import ConnectionPool, Redis
2 pool = ConnectionPool(host='localhost', port=6379, db=0, decode_responses=True)
3 rdb = Redis(connection_pool=pool)4 rdb.set('name2', 'rooter')
5 name2 = rdb.get('name2')6 print(name2)
```
{% endraw %}
