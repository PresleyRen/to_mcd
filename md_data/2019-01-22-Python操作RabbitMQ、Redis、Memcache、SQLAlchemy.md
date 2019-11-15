---
layout:     post
title:      Python操作RabbitMQ、Redis、Memcache、SQLAlchemy
subtitle:   
date:       2019-01-22
author:     P
header-img: img/post-bg-alibaba.jpg
catalog: true
tags:
    - python
---
### Memcached

Memcached 是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高动态、数据库驱动网站的速度。Memcached基于一个存储键/值对的[hashmap](http://baike.baidu.com/view/1487140.htm)。其[守护进程](http://baike.baidu.com/view/53123.htm)（daemon ）是用[C](http://baike.baidu.com/subview/10075/6770152.htm)写的，但是客户端可以用任何语言来编写，并通过memcached协议与守护进程通信。

Memcached安装和基本使用

Memcached安装：
<td class="gutter">12345678</td><td class="code">`wget http:``/``/``memcached.org``/``latest``tar ``-``zxvf memcached``-``1.x``.x.tar.gz``cd memcached``-``1.x``.x``.``/``configure && make && make test && sudo make install` `PS：依赖libevent``       ``yum install libevent``-``devel``       ``apt``-``get install libevent``-``dev`</td>

启动Memcached
<td class="gutter">12345678910</td><td class="code">`memcached ``-``d ``-``m ``10`    `-``u root ``-``l ``10.211``.``55.4` `-``p ``12000` `-``c ``256` `-``P ``/``tmp``/``memcached.pid` `参数说明:``    ``-``d 是启动一个守护进程``    ``-``m 是分配给Memcache使用的内存数量，单位是MB``    ``-``u 是运行Memcache的用户``    ``-``l 是监听的服务器IP地址``    ``-``p 是设置Memcache监听的端口,最好是``1024``以上的端口``    ``-``c 选项是最大运行的并发连接数，默认是``1024``，按照你服务器的负载量来设定``    ``-``P 是设置保存Memcache的pid文件`</td>

Memcached命令
<td class="gutter">123</td><td class="code">`存储命令: ``set``/``add``/``replace``/``append``/``prepend``/``cas ``获取命令: get``/``gets``其他命令: delete``/``stats..`</td>

安装API
<td class="gutter">12</td><td class="code">`python操作Memcached使用Python``-``memcached模块``下载安装：https:``/``/``pypi.python.org``/``pypi``/``python``-``memcached`</td>

**1、第一次操作**
<td class="gutter">123456</td><td class="code">`import` `memcache` `mc ``=` `memcache.Client([``'10.211.55.4:12000'``], debug``=``True``)``mc.``set``(``"foo"``, ``"bar"``)``ret ``=` `mc.get(``'foo'``)``print` `ret`</td>

**Ps：debug = True 表示运行出现错误时，现实错误信息，上线后移除该参数。**

**2、天生支持集群**
<td class="gutter">1234567</td><td class="code">`     ``主机    权重``    ``1.1``.``1.1`   `1``    ``1.1``.``1.2`   `2``    ``1.1``.``1.3`   `1` `那么在内存中主机列表为：``    ``host_list ``=` `[``"1.1.1.1"``, ``"1.1.1.2"``, ``"1.1.1.2"``, ``"1.1.1.3"``, ]`</td>

如果用户根据如果要在内存中创建一个键值对（如：k1 = "v1"），那么要执行一下步骤：

- 根据算法将 k1 转换成一个数字
- 将数字和主机列表长度求余数，得到一个值 N（ 0 <= N < 列表长度 ）
- 在主机列表中根据 第2步得到的值为索引获取主机，例如：host_list[N]
- 连接 将第3步中获取的主机，将 k1 = "v1" 放置在该服务器的内存中

代码实现如下：
<td class="gutter">123</td><td class="code">`mc ``=` `memcache.Client([(``'1.1.1.1:12000'``, ``1``), (``'1.1.1.2:12000'``, ``2``), (``'1.1.1.3:12000'``, ``1``)], debug``=``True``)` `mc.``set``(``'k1'``, ``'v1'``)`</td>

**3、add**添加一条键值对，如果已经存在的 key，重复执行add操作异常
<td class="gutter">1234567</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-``import` `memcache` `mc ``=` `memcache.Client([``'10.211.55.4:12000'``], debug``=``True``)``mc.add(``'k1'``, ``'v1'``)``# mc.add('k1', 'v2') # 报错，对已经存在的key重复添加，失败！！！`</td>

**4、replace**replace  修改某个key的值，如果key不存在，则异常
<td class="gutter">1234567</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-``import` `memcache` `mc ``=` `memcache.Client([``'10.211.55.4:12000'``], debug``=``True``)``# 如果memcache中存在kkkk，则替换成功，否则一场``mc.replace(``'kkkk'``,``'999'``)`</td>

**5、set 和 set_multi**
<td class="gutter">123456789</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-``import` `memcache` `mc ``=` `memcache.Client([``'10.211.55.4:12000'``], debug``=``True``)` `mc.``set``(``'key0'``, ``'wupeiqi'``)` `mc.set_multi({``'key1'``: ``'val1'``, ``'key2'``: ``'val2'``})`</td>

**6、delete 和 delete_multi**

delete             在Memcached中删除指定的一个键值对delete_multi    在Memcached中删除指定的多个键值对
<td class="gutter">12345678</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-``import` `memcache` `mc ``=` `memcache.Client([``'10.211.55.4:12000'``], debug``=``True``)` `mc.delete(``'key0'``)``mc.delete_multi([``'key1'``, ``'key2'``])`</td>

**7、get 和 get_multi**

get            获取一个键值对get_multi   获取多一个键值对
<td class="gutter">12345678</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-``import` `memcache` `mc ``=` `memcache.Client([``'10.211.55.4:12000'``], debug``=``True``)` `val ``=` `mc.get(``'key0'``)``item_dict ``=` `mc.get_multi([``"key1"``, ``"key2"``, ``"key3"``])`</td>

**8、append 和 prepend**

append    修改指定key的值，在该值 后面 追加内容prepend   修改指定key的值，在该值 前面 插入内容
<td class="gutter">123456789101112</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-``import` `memcache` `mc ``=` `memcache.Client([``'10.211.55.4:12000'``], debug``=``True``)``# k1 = "v1"` `mc.append(``'k1'``, ``'after'``)``# k1 = "v1after"` `mc.prepend(``'k1'``, ``'before'``)``# k1 = "beforev1after"`</td>

**9、decr 和 incr　**　

incr  自增，将Memcached中的某一个值增加 N （ N默认为1 ）decr 自减，将Memcached中的某一个值减少 N （ N默认为1 ）
<td class="gutter">123456789101112131415161718</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-``import` `memcache` `mc ``=` `memcache.Client([``'10.211.55.4:12000'``], debug``=``True``)``mc.``set``(``'k1'``, ``'777'``)` `mc.incr(``'k1'``)``# k1 = 778` `mc.incr(``'k1'``, ``10``)``# k1 = 788` `mc.decr(``'k1'``)``# k1 = 787` `mc.decr(``'k1'``, ``10``)``# k1 = 777`</td>

**10、gets 和 cas**

如商城商品剩余个数，假设改值保存在memcache中，product_count = 900A用户刷新页面从memcache中读取到product_count = 900B用户刷新页面从memcache中读取到product_count = 900

如果A、B用户均购买商品

A用户修改商品剩余个数 product_count＝899B用户修改商品剩余个数 product_count＝899  

如此一来缓存内的数据便不在正确，两个用户购买商品后，商品剩余还是 899如果使用python的set和get来操作以上过程，那么程序就会如上述所示情况！

如果想要避免此情况的发生，只要使用 gets 和 cas 即可，如：
<td class="gutter">123456789</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-``import` `memcache``mc ``=` `memcache.Client([``'10.211.55.4:12000'``], debug``=``True``, cache_cas``=``True``)` `v ``=` `mc.gets(``'product_count'``)``# ...``# 如果有人在gets之后和cas之前修改了product_count，那么，下面的设置将会执行失败，剖出异常，从而避免非正常数据的产生``mc.cas(``'product_count'``, ``"899"``)`</td>

[Memcached 真的过时了吗？](http://www.oschina.net/news/26691/memcached-timeout)

### Redis

redis是一个key-value[存储系统](http://baike.baidu.com/view/51839.htm)。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list([链表](http://baike.baidu.com/view/549479.htm))、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些[数据类型](http://baike.baidu.com/view/675645.htm)都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

{% raw %}
```
1. 使用Redis有哪些好处？

(1) 速度快，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)

(2) 支持丰富数据类型，支持string，list，set，sorted set，hash

(3) 支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行

(4) 丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除


2. redis相比memcached有哪些优势？

(1) memcached所有的值均是简单的字符串，redis作为其替代者，支持更为丰富的数据类型

(2) redis的速度比memcached快很多

(3) redis可以持久化其数据


3. redis常见性能问题和解决方案：

(1) Master最好不要做任何持久化工作，如RDB内存快照和AOF日志文件

(2) 如果数据比较重要，某个Slave开启AOF备份数据，策略设置为每秒同步一次

(3) 为了主从复制的速度和连接的稳定性，Master和Slave最好在同一个局域网内

(4) 尽量避免在压力很大的主库上增加从库

(5) 主从复制不要用图状结构，用单向链表结构更为稳定，即：Master <- Slave1 <- Slave2 <- Slave3...

这样的结构方便解决单点故障问题，实现Slave对Master的替换。如果Master挂了，可以立刻启用Slave1做Master，其他不变。



 

4. MySQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据

 相关知识：redis 内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略。redis 提供 6种数据淘汰策略：

voltile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰

volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰

volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰

allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰

allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰

no-enviction（驱逐）：禁止驱逐数据

 

5. Memcache与Redis的区别都有哪些？

1)、存储方式

Memecache把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小。

Redis有部份存在硬盘上，这样能保证数据的持久性。

2)、数据支持类型

Memcache对数据类型支持相对简单。

Redis有复杂的数据类型。


3），value大小

redis最大可以达到1GB，而memcache只有1MB



6. Redis 常见的性能问题都有哪些？如何解决？

 

1).Master写内存快照，save命令调度rdbSave函数，会阻塞主线程的工作，当快照比较大时对性能影响是非常大的，会间断性暂停服务，所以Master最好不要写内存快照。


2).Master AOF持久化，如果不重写AOF文件，这个持久化方式对性能的影响是最小的，但是AOF文件会不断增大，AOF文件过大会影响Master重启的恢复速度。Master最好不要做任何持久化工作，包括内存快照和AOF日志文件，特别是不要启用内存快照做持久化,如果数据比较关键，某个Slave开启AOF备份数据，策略为每秒同步一次。

 
3).Master调用BGREWRITEAOF重写AOF文件，AOF在重写的时候会占大量的CPU和内存资源，导致服务load过高，出现短暂服务暂停现象。

4). Redis主从复制的性能问题，为了主从复制的速度和连接的稳定性，Slave和Master最好在同一个局域网内




7, redis 最适合的场景


Redis最适合所有数据in-momory的场景，虽然Redis也提供持久化功能，但实际更多的是一个disk-backed的功能，跟传统意义上的持久化有比较大的差别，那么可能大家就会有疑问，似乎Redis更像一个加强版的Memcached，那么何时使用Memcached,何时使用Redis呢?

       如果简单地比较Redis与Memcached的区别，大多数都会得到以下观点：

     1 、Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
     2 、Redis支持数据的备份，即master-slave模式的数据备份。
     3 、Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。

（1）、会话缓存（Session Cache）

最常用的一种使用Redis的情景是会话缓存（session cache）。用Redis缓存会话比其他存储（如Memcached）的优势在于：Redis提供持久化。当维护一个不是严格要求一致性的缓存时，如果用户的购物车信息全部丢失，大部分人都会不高兴的，现在，他们还会这样吗？

幸运的是，随着 Redis 这些年的改进，很容易找到怎么恰当的使用Redis来缓存会话的文档。甚至广为人知的商业平台Magento也提供Redis的插件。

（2）、全页缓存（FPC）

除基本的会话token之外，Redis还提供很简便的FPC平台。回到一致性问题，即使重启了Redis实例，因为有磁盘的持久化，用户也不会看到页面加载速度的下降，这是一个极大改进，类似PHP本地FPC。

再次以Magento为例，Magento提供一个插件来使用Redis作为全页缓存后端。

此外，对WordPress的用户来说，Pantheon有一个非常好的插件  wp-redis，这个插件能帮助你以最快速度加载你曾浏览过的页面。

（3）、队列

Reids在内存存储引擎领域的一大优点是提供 list 和 set 操作，这使得Redis能作为一个很好的消息队列平台来使用。Redis作为队列使用的操作，就类似于本地程序语言（如Python）对 list 的 push/pop 操作。

如果你快速的在Google中搜索Redis queues，你马上就能找到大量的开源项目，这些项目的目的就是利用Redis创建非常好的后端工具，以满足各种队列需求。例如，Celery有一个后台就是使用Redis作为broker，你可以从这里去查看。

（4），排行榜/计数器

Redis在内存中对数字进行递增或递减的操作实现的非常好。集合（Set）和有序集合（Sorted Set）也使得我们在执行这些操作的时候变的非常简单，Redis只是正好提供了这两种数据结构。所以，我们要从排序集合中获取到排名最靠前的10个用户我们称之为user_scores，我们只需要像下面一样执行即可：

当然，这是假定你是根据你用户的分数做递增的排序。如果你想返回用户及用户的分数，你需要这样执行：

ZRANGE user_scores 0 10 WITHSCORES

Agora Games就是一个很好的例子，用Ruby实现的，它的排行榜就是使用Redis来存储数据的，你可以在这里看到。

（5）、发布/订阅

最后（但肯定不是最不重要的）是Redis的发布/订阅功能。发布/订阅的使用场景确实非常多。我已看见人们在社交网络连接中使用，还可作为基于发布/订阅的脚本触发器，甚至用Redis的发布/订阅功能来建立聊天系统！（不，这是真的，你可以去核实）。

Redis提供的所有特性中，我感觉这个是喜欢的人最少的一个，虽然它为用户提供如果此多功能。
```
{% endraw %}

一、Redis安装和基本使用
<td class="gutter">1234</td><td class="code">`wget http:``/``/``download.redis.io``/``releases``/``redis``-``3.0``.``6.tar``.gz``tar xzf redis``-``3.0``.``6.tar``.gz``cd redis``-``3.0``.``6``make`</td>

启动服务端
<td class="gutter">1</td><td class="code">`src``/``redis``-``server`</td>

启动客户端
<td class="gutter">12345</td><td class="code">`src``/``redis``-``cli``redis> ``set` `foo bar``OK``redis> get foo``"bar"`</td>

二、Python操作Redis
<td class="gutter">1234567</td><td class="code">`sudo pip install redis``or``sudo easy_install redis``or``源码安装` `详见：https:``/``/``github.com``/``WoLpH``/``redis``-``py`</td>

API使用

redis-py 的API的使用可以分类为：

- 连接方式
- 连接池
<li>操作
<ul>
- String 操作
- Hash 操作
- List 操作
- Set 操作
- Sort Set 操作

1、操作模式

redis-py提供两个类Redis和StrictRedis用于实现Redis的命令，StrictRedis用于实现大部分官方的命令，并使用官方的语法和命令，Redis是StrictRedis的子类，用于向后兼容旧版本的redis-py。
<td class="gutter">12345678</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-` `import` `redis` `r ``=` `redis.Redis(host``=``'10.211.55.4'``, port``=``6379``)``r.``set``(``'foo'``, ``'Bar'``)``print` `r.get(``'foo'``)`</td>

2、连接池

redis-py使用connection pool来管理对一个redis server的所有连接，避免每次建立、释放连接的开销。默认，每个Redis实例都会维护一个自己的连接池。可以直接建立一个连接池，然后作为参数Redis，这样就可以实现多个Redis实例共享一个连接池。
<td class="gutter">12345678910</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-` `import` `redis` `pool ``=` `redis.ConnectionPool(host``=``'10.211.55.4'``, port``=``6379``)` `r ``=` `redis.Redis(connection_pool``=``pool)``r.``set``(``'foo'``, ``'Bar'``)``print` `r.get(``'foo'``)`</td>

3、操作

<img style="display: block; margin-left: auto; margin-right: auto;" src="https://images2015.cnblogs.com/blog/425762/201602/425762-20160222213200645-359371350.png" alt="" width="312" height="202" />

> 
set(name, value, ex=None, px=None, nx=False, xx=False)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
</td>
<td class="code">

`在Redis中设置值，默认，不存在则创建，存在则修改`
`参数：`
`     ``ex，过期时间（秒）`
`     ``px，过期时间（毫秒）`
`     ``nx，如果设置为True，则只有name不存在时，当前set操作才执行`
`     ``xx，如果设置为True，则只有name存在时，岗前set操作才执行`

</td>
</tr>
</tbody>
</table>



setnx(name, value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`设置值，只有name不存在时，执行设置操作（添加）`

</td>
</tr>
</tbody>
</table>



setex(name, value, time)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
</td>
<td class="code">

`# 设置值`
`# 参数：`
`    ``# time，过期时间（数字秒 或 timedelta对象）`

</td>
</tr>
</tbody>
</table>



psetex(name, time_ms, value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
</td>
<td class="code">

`# 设置值`
`# 参数：`
`    ``# time_ms，过期时间（数字毫秒 或 timedelta对象）`

</td>
</tr>
</tbody>
</table>



mset(*args, **kwargs)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
</td>
<td class="code">

`批量设置值`
`如：`
`    ``mset(k1=``'v1'``, k2=``'v2'``)`
`    ``或`
`    ``mget({``'k1'``: ``'v1'``, ``'k2'``: ``'v2'``})`

</td>
</tr>
</tbody>
</table>



get(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`获取值`

</td>
</tr>
</tbody>
</table>



mget(keys, *args)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
</td>
<td class="code">

`批量获取`
`如：`
`    ``mget(``'ylr'``, ``'wupeiqi'``)`
`    ``或`
`    ``r.mget([``'ylr'``, ``'wupeiqi'``])`

</td>
</tr>
</tbody>
</table>



getset(name, value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`设置新值并获取原来的值`

</td>
</tr>
</tbody>
</table>



getrange(key, start, end)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
</td>
<td class="code">

`# 获取子序列（根据字节获取，非字符）`
`# 参数：`
`    ``# name，Redis 的 name`
`    ``# start，起始位置（字节）`
`    ``# end，结束位置（字节）`
`# 如： "武沛齐" ，0-3表示 "武"`

</td>
</tr>
</tbody>
</table>



setrange(name, offset, value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
</td>
<td class="code">

`# 修改字符串内容，从指定字符串索引开始向后替换（新值太长时，则向后添加）`
`# 参数：`
`    ``# offset，字符串的索引，字节（一个汉字三个字节）`
`    ``# value，要设置的值`

</td>
</tr>
</tbody>
</table>



setbit(name, offset, value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
</td>
<td class="code">

`# 对name对应值的二进制表示的位进行操作`
 
`# 参数：`
`    ``# name，redis的name`
`    ``# offset，位的索引（将值变换成二进制后再进行索引）`
`    ``# value，值只能是 1 或 0`
 
`# 注：如果在Redis中有一个对应： n1 = "foo"，`
`        ``那么字符串foo的二进制表示为：``01100110` `01101111` `01101111`
`    ``所以，如果执行 setbit(``'n1'``, ``7``, ``1``)，则就会将第``7``位设置为``1``，`
`        ``那么最终二进制则变成 ``01100111` `01101111` `01101111``，即：``"goo"`
 
`# 扩展，转换二进制表示：`
 
`    ``# source = "武沛齐" `
`    ``source ``=` `"foo"`
 
`    ``for` `i ``in` `source:`
`        ``num ``=` `ord``(i)`
`        ``print` `bin``(num).replace(``'b'``,'')`
 
`    ``特别的，如果source是汉字 ``"武沛齐"``怎么办？`
`    ``答：对于utf``-``8``，每一个汉字占 ``3` `个字节，那么 ``"武沛齐"` `则有 ``9``个字节`
`       ``对于汉字，``for``循环时候会按照 字节 迭代，那么在迭代时，将每一个字节转换 十进制数，然后再将十进制数转换成二进制`
`        ``11100110` `10101101` `10100110` `11100110` `10110010` `10011011` `11101001` `10111101` `10010000`
`        ``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-` `-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-` `-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-`
`                    ``武                         沛                           齐`

</td>
</tr>
</tbody>
</table>



getbit(name, offset)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 获取name对应的值的二进制表示中的某位的值 （0或1）`

</td>
</tr>
</tbody>
</table>



bitcount(key, start=None, end=None)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
</td>
<td class="code">

`# 获取name对应的值的二进制表示中 1 的个数`
`# 参数：`
`    ``# key，Redis的name`
`    ``# start，位起始位置`
`    ``# end，位结束位置`

</td>
</tr>
</tbody>
</table>



bitop(operation, dest, *keys)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
9
10
</td>
<td class="code">

`# 获取多个值，并将值做位运算，将最后的结果保存至新的name对应的值`
 
`# 参数：`
`    ``# operation,AND（并） 、 OR（或） 、 NOT（非） 、 XOR（异或）`
`    ``# dest, 新的Redis的name`
`    ``# *keys,要查找的Redis的name`
 
`# 如：`
`    ``bitop(``"AND"``, ``'new_name'``, ``'n1'``, ``'n2'``, ``'n3'``)`
`    ``# 获取Redis中n1,n2,n3对应的值，然后讲所有的值做位运算（求并集），然后将结果保存 new_name 对应的值中`

</td>
</tr>
</tbody>
</table>



strlen(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 返回name对应值的字节长度（一个汉字3个字节）`

</td>
</tr>
</tbody>
</table>



incr(self, name, amount=1)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
</td>
<td class="code">

`# 自增 name对应的值，当name不存在时，则创建name＝amount，否则，则自增。`
 
`# 参数：`
`    ``# name,Redis的name`
`    ``# amount,自增数（必须是整数）`
 
`# 注：同incrby`

</td>
</tr>
</tbody>
</table>



incrbyfloat(self, name, amount=1.0)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
</td>
<td class="code">

`# 自增 name对应的值，当name不存在时，则创建name＝amount，否则，则自增。`
 
`# 参数：`
`    ``# name,Redis的name`
`    ``# amount,自增数（浮点型）`

</td>
</tr>
</tbody>
</table>



decr(self, name, amount=1)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
</td>
<td class="code">

`# 自减 name对应的值，当name不存在时，则创建name＝amount，否则，则自减。`
 
`# 参数：`
`    ``# name,Redis的name`
`    ``# amount,自减数（整数）`

</td>
</tr>
</tbody>
</table>



append(key, value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
</td>
<td class="code">

`# 在redis name对应的值后面追加内容`
 
`# 参数：`
`    ``key, redis的name`
`    ``value, 要追加的字符串`

</td>
</tr>
</tbody>
</table>



　　


Hash操作，redis中Hash在内存中的存储格式如下图：

<img style="display: block; margin-left: auto; margin-right: auto;" src="https://images2015.cnblogs.com/blog/425762/201602/425762-20160223115506630-113443460.png" alt="" width="345" height="232" />

> 
hset(name, key, value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
9
</td>
<td class="code">

`# name对应的hash中设置一个键值对（不存在，则创建；否则，修改）`
 
`# 参数：`
`    ``# name，redis的name`
`    ``# key，name对应的hash中的key`
`    ``# value，name对应的hash中的value`
 
`# 注：`
`    ``# hsetnx(name, key, value),当name对应的hash中不存在当前key时则创建（相当于添加）`

</td>
</tr>
</tbody>
</table>



hmset(name, mapping)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
</td>
<td class="code">

`# 在name对应的hash中批量设置键值对`
 
`# 参数：`
`    ``# name，redis的name`
`    ``# mapping，字典，如：{'k1':'v1', 'k2': 'v2'}`
 
`# 如：`
`    ``# r.hmset('xx', {'k1':'v1', 'k2': 'v2'})`

</td>
</tr>
</tbody>
</table>



hget(name,key)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 在name对应的hash中获取根据key获取value`

</td>
</tr>
</tbody>
</table>



hmget(name, keys, *args)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
9
10
11
</td>
<td class="code">

`# 在name对应的hash中获取多个key的值`
 
`# 参数：`
`    ``# name，reids对应的name`
`    ``# keys，要获取key集合，如：['k1', 'k2', 'k3']`
`    ``# *args，要获取的key，如：k1,k2,k3`
 
`# 如：`
`    ``# r.mget('xx', ['k1', 'k2'])`
`    ``# 或`
`    ``# print r.hmget('xx', 'k1', 'k2')`

</td>
</tr>
</tbody>
</table>



hgetall(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`获取name对应``hash``的所有键值`

</td>
</tr>
</tbody>
</table>



hlen(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 获取name对应的hash中键值对的个数`

</td>
</tr>
</tbody>
</table>



hkeys(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 获取name对应的hash中所有的key的值`

</td>
</tr>
</tbody>
</table>



hvals(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 获取name对应的hash中所有的value的值`

</td>
</tr>
</tbody>
</table>



hexists(name, key)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 检查name对应的hash是否存在当前传入的key`

</td>
</tr>
</tbody>
</table>



hdel(name,*keys)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 将name对应的hash中指定key的键值对删除`

</td>
</tr>
</tbody>
</table>



hincrby(name, key, amount=1)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
</td>
<td class="code">

`# 自增name对应的hash中的指定key的值，不存在则创建key=amount`
`# 参数：`
`    ``# name，redis中的name`
`    ``# key， hash对应的key`
`    ``# amount，自增数（整数）`

</td>
</tr>
</tbody>
</table>



hincrbyfloat(name, key, amount=1.0)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
</td>
<td class="code">

`# 自增name对应的hash中的指定key的值，不存在则创建key=amount`
 
`# 参数：`
`    ``# name，redis中的name`
`    ``# key， hash对应的key`
`    ``# amount，自增数（浮点数）`
 
`# 自增name对应的hash中的指定key的值，不存在则创建key=amount`

</td>
</tr>
</tbody>
</table>



hscan(name, cursor=0, match=None, count=None)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
9
10
11
12
13
</td>
<td class="code">

`# 增量式迭代获取，对于数据大的数据非常有用，hscan可以实现分片的获取数据，并非一次性将数据全部获取完，从而放置内存被撑爆`
 
`# 参数：`
`    ``# name，redis的name`
`    ``# cursor，游标（基于游标分批取获取数据）`
`    ``# match，匹配指定key，默认None 表示所有的key`
`    ``# count，每次分片最少获取个数，默认None表示采用Redis的默认分片个数`
 
`# 如：`
`    ``# 第一次：cursor1, data1 = r.hscan('xx', cursor=0, match=None, count=None)`
`    ``# 第二次：cursor2, data1 = r.hscan('xx', cursor=cursor1, match=None, count=None)`
`    ``# ...`
`    ``# 直到返回值cursor的值为0时，表示数据已经通过分片获取完毕`

</td>
</tr>
</tbody>
</table>



hscan_iter(name, match=None, count=None)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
9
</td>
<td class="code">

`# 利用yield封装hscan创建生成器，实现分批去redis中获取数据`
 
`# 参数：`
`    ``# match，匹配指定key，默认None 表示所有的key`
`    ``# count，每次分片最少获取个数，默认None表示采用Redis的默认分片个数`
 
`# 如：`
`    ``# for item in r.hscan_iter('xx'):`
`    ``#     print item`

</td>
</tr>
</tbody>
</table>



　　


List操作，redis中的List在在内存中按照一个name对应一个List来存储。如图：

<img style="display: block; margin-left: auto; margin-right: auto;" src="https://images2015.cnblogs.com/blog/425762/201602/425762-20160223172249115-189393001.png" alt="" width="280" height="177" />

> 
lpush(name,values)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
</td>
<td class="code">

`# 在name对应的list中添加元素，每个新的元素都添加到列表的最左边`
 
`# 如：`
`    ``# r.lpush('oo', 11,22,33)`
`    ``# 保存顺序为: 33,22,11`
 
`# 扩展：`
`    ``# rpush(name, values) 表示从右向左操作`

</td>
</tr>
</tbody>
</table>



lpushx(name,value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
</td>
<td class="code">

`# 在name对应的list中添加元素，只有name已经存在时，值添加到列表的最左边`
 
`# 更多：`
`    ``# rpushx(name, value) 表示从右向左操作`

</td>
</tr>
</tbody>
</table>



llen(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# name对应的list元素的个数`

</td>
</tr>
</tbody>
</table>



linsert(name, where, refvalue, value))



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
</td>
<td class="code">

`# 在name对应的列表的某一个值前或后插入一个新值`
 
`# 参数：`
`    ``# name，redis的name`
`    ``# where，BEFORE或AFTER`
`    ``# refvalue，标杆值，即：在它前后插入数据`
`    ``# value，要插入的数据`

</td>
</tr>
</tbody>
</table>



r.lset(name, index, value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
</td>
<td class="code">

`# 对name对应的list中的某一个索引位置重新赋值`
 
`# 参数：`
`    ``# name，redis的name`
`    ``# index，list的索引位置`
`    ``# value，要设置的值`

</td>
</tr>
</tbody>
</table>



r.lrem(name, value, num)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
</td>
<td class="code">

`# 在name对应的list中删除指定的值`
 
`# 参数：`
`    ``# name，redis的name`
`    ``# value，要删除的值`
`    ``# num，  num=0，删除列表中所有的指定值；`
`           ``# num=2,从前到后，删除2个；`
`           ``# num=-2,从后向前，删除2个`

</td>
</tr>
</tbody>
</table>



lpop(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
</td>
<td class="code">

`# 在name对应的列表的左侧获取第一个元素并在列表中移除，返回值则是第一个元素`
 
`# 更多：`
`    ``# rpop(name) 表示从右向左操作`

</td>
</tr>
</tbody>
</table>



lindex(name, index)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`在name对应的列表中根据索引获取列表元素`

</td>
</tr>
</tbody>
</table>



lrange(name, start, end)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
</td>
<td class="code">

`# 在name对应的列表分片获取数据`
`# 参数：`
`    ``# name，redis的name`
`    ``# start，索引的起始位置`
`    ``# end，索引结束位置`

</td>
</tr>
</tbody>
</table>



ltrim(name, start, end)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
</td>
<td class="code">

`# 在name对应的列表中移除没有在start-end索引之间的值`
`# 参数：`
`    ``# name，redis的name`
`    ``# start，索引的起始位置`
`    ``# end，索引结束位置`

</td>
</tr>
</tbody>
</table>



rpoplpush(src, dst)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
</td>
<td class="code">

`# 从一个列表取出最右边的元素，同时将其添加至另一个列表的最左边`
`# 参数：`
`    ``# src，要取数据的列表的name`
`    ``# dst，要添加数据的列表的name`

</td>
</tr>
</tbody>
</table>



blpop(keys, timeout)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
</td>
<td class="code">

`# 将多个列表排列，按照从左到右去pop对应列表的元素`
 
`# 参数：`
`    ``# keys，redis的name的集合`
`    ``# timeout，超时时间，当元素所有列表的元素获取完之后，阻塞等待列表内有数据的时间（秒）, 0 表示永远阻塞`
 
`# 更多：`
`    ``# r.brpop(keys, timeout)，从右向左获取数据`

</td>
</tr>
</tbody>
</table>



brpoplpush(src, dst, timeout=0)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
</td>
<td class="code">

`# 从一个列表的右侧移除一个元素并将其添加到另一个列表的左侧`
 
`# 参数：`
`    ``# src，取出并要移除元素的列表对应的name`
`    ``# dst，要插入元素的列表对应的name`
`    ``# timeout，当src对应的列表中没有数据时，阻塞等待其有数据的超时时间（秒），0 表示永远阻塞`

</td>
</tr>
</tbody>
</table>



自定义增量迭代



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
</td>
<td class="code">

`# 由于redis类库中没有提供对列表元素的增量迭代，如果想要循环name对应的列表的所有元素，那么就需要：`
`    ``# 1、获取name对应的所有列表`
`    ``# 2、循环列表`
`# 但是，如果列表非常大，那么就有可能在第一步时就将程序的内容撑爆，所有有必要自定义一个增量迭代的功能：`
 
`def` `list_iter(name):`
`    ``"""`
`    ``自定义redis列表增量迭代`
`    ``:param name: redis中的name，即：迭代name对应的列表`
`    ``:return: yield 返回 列表元素`
`    ``"""`
`    ``list_count ``=` `r.llen(name)`
`    ``for` `index ``in` `xrange``(list_count):`
`        ``yield` `r.lindex(name, index)`
 
`# 使用`
`for` `item ``in` `list_iter(``'pp'``):`
`    ``print` `item`

</td>
</tr>
</tbody>
</table>



Set操作，Set集合就是不允许重复的列表
sadd(name,values)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# name对应的集合中添加元素`

</td>
</tr>
</tbody>
</table>



scard(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`获取name对应的集合中元素个数`

</td>
</tr>
</tbody>
</table>



sdiff(keys, *args)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`在第一个name对应的集合中且不在其他name对应的集合的元素集合`

</td>
</tr>
</tbody>
</table>



sdiffstore(dest, keys, *args)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 获取第一个name对应的集合中且不在其他name对应的集合，再将其新加入到dest对应的集合中`

</td>
</tr>
</tbody>
</table>



sinter(keys, *args)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 获取多一个name对应集合的并集`

</td>
</tr>
</tbody>
</table>



sinterstore(dest, keys, *args)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 获取多一个name对应集合的并集，再讲其加入到dest对应的集合中`

</td>
</tr>
</tbody>
</table>



sismember(name, value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 检查value是否是name对应的集合的成员`

</td>
</tr>
</tbody>
</table>



smembers(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 获取name对应的集合的所有成员`

</td>
</tr>
</tbody>
</table>



smove(src, dst, value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 将某个成员从一个集合中移动到另外一个集合`

</td>
</tr>
</tbody>
</table>



spop(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 从集合的右侧（尾部）移除一个成员，并将其返回`

</td>
</tr>
</tbody>
</table>



srandmember(name, numbers)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 从name对应的集合中随机获取 numbers 个元素`

</td>
</tr>
</tbody>
</table>



srem(name, values)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 在name对应的集合中删除某些值`

</td>
</tr>
</tbody>
</table>



sunion(keys, *args)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 获取多一个name对应的集合的并集`

</td>
</tr>
</tbody>
</table>



sunionstore(dest,keys, *args)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
</td>
<td class="code">

`# 获取多一个name对应的集合的并集，并将结果保存到dest对应的集合中`

</td>
</tr>
</tbody>
</table>



sscan(name, cursor=0, match=None, count=None)sscan_iter(name, match=None, count=None)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 同字符串的操作，用于增量迭代分批获取元素，避免内存消耗太大`



</td>

</tr>

</tbody>

</table>







 




有序集合，在集合的基础上，为每元素排序；元素的排序需要根据另外一个值来进行比较，所以，对于有序集合，每一个元素有两个值，即：值和分数，分数专门用来做排序。

> 
zadd(name, *args, **kwargs)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5

</td>
<td class="code">

`# 在name对应的有序集合中添加元素`
`# 如：`
`     ``# zadd('zz', 'n1', 1, 'n2', 2)`
`     ``# 或`
`     ``# zadd('zz', n1=11, n2=22)`



</td>

</tr>

</tbody>

</table>







zcard(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 获取name对应的有序集合元素的数量`



</td>

</tr>

</tbody>

</table>







zcount(name, min, max)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 获取name对应的有序集合中分数 在 [min,max] 之间的个数`



</td>

</tr>

</tbody>

</table>







zincrby(name, value, amount)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 自增name对应的有序集合的 name 对应的分数`



</td>

</tr>

</tbody>

</table>







r.zrange( name, start, end, desc=False, withscores=False, score_cast_func=float)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18

</td>
<td class="code">

`# 按照索引范围获取name对应的有序集合的元素`
 
`# 参数：`
`    ``# name，redis的name`
`    ``# start，有序集合索引起始位置（非分数）`
`    ``# end，有序集合索引结束位置（非分数）`
`    ``# desc，排序规则，默认按照分数从小到大排序`
`    ``# withscores，是否获取元素的分数，默认只获取元素的值`
`    ``# score_cast_func，对分数进行数据转换的函数`
 
`# 更多：`
`    ``# 从大到小排序`
`    ``# zrevrange(name, start, end, withscores=False, score_cast_func=float)`
 
`    ``# 按照分数范围获取name对应的有序集合的元素`
`    ``# zrangebyscore(name, min, max, start=None, num=None, withscores=False, score_cast_func=float)`
`    ``# 从大到小排序`
`    ``# zrevrangebyscore(name, max, min, start=None, num=None, withscores=False, score_cast_func=float)`



</td>

</tr>

</tbody>

</table>







zrank(name, value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4

</td>
<td class="code">

`# 获取某个值在 name对应的有序集合中的排行（从 0 开始）`
 
`# 更多：`
`    ``# zrevrank(name, value)，从大到小排序`



</td>

</tr>

</tbody>

</table>







zrangebylex(name, min, max, start=None, num=None)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17

</td>
<td class="code">

`# 当有序集合的所有成员都具有相同的分值时，有序集合的元素会根据成员的 值 （lexicographical ordering）来进行排序，而这个命令则可以返回给定的有序集合键 key 中， 元素的值介于 min 和 max 之间的成员`
`# 对集合中的每个成员进行逐个字节的对比（byte-by-byte compare）， 并按照从低到高的顺序， 返回排序后的集合成员。 如果两个字符串有一部分内容是相同的话， 那么命令会认为较长的字符串比较短的字符串要大`
 
`# 参数：`
`    ``# name，redis的name`
`    ``# min，左区间（值）。 + 表示正无限； - 表示负无限； ( 表示开区间； [ 则表示闭区间`
`    ``# min，右区间（值）`
`    ``# start，对结果进行分片处理，索引位置`
`    ``# num，对结果进行分片处理，索引后面的num个元素`
 
`# 如：`
`    ``# ZADD myzset 0 aa 0 ba 0 ca 0 da 0 ea 0 fa 0 ga`
`    ``# r.zrangebylex('myzset', "-", "[ca") 结果为：['aa', 'ba', 'ca']`
 
`# 更多：`
`    ``# 从大到小排序`
`    ``# zrevrangebylex(name, max, min, start=None, num=None)`



</td>

</tr>

</tbody>

</table>







zrem(name, values)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3

</td>
<td class="code">

`# 删除name对应的有序集合中值是values的成员`
 
`# 如：zrem('zz', ['s1', 's2'])`



</td>

</tr>

</tbody>

</table>







zremrangebyrank(name, min, max)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 根据排行范围删除`



</td>

</tr>

</tbody>

</table>







zremrangebyscore(name, min, max)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 根据分数范围删除`



</td>

</tr>

</tbody>

</table>







zremrangebylex(name, min, max)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 根据值返回删除`



</td>

</tr>

</tbody>

</table>







zscore(name, value)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 获取name对应有序集合中 value 对应的分数`



</td>

</tr>

</tbody>

</table>







zinterstore(dest, keys, aggregate=None)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2

</td>
<td class="code">

`# 获取两个有序集合的交集，如果遇到相同值不同分数，则按照aggregate进行操作`
`# aggregate的值为:  SUM  MIN  MAX`



</td>

</tr>

</tbody>

</table>







zunionstore(dest, keys, aggregate=None)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2

</td>
<td class="code">

`# 获取两个有序集合的并集，如果遇到相同值不同分数，则按照aggregate进行操作`
`# aggregate的值为:  SUM  MIN  MAX`



</td>

</tr>

</tbody>

</table>







zscan(name, cursor=0, match=None, count=None, score_cast_func=float)zscan_iter(name, match=None, count=None,score_cast_func=float)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 同字符串相似，相较于字符串新增score_cast_func，用来对分数进行操作`



</td>

</tr>

</tbody>

</table>







　　




其他常用操作

> 
delete(*names)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 根据删除redis中的任意数据类型`



</td>

</tr>

</tbody>

</table>







exists(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 检测redis的name是否存在`



</td>

</tr>

</tbody>

</table>







keys(pattern='*')



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1
2
3
4
5
6
7

</td>
<td class="code">

`# 根据模型获取redis的name`
 
`# 更多：`
`    ``# KEYS * 匹配数据库中所有 key 。`
`    ``# KEYS h?llo 匹配 hello ， hallo 和 hxllo 等。`
`    ``# KEYS h*llo 匹配 hllo 和 heeeeello 等。`
`    ``# KEYS h[ae]llo 匹配 hello 和 hallo ，但不匹配 hillo `



</td>

</tr>

</tbody>

</table>







expire(name ,time)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 为某个redis的某个name设置超时时间`



</td>

</tr>

</tbody>

</table>







rename(src, dst)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 对redis的name重命名为`



</td>

</tr>

</tbody>

</table>







move(name, db))



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 将redis的某个值移动到指定的db下`



</td>

</tr>

</tbody>

</table>







randomkey()



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 随机获取一个redis的name（不删除）`



</td>

</tr>

</tbody>

</table>







type(name)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 获取name对应值的类型`



</td>

</tr>

</tbody>

</table>







scan(cursor=0, match=None, count=None)scan_iter(match=None, count=None)



[?](#)
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
1

</td>
<td class="code">

`# 同字符串操作，用于增量迭代获取key`



</td>

</tr>

</tbody>

</table>







　




4、管道

redis-py默认在执行每次请求都会创建（连接池申请连接）和断开（归还连接池）一次连接操作，如果想要在一次请求中指定多个命令，则可以使用pipline实现一次请求指定多个命令，并且默认情况下一次pipline 是原子性操作。
<td class="gutter">12345678910111213141516</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-` `import` `redis` `pool ``=` `redis.ConnectionPool(host``=``'10.211.55.4'``, port``=``6379``)` `r ``=` `redis.Redis(connection_pool``=``pool)` `# pipe = r.pipeline(transaction=False)``pipe ``=` `r.pipeline(transaction``=``True``)``pipe.multi()``pipe.``set``(``'name'``, ``'alex'``)``pipe.``set``(``'role'``, ``'sb'``)` `pipe.execute()`</td>

{% raw %}
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import redis

conn = redis.Redis(host='192.168.1.41',port=6379)

conn.set('count',1000)

with conn.pipeline() as pipe:

    # 先监视，自己的值没有被修改过
    conn.watch('count')

    # 事务开始
    pipe.multi()
    old_count = conn.get('count')
    count = int(old_count)
    if count > 0:  # 有库存
        pipe.set('count', count - 1)

    # 执行，把所有命令一次性推送过去
    pipe.execute()
```
{% endraw %}

5、发布订阅

<img src="https://images2015.cnblogs.com/blog/425762/201601/425762-20160121152411125-1838441844.png" alt="" width="467" height="273" />

发布者：服务器

订阅者：Dashboad和数据处理

Demo如下：

{% raw %}
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import redis


class RedisHelper:

    def __init__(self):
        self.__conn = redis.Redis(host='10.211.55.4')
        self.chan_sub = 'fm104.5'
        self.chan_pub = 'fm104.5'

    def public(self, msg):
        self.__conn.publish(self.chan_pub, msg)
        return True

    def subscribe(self):
        pub = self.__conn.pubsub()
        pub.subscribe(self.chan_sub)
        pub.parse_response()
        return pub
```
{% endraw %}

订阅者：
<td class="gutter">1234567891011</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-` `from` `monitor.RedisHelper ``import` `RedisHelper` `obj ``=` `RedisHelper()``redis_sub ``=` `obj.subscribe()` `while` `True``:``    ``msg``=` `redis_sub.parse_response()``    ``print` `msg`</td>

发布者：
<td class="gutter">1234567</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-` `from` `monitor.RedisHelper ``import` `RedisHelper` `obj ``=` `RedisHelper()``obj.public(``'hello'``)`</td>

6. sentinel

redis重的sentinel主要用于在redis主从复制中，如果master顾上，则自动将slave替换成master
<td class="gutter">123456789101112131415161718192021222324252627282930</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-` `from` `redis.sentinel ``import` `Sentinel` `# 连接哨兵服务器(主机名也可以用域名)``sentinel ``=` `Sentinel([(``'10.211.55.20'``, ``26379``),``                     ``(``'10.211.55.20'``, ``26380``),``                     ``],``                    ``socket_timeout``=``0.5``)` `# # 获取主服务器地址``# master = sentinel.discover_master('mymaster')``# print(master)``#``# # # 获取从服务器地址``# slave = sentinel.discover_slaves('mymaster')``# print(slave)``#``#``# # # 获取主服务器进行写入``# master = sentinel.master_for('mymaster')``# master.set('foo', 'bar')`   `# # # # 获取从服务器进行读取（默认是round-roubin）``# slave = sentinel.slave_for('mymaster', password='redis_auth_pass')``# r_ret = slave.get('foo')``# print(r_ret)`</td>

　　

更多参见：https://github.com/andymccurdy/redis-py/

http://doc.redisfans.com/

### RabbitMQ

RabbitMQ是一个在AMQP基础上完整的，可复用的企业消息系统。他遵循Mozilla Public License开源协议。

MQ全称为Message Queue, [消息队列](http://baike.baidu.com/view/262473.htm)（MQ）是一种应用程序对应用程序的通信方法。应用程序通过读写出入队列的消息（针对应用程序的数据）来通信，而无需专用连接来链接它们。消 息传递指的是程序之间通过在消息中发送数据进行通信，而不是通过直接调用彼此来通信，直接调用通常是用于诸如[远程过程调用](http://baike.baidu.com/view/431455.htm)的技术。排队指的是应用程序通过 队列来通信。队列的使用除去了接收和发送应用程序同时执行的要求。

RabbitMQ安装
<td class="gutter">12345678</td><td class="code">`安装配置epel源``   ``$ rpm ``-``ivh http:``/``/``dl.fedoraproject.org``/``pub``/``epel``/``6``/``i386``/``epel``-``release``-``6``-``8.noarch``.rpm` `安装erlang``   ``$ yum ``-``y install erlang` `安装RabbitMQ``   ``$ yum ``-``y install rabbitmq``-``server`</td>

注意：service rabbitmq-server start/stop

安装API
<td class="gutter">1234567</td><td class="code">`pip install pika``or``easy_install pika``or``源码` `https:``/``/``pypi.python.org``/``pypi``/``pika`</td>

使用API操作RabbitMQ

基于Queue实现生产者消费者模型

{% raw %}
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import Queue
import threading


message = Queue.Queue(10)


def producer(i):
    while True:
        message.put(i)


def consumer(i):
    while True:
        msg = message.get()


for i in range(12):
    t = threading.Thread(target=producer, args=(i,))
    t.start()

for i in range(10):
    t = threading.Thread(target=consumer, args=(i,))
    t.start()
```
{% endraw %}

对于RabbitMQ来说，生产和消费不再针对内存里的一个Queue对象，而是某台服务器上的RabbitMQ Server实现的消息队列。
<td class="gutter">12345678910111213141516</td><td class="code">`#!/usr/bin/env python``import` `pika` `# ######################### 生产者 #########################` `connection ``=` `pika.BlockingConnection(pika.ConnectionParameters(``        ``host``=``'localhost'``))``channel ``=` `connection.channel()` `channel.queue_declare(queue``=``'hello'``)` `channel.basic_publish(exchange``=``'',``                      ``routing_key``=``'hello'``,``                      ``body``=``'Hello World!'``)``print``(``" [x] Sent 'Hello World!'"``)``connection.close()`</td>
<td class="gutter">1234567891011121314151617181920</td><td class="code">`#!/usr/bin/env python``import` `pika` `# ########################## 消费者 ##########################` `connection ``=` `pika.BlockingConnection(pika.ConnectionParameters(``        ``host``=``'localhost'``))``channel ``=` `connection.channel()` `channel.queue_declare(queue``=``'hello'``)` `def` `callback(ch, method, properties, body):``    ``print``(``" [x] Received %r"` `%` `body)` `channel.basic_consume(callback,``                      ``queue``=``'hello'``,``                      ``no_ack``=``True``)` `print``(``' [*] Waiting for messages. To exit press CTRL+C'``)``channel.start_consuming()`</td>

1、acknowledgment 消息不丢失

{% raw %}
```
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(
        host='10.211.55.4'))
channel = connection.channel()

channel.queue_declare(queue='hello')

def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)
    import time
    time.sleep(10)
    print 'ok'
    ch.basic_ack(delivery_tag = method.delivery_tag)

channel.basic_consume(callback,
                      queue='hello',
                      no_ack=False)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```
{% endraw %}

2、durable   消息不丢失

{% raw %}
```
#!/usr/bin/env python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(host='10.211.55.4'))
channel = connection.channel()

# make message persistent
channel.queue_declare(queue='hello', durable=True)

channel.basic_publish(exchange='',
                      routing_key='hello',
                      body='Hello World!',
                      properties=pika.BasicProperties(
                          delivery_mode=2, # make message persistent
                      ))
print(" [x] Sent 'Hello World!'")
connection.close()
```
{% endraw %}

{% raw %}
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(host='10.211.55.4'))
channel = connection.channel()

# make message persistent
channel.queue_declare(queue='hello', durable=True)


def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)
    import time
    time.sleep(10)
    print 'ok'
    ch.basic_ack(delivery_tag = method.delivery_tag)

channel.basic_consume(callback,
                      queue='hello',
                      no_ack=False)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```
{% endraw %}

3、消息获取顺序

默认消息队列里的数据是按照顺序被消费者拿走，例如：消费者1 去队列中获取 奇数 序列的任务，消费者1去队列中获取 偶数 序列的任务。

channel.basic_qos(prefetch_count=1) 表示谁来谁取，不再按照奇偶数排列

{% raw %}
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(host='10.211.55.4'))
channel = connection.channel()

# make message persistent
channel.queue_declare(queue='hello')


def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)
    import time
    time.sleep(10)
    print 'ok'
    ch.basic_ack(delivery_tag = method.delivery_tag)

channel.basic_qos(prefetch_count=1)

channel.basic_consume(callback,
                      queue='hello',
                      no_ack=False)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```
{% endraw %}

4、发布订阅

<img src="https://images2015.cnblogs.com/blog/425762/201607/425762-20160717140730998-2143093474.png" alt="" width="634" height="189" />

发布订阅和简单的消息队列区别在于，发布订阅会将消息发送给所有的订阅者，而消息队列中的数据被消费一次便消失。所以，RabbitMQ实现发布和订阅时，会为每一个订阅者创建一个队列，而发布者发布消息时，会将消息放置在所有相关队列中。

{% raw %}
```
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(pika.ConnectionParameters(
        host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='logs',
                         type='fanout')

message = ' '.join(sys.argv[1:]) or "info: Hello World!"
channel.basic_publish(exchange='logs',
                      routing_key='',
                      body=message)
print(" [x] Sent %r" % message)
connection.close()
```
{% endraw %}

{% raw %}
```
#!/usr/bin/env python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(
        host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='logs',
                         type='fanout')

result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue

channel.queue_bind(exchange='logs',
                   queue=queue_name)

print(' [*] Waiting for logs. To exit press CTRL+C')

def callback(ch, method, properties, body):
    print(" [x] %r" % body)

channel.basic_consume(callback,
                      queue=queue_name,
                      no_ack=True)

channel.start_consuming()
```
{% endraw %}

5、关键字发送

<img src="https://images2015.cnblogs.com/blog/425762/201607/425762-20160717140748795-1181706200.png" alt="" width="585" height="191" />

之前事例，发送消息时明确指定某个队列并向其中发送消息，RabbitMQ还支持根据关键字发送，即：队列绑定关键字，发送者将数据根据关键字发送到消息exchange，exchange根据 关键字 判定应该将数据发送至指定队列。

{% raw %}
```
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(pika.ConnectionParameters(
        host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='direct_logs',
                         type='direct')

result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue

severities = sys.argv[1:]
if not severities:
    sys.stderr.write("Usage: %s [info] [warning] [error]\n" % sys.argv[0])
    sys.exit(1)

for severity in severities:
    channel.queue_bind(exchange='direct_logs',
                       queue=queue_name,
                       routing_key=severity)

print(' [*] Waiting for logs. To exit press CTRL+C')

def callback(ch, method, properties, body):
    print(" [x] %r:%r" % (method.routing_key, body))

channel.basic_consume(callback,
                      queue=queue_name,
                      no_ack=True)

channel.start_consuming()
```
{% endraw %}

{% raw %}
```
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(pika.ConnectionParameters(
        host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='direct_logs',
                         type='direct')

severity = sys.argv[1] if len(sys.argv) > 1 else 'info'
message = ' '.join(sys.argv[2:]) or 'Hello World!'
channel.basic_publish(exchange='direct_logs',
                      routing_key=severity,
                      body=message)
print(" [x] Sent %r:%r" % (severity, message))
connection.close()
```
{% endraw %}

6、模糊匹配

<img src="https://images2015.cnblogs.com/blog/425762/201607/425762-20160717140807232-1395723247.png" alt="" width="544" height="181" />

在topic类型下，可以让队列绑定几个模糊的关键字，之后发送者将数据发送到exchange，exchange将传入路由值和 关键字进行匹配，匹配成功，则将数据发送到指定队列。

- # 表示可以匹配 0 个 或 多个 单词
- *  表示只能匹配 一个 单词
<td class="gutter">123</td><td class="code">`发送者路由值              队列中``old.boy.python          old.``*`  `-``-` `不匹配``old.boy.python          old.``#  -- 匹配`</td>

{% raw %}
```
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(pika.ConnectionParameters(
        host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='topic_logs',
                         type='topic')

result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue

binding_keys = sys.argv[1:]
if not binding_keys:
    sys.stderr.write("Usage: %s [binding_key]...\n" % sys.argv[0])
    sys.exit(1)

for binding_key in binding_keys:
    channel.queue_bind(exchange='topic_logs',
                       queue=queue_name,
                       routing_key=binding_key)

print(' [*] Waiting for logs. To exit press CTRL+C')

def callback(ch, method, properties, body):
    print(" [x] %r:%r" % (method.routing_key, body))

channel.basic_consume(callback,
                      queue=queue_name,
                      no_ack=True)

channel.start_consuming()
```
{% endraw %}

{% raw %}
```
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(pika.ConnectionParameters(
        host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='topic_logs',
                         type='topic')

routing_key = sys.argv[1] if len(sys.argv) > 1 else 'anonymous.info'
message = ' '.join(sys.argv[2:]) or 'Hello World!'
channel.basic_publish(exchange='topic_logs',
                      routing_key=routing_key,
                      body=message)
print(" [x] Sent %r:%r" % (routing_key, message))
connection.close()
```
{% endraw %}

注意：

{% raw %}
```
sudo rabbitmqctl add_user wupeiqi 123
# 设置用户为administrator角色
sudo rabbitmqctl set_user_tags wupeiqi administrator
# 设置权限
sudo rabbitmqctl set_permissions -p "/" root ".*" ".*" ".*"

# 然后重启rabbiMQ服务
sudo /etc/init.d/rabbitmq-server restart
 
# 然后可以使用刚才的用户远程连接rabbitmq server了。


------------------------------
credentials = pika.PlainCredentials("wupeiqi","123")

connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.14.47',credentials=credentials))
```
{% endraw %}

{% raw %}
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import pika
from pika.adapters.blocking_connection import BlockingChannel

credentials = pika.PlainCredentials("root", "123")

conn = pika.BlockingConnection(pika.ConnectionParameters(host='10.211.55.20', credentials=credentials))
# 超时时间
conn.add_timeout(5, lambda: channel.stop_consuming())

channel = conn.channel()

channel.queue_declare(queue='hello')


def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)
    channel.stop_consuming()


channel.basic_consume(callback,
                      queue='hello',
                      no_ack=True)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```
{% endraw %}

### SQLAlchemy

<img src="https://images2015.cnblogs.com/blog/425762/201601/425762-20160117042127803-263417768.png" alt="" width="539" height="377" />

Dialect用于和数据API进行交流，根据配置文件的不同调用不同的数据库API，从而实现对数据库的操作，如：
<td class="gutter">12345678910111213</td><td class="code">`MySQL``-``Python``    ``mysql``+``mysqldb:``/``/``<user>:<password>@<host>[:<port>]``/``<dbname>` `pymysql``    ``mysql``+``pymysql:``/``/``<username>:<password>@<host>``/``<dbname>[?<options>]` `MySQL``-``Connector``    ``mysql``+``mysqlconnector:``/``/``<user>:<password>@<host>[:<port>]``/``<dbname>` `cx_Oracle``    ``oracle``+``cx_oracle:``/``/``user:``pass``@host:port``/``dbname[?key``=``value&key``=``value...]` `更多详见：http:``/``/``docs.sqlalchemy.org``/``en``/``latest``/``dialects``/``index.html`</td>

步骤一：

使用 Engine/ConnectionPooling/Dialect 进行数据库操作，Engine使用ConnectionPooling连接数据库，然后再通过Dialect执行SQL语句。
<td class="gutter">1234567891011121314151617181920212223</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-` `from` `sqlalchemy ``import` `create_engine`  `engine ``=` `create_engine(``"mysql+mysqldb://root:123@127.0.0.1:3306/s11"``, max_overflow``=``5``)` `engine.execute(``    ``"INSERT INTO ts_test (a, b) VALUES ('2', 'v1')"``)` `engine.execute(``     ``"INSERT INTO ts_test (a, b) VALUES (%s, %s)"``,``    ``((``555``, ``"v1"``),(``666``, ``"v1"``),)``)``engine.execute(``    ``"INSERT INTO ts_test (a, b) VALUES (%(id)s, %(name)s)"``,``    ``id``=``999``, name``=``"v1"``)` `result ``=` `engine.execute(``'select * from ts_test'``)``result.fetchall()`</td>

{% raw %}
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

from sqlalchemy import create_engine


engine = create_engine("mysql+mysqldb://root:123@127.0.0.1:3306/s11", max_overflow=5)


# 事务操作
with engine.begin() as conn:
    conn.execute("insert into table (x, y, z) values (1, 2, 3)")
    conn.execute("my_special_procedure(5)")
    
    
conn = engine.connect()
# 事务操作 
with conn.begin():
       conn.execute("some statement", {'x':5, 'y':10})
```
{% endraw %}

**注：查看数据库连接：show status like 'Threads%';**

步骤二：

使用 Schema Type/SQL Expression Language/Engine/ConnectionPooling/Dialect 进行数据库操作。Engine使用Schema Type创建一个特定的结构对象，之后通过SQL Expression Language将该对象转换成SQL语句，然后通过 ConnectionPooling 连接数据库，再然后通过 Dialect 执行SQL，并获取结果。
<td class="gutter">123456789101112131415161718192021</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-` `from` `sqlalchemy ``import` `create_engine, Table, Column, Integer, String, MetaData, ForeignKey` `metadata ``=` `MetaData()` `user ``=` `Table(``'user'``, metadata,``    ``Column(``'id'``, Integer, primary_key``=``True``),``    ``Column(``'name'``, String(``20``)),``)` `color ``=` `Table(``'color'``, metadata,``    ``Column(``'id'``, Integer, primary_key``=``True``),``    ``Column(``'name'``, String(``20``)),``)``engine ``=` `create_engine(``"mysql+mysqldb://root:123@127.0.0.1:3306/s11"``, max_overflow``=``5``)` `metadata.create_all(engine)``# metadata.clear()``# metadata.remove()`</td>

{% raw %}
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

from sqlalchemy import create_engine, Table, Column, Integer, String, MetaData, ForeignKey

metadata = MetaData()

user = Table('user', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(20)),
)

color = Table('color', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(20)),
)
engine = create_engine("mysql+mysqldb://root:123@127.0.0.1:3306/s11", max_overflow=5)

conn = engine.connect()

# 创建SQL语句，INSERT INTO "user" (id, name) VALUES (:id, :name)
conn.execute(user.insert(),{'id':7,'name':'seven'})
conn.close()

# sql = user.insert().values(id=123, name='wu')
# conn.execute(sql)
# conn.close()

# sql = user.delete().where(user.c.id > 1)

# sql = user.update().values(fullname=user.c.name)
# sql = user.update().where(user.c.name == 'jack').values(name='ed')

# sql = select([user, ])
# sql = select([user.c.id, ])
# sql = select([user.c.name, color.c.name]).where(user.c.id==color.c.id)
# sql = select([user.c.name]).order_by(user.c.name)
# sql = select([user]).group_by(user.c.name)

# result = conn.execute(sql)
# print result.fetchall()
# conn.close()
```
{% endraw %}

更多内容详见：

注：SQLAlchemy无法修改表结构，如果需要可以使用SQLAlchemy开发者开源的另外一个软件Alembic来完成。

步骤三：

使用 ORM/Schema Type/SQL Expression Language/Engine/ConnectionPooling/Dialect 所有组件对数据进行操作。根据类创建对象，对象转换成SQL，执行SQL。
<td class="gutter">1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859</td><td class="code">`#!/usr/bin/env python``# -*- coding:utf-8 -*-` `from` `sqlalchemy.ext.declarative ``import` `declarative_base``from` `sqlalchemy ``import` `Column, Integer, String``from` `sqlalchemy.orm ``import` `sessionmaker``from` `sqlalchemy ``import` `create_engine` `engine ``=` `create_engine(``"mysql+mysqldb://root:123@127.0.0.1:3306/s11"``, max_overflow``=``5``)` `Base ``=` `declarative_base()`  `class` `User(Base):``    ``__tablename__ ``=` `'users'``    ``id` `=` `Column(Integer, primary_key``=``True``)``    ``name ``=` `Column(String(``50``))` `# 寻找Base的所有子类，按照子类的结构在数据库中生成对应的数据表信息``# Base.metadata.create_all(engine)` `Session ``=` `sessionmaker(bind``=``engine)``session ``=` `Session()`  `# ########## 增 ##########``# u = User(id=2, name='sb')``# session.add(u)``# session.add_all([``#     User(id=3, name='sb'),``#     User(id=4, name='sb')``# ])``# session.commit()` `# ########## 删除 ##########``# session.query(User).filter(User.id > 2).delete()``# session.commit()` `# ########## 修改 ##########``# session.query(User).filter(User.id > 2).update({'cluster_id' : 0})``# session.commit()``# ########## 查 ##########``# ret = session.query(User).filter_by(name='sb').first()` `# ret = session.query(User).filter_by(name='sb').all()``# print ret` `# ret = session.query(User).filter(User.name.in_(['sb','bb'])).all()``# print ret` `# ret = session.query(User.name.label('name_label')).all()``# print ret,type(ret)` `# ret = session.query(User).order_by(User.id).all()``# print ret` `# ret = session.query(User).order_by(User.id)[1:3]``# print ret``# session.commit()`</td>

更多功能参见文档，[猛击这里](http://files.cnblogs.com/files/wupeiqi/sqlalchemy.pdf.zip)下载PDF
