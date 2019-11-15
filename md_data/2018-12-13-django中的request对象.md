---
layout:     post
title:      django中的request对象
subtitle:   
date:       2018-12-13
author:     P
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - python
---
Request

　　我们知道当URLconf文件匹配到用户输入的路径后，会调用对应的view函数，并将  **HttpRequest对象**  作为第一个参数传入该函数。

　　我们来看一看这个HttpRequest对象有哪些属性或者方法：

**属性：**

1  HttpRequest.scheme 　     请求的协议，一般为http或者https，字符串格式(以下属性中若无特殊指明，均为字符串格式)

2  HttpRequest.body  　　    http请求的主体，二进制格式。

3  HttpRequest.path             所请求页面的完整路径(但不包括协议以及域名)，也就是相对于网站根目录的路径。

4  HttpRequest.path_info     获取具有 URL 扩展名的资源的附加路径信息。相对于HttpRequest.path，使用该方法便于移植。

```
if the WSGIScriptAlias for your application is set to "/minfo", then path might be "/minfo/music/bands/the_beatles/" and path_info would be "/music/bands/the_beatles/".
```

5  HttpRequest.method               获取该请求的方法，比如： GET   POST .........

6  HttpRequest.encoding             获取请求中表单提交数据的编码。

7  HttpRequest.content_type      获取请求的MIME类型(从CONTENT_TYPE头部中获取)，django1.10的新特性。

8  HttpRequest.content_params  获取CONTENT_TYPE中的键值对参数，并以字典的方式表示，django1.10的新特性。

9  HttpRequest.GET                    返回一个 **querydict **对象(类似于字典，本文最后有querydict的介绍)，该对象包含了所有的HTTP GET参数

10  HttpRequest.POST                返回一个 **querydict **，该对象包含了所有的HTTP POST参数，通过表单上传的所有  **字符**  都会保存在该属性中。

11  HttpRequest.COOKIES  　     返回一个包含了所有cookies的**字典**。

12  HttpRequest.FILES  　　       返回一个包含了所有的上传文件的  **querydict**  对象。通过表单所上传的所有  **文件**  都会保存在该属性中。

　　                                             key的值是input标签中name属性的值，value的值是一个UploadedFile对象

13  HttpRequest.META                返回一个包含了所有http头部信息的字典

14  HttpRequest.session       中间件属性

15  HttpRequest.site　　      中间件属性

16  HttpRequest.user　　     中间件属性，表示当前登录的用户。

　　 HttpRequest.user实际上是由一个定义在django.contrib.auth.models 中的  **user model**  类  所创建的对象。

　　 该类有许多字段，属性和方法。列举几个常用的：        获取更详细信息-->[官方文档](https://docs.djangoproject.com/en/1.10/ref/contrib/auth/#django.contrib.auth.models.User.is_authenticated)。

　　　　1  字段： 

　　　　　　**username**    用户名

　　　　　　first_name  

　　　　　　last_name 

　　　　　　email

　　　　　　password   

　　　　　　groups

　　　　　　user_permissions,

　　　　　　is_staff     布尔值，标明用户是否可以访问admin页面

　　　　　　is_superuser 

　　　　　　**last_login**  上一次登陆时间

　　　　　　**date_joined**     用户创建时间

　　　　2  属性  

　　　　　　is_authenticated   布尔值，标志着用户是否已认证。在django1.10之前，没有该属性，但有与该属性同名的方法。

　　　　3  方法

　　　　　　1  HttpRequest.user.get_username()  注意：方法的圆括号在templates标签中必需省略！！

　　　　　　　　　获取username。尽量使用该方法来代替使用username字段

　　　　　　2  HttpRequest.user.get_full_name()  注意：方法的圆括号在templates标签中必需省略！！

　　　　　　　　　获取first_name和last_name

　　　　　　3  HttpRequest.user.short_name()  注意：方法的圆括号在templates标签中必需省略！！

　　　　　　　　　获取first_name

　　　　　　4  HttpRequest.user.set_password(raw_password)  注意：该方法无法在template标签中使用！！

　　　　　　　　　设置密码

　　　　　　5  HttpRequest.user.check_password(raw_password)  注意：该方法无法在template标签中使用！！

　　　　　　　　　如果raw_password与用户密码相等，则返回True````

**方法：**

1  HttpRequest.get_host()            返回请求的源主机。example:  127.0.0.1:8000

2  HttpRequest.get_port()            django1.9的新特性。

3  HttpRequest.get_full_path()     返回完整路径，并包括附加的查询信息。example:  `"/music/bands/the_beatles/?print=true"`

4  HttpRequest.bulid_absolute_uri(location)      返回location的绝对uri，location默认为request.get_full_path()。``

　　  Example: "https://example.com/music/bands/the_beatles/?print=true"

QueryDict 

　　是一个类似于Python中字典的一种对象，他是Python中字典的子类，所以继承了字典的所有方法，

　　当然QueryDict对字典的某些方法进行了加工，并补充了一些独特的方法。这里列出部分方法。详情请看： [官方文档](https://docs.djangoproject.com/en/1.10/ref/request-response/) 。

1  QueryDict.get(key,default=None)   返回key所对应的value，若key不存在，则返回default的值

2  QueryDict.update(other_dict)   更新

3  QueryDict.values()   列出所有的值

4  QueryDict.items()   列出所有的键值对,若一个key有多个值，只显示最后一个值。

5  QueryDict.pop(key)   删除某个键值对

6  QueryDict.getlist(key)   根据输入的key返回一个Python中的list

7  QueryDict.dict()   返回QueryDict的字典的表现形式
