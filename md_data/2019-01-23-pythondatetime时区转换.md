---
layout:     post
title:      pythondatetime时区转换
subtitle:   
date:       2019-01-23
author:     P
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - python
---
From: http://www.dannysite.com/blog/122/
<td class="gutter">1234</td><td class="code">`>>> ``import` `datetime``>>> utc_now ``=` `datetime.datetime.utcnow()``>>> utc_now``datetime.datetime(``2013``, ``12``, ``4``, ``15``, ``43``, ``21``, ``872000``)`</td>

而此时其tzinfo为none：
<td class="gutter">12</td><td class="code">`>>> utc_now.tzinfo``>>>`</td>

当涉及国际时区时，时区转换则会经常使用到，比如来尝试将刚才的UTC时间转换为本地时间。对于Python3.3+版本，可以这么做：
<td class="gutter">12</td><td class="code">`>>> utc_now.replace(tzinfo``=``datetime.timezone.utc).astimezone(tz``=``None``)``datetime.datetime(``2013``, ``12``, ``4``, ``23``, ``43``, ``21``, ``872000``, tzinfo``=``datetime.timezone(datetime.timedelta(``0``, ``28800``), ``'中国标准时间'``))`</td>

不过该方法貌似并不方便，特别是在转换其他时区的时候。而对于更低版本的Python，则datetime.timezone可能压根就还没有。因此更便捷的方法是借助第三方包来实现  pyzt，下面就借助于它来实现时区转换：
<td class="gutter">12345678</td><td class="code">`>>> ``from` `pytz ``import` `timezone``>>> utc_now.tzinfo``>>> tzchina ``=` `timezone(``'Asia/Chongqing'``)``>>> tzchina``<DstTzInfo ``'Asia/Chongqing'` `LMT``+``7``:``06``:``00` `STD>``>>> utc ``=` `timezone(``'UTC'``)``>>> utc_now.replace(tzinfo``=``utc).astimezone(tzchina)``datetime.datetime(``2013``, ``12``, ``4``, ``23``, ``43``, ``21``, ``872000``, tzinfo``=``<DstTzInfo ``'Asia/Chongqing'` `CST``+``8``:``00``:``00` `STD>)`</td>

要转换为其他时区，则以此类推。

对于我自己来说，时区的转换主要出现在Django中，会经常需要将UTC时间转换为本地时间，而Django本身也已经为我们考虑到了这一点，因此实际操作起来更为方便：
<td class="gutter">12345</td><td class="code">`>>> ``from` `django.utils.timezone ``import` `utc``>>> ``from` `django.utils.timezone ``import` `localtime``>>> now ``=` `datetime.datetime.utcnow().replace(tzinfo``=``utc)``>>> localtime(now)``datetime.datetime(``2013``, ``12``, ``5``, ``0``, ``3``, ``13``, ``122000``, tzinfo``=``<DstTzInfo ``'Asia/Shanghai'` `CST``+``8``:``00``:``00` `STD>)`</td>

在Python中转换时区的方法还有很多，通过探索也许还能找到更好的方法。

```
from pytz import timezone

def datetime_as_timezone(date_time, time_zone):
    tz = timezone(time_zone)
    utc = timezone('UTC')
    return date_time.replace(tzinfo=utc).astimezone(tz)


def datetime_to_str(date_time):
    date_time_tzone = datetime_as_timezone(date_time, 'Asia/Shanghai')
    return '{0:%Y-%m-%d %H:%M}'.format(date_time_tzone)
```
