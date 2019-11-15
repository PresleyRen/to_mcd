---
layout:     post
title:      python中的用chr()数值转化为字符串，字符转化为数值ord(s)函数
subtitle:   
date:       2019-08-16
author:     P
header-img: img/post-bg-digital-native.jpg
catalog: true
tags:
    - python
---
### <a name="t0"></a>1.1 python字符串定义

<li>

#!/usr/bin/python

</li>
<li>

# -*- coding: utf8 -*-

</li>
<li>

# 定义一个字符串

</li>
<li>

s1 = 'this is long String that spans two lines'

</li>
<li>

# 表示下面一行是上一行的延续

</li>
<li>

s2 = 'this is long String\

</li>
<li>

that spans two lines'

</li>
<li>

#原样输出字符串

</li>
<li>

s3 = """this is long String

</li>
<li>

 that spans two lines

</li>
<li>

 """

</li>
<li>

# 换行输出字符串

</li>
<li>

s4 = 'this is long String\n\

</li>
<li>

 that spans two lines'

</li>
<li>

print "s1=",s1

</li>
<li>

print "s2=",s2

</li>
<li>

print "s3=",s3

</li>
<li>

print "s4=",s4

</li>

输出结果：

### <a name="t1"></a>1.2 python字符与字符值之间的转换

<li>

#!/usr/bin/python

</li>
<li>

# -*- coding: utf8 -*-

</li>
<li>

# 字符转化为数值

</li>
<li>

s = 'a'

</li>
<li>

print ord(s)

</li>
<li>

# 数值转化为字符串

</li>
<li>

b = 97

</li>
<li>

print chr(b)

</li>

<img class="has" src="https://img-blog.csdnimg.cn/20181208190107285.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob3VsaXFpbmdrZTEyMw==,size_16,color_FFFFFF,t_70" alt="" width="849" height="234" />

C：把一个Unicode码值转化为一个Unicode字符串

<li>

#!/usr/bin/python

</li>
<li>

# -*- coding: utf8 -*-

</li>
<li>

#把unicode码值转化为字符串

</li>
<li>

print repr(unichr(8224))

</li>
<li>

#把unicode字符串转化为码值

</li>
<li>

print repr(ord(u'\u2020'))

</li>

D：chr()函数与str函数的区别：
