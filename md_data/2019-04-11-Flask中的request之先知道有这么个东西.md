---
layout:     post
title:      Flask中的request之先知道有这么个东西
subtitle:   
date:       2019-04-11
author:     P
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - python
---
每个框架中都有处理请求的机制(request),但是每个框架的处理方式和机制是不同的

为了了解Flask的request中都有什么东西,首先我们要写一个前后端的交互

基于HTML + Flask 写一段前后端的交互

先写一段儿HTML form表单中提交方式是post  action地址是 /req

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180703121332363-1740890545.png" alt="" />

写好一个标准 form 表单,一点提交,搜就向后端提交一个POST请求过去了

后端的接收方式就 666 了

首先要从 flask 包中导入 request 模块 , 至于为什么要导入 request 呢? 这里不做解释,暂时你就知道 request 如果要用,需要导入

解释一个 @app.route("/req",methods=["POST"]) :

methods=["POST"]  代表这个url地址只允许 POST 请求,是个列表也就是意味着可以允许多重请求方式,例如GET之类的

1.request.method 之 肯定知道前端用什么方式提交的

Flask 的 request 中给我们提供了一个 method 属性里面保存的就是前端的请求的方式

{% raw %}
```
print(request.method) # POST 看来可以使用这种方式来验证请求方式了
```
{% endraw %}

2.request.form 之 拿他来举例的话再好不过了

Form表单中传递过来的值 使用 request.form 中拿到

{% raw %}
```
    print(request.form)  # ImmutableMultiDict([('user', 'Oldboy'), ('pwd', 'DragonFire')])
    # ImmutableMultiDict 它看起来像是的Dict 就用Dict的方法取值试一下吧
    print(request.form["user"])  # Oldboy
    print(request.form.get("pwd"))  # DragonFire
    # 看来全部才对了, ImmutableMultiDict 似乎就是个字典,再来玩一玩它
    print(list(request.form.keys()))  # ['user', 'pwd'] 看来是又才对了
    #如果以上所有的方法你都觉得用的不爽的话
    req_dict = dict(request.form)
    print(req_dict)  # 如果你觉得用字典更爽的话,也可以转成字典操作(这里有坑)
```
{% endraw %}

3.request.args 之 你能看见的Url参数全在里面

request.args 中保存的是url中传递的参数

先把后端请求代码改动一下:

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180703153125716-2062288157.png" alt="" />

然后使用URL地址直接传递参数

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180703153204399-716030615.png" alt="" />

然后会在控制台中看到 ImmutableMultiDict([('id', '1'), ('age', '20')])

哎呀我去,这不是和刚才一样吗? 是的!

{% raw %}
```
    print(request.args)  # ImmutableMultiDict([('id', '1'), ('age', '20')])
    print(request.args["id"])  # 1
    print(request.args.get("age"))  # 20
    print(list(request.args.keys()))  # ['id', 'age']
    print(list(request.args.values()))  # ['1', '20']
    req_dict = dict(request.args)  # {'id': ['1'], 'age': ['20']}
    print(req_dict)
```
{% endraw %}

request.args 与 request.form 的区别就是:

request.args 是获取url中的参数

request.form 是获取form表单中的参数

4.request.values 之 只要是个参数我都要

改动一下前端代码:

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180703153941556-951051095.png" alt="" />

这是让我们在使用form表单提交的同时使用url参数提交

{% raw %}
```
print(request.values)  # CombinedMultiDict([ImmutableMultiDict([('id', '1'), ('age', '20')]), ImmutableMultiDict([('user', 'Oldboy'), ('pwd', 'DragonFire')])])
print(request.values.get("id"))  # 1
print(request.values["user"])  # Oldboy
# 这回喜欢直接操作字典的小伙伴们有惊喜了! to_dict() 方法可以直接将我们的参数全部转为字典形式
print(request.values.to_dict()) # {'user': 'Oldboy', 'pwd': 'DragonFire', 'id': '1', 'age': '20'}
```
{% endraw %}

注意啦!注意啦!

{% raw %}
```
# 注意这里的坑来啦! 坑来啦!
# 如果url和form中的Key重名的话,form中的同名的key中value会被url中的value覆盖
# http://127.0.0.1:5000/req?id=1&user=20
print(request.values.to_dict())  # {'user': 20 'pwd': 'DragonFire', 'id': '1'}
```
{% endraw %}

5.request.cookies 之 存在浏览器端的字符串儿也会一起带过来

前提是你要开启浏览器的 cookies

request.cookies 是将cookies中信息读取出来

6.request.headres 之 请求头中的秘密

用来获取本次请求的请求头

{% raw %}
```
    print(type(request.headers))
    """
    Host: 127.0.0.1:5000
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:60.0) Gecko/20100101 Firefox/60.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
    Accept-Encoding: gzip, deflate
    Referer: http://127.0.0.1:5000/home
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 26
    Cookie: csrftoken=vDIozqveCEfArdYXlM6goHVlSQEn7h4bDygNphL2Feas60DiM2di0jlqKfxo7xhA
    Connection: keep-alive
    Upgrade-Insecure-Requests: 1
    Cache-Control: max-age=0
    """
```
{% endraw %}

7.request.data 之 如果处理不了的就变成字符串儿存在data里面

你一定要知道 request 是基于 mimetype 进行处理的

mimetype的类型 以及 字符串儿 : [http://www.w3school.com.cn/media/media_mimeref.asp](http://www.w3school.com.cn/media/media_mimeref.asp)

如果不属于上述类型的描述,request就会将无法处理的参数转为Json存入到 data 中

其实我们可以将 request.data , json.loads 同样可以拿到里面的参数

8.request.files 之 给我一个文件我帮你保管

如果遇到文件上传的话,request.files 里面存的是你上传的文件,但是 Flask 在这个文件的操作中加了一定的封装,让操作变得极为简单

首先改下前端代码:

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180703163119727-2114222908.png" alt="" />

后端这样写

{% raw %}
```
    print(request.files)  # ImmutableMultiDict([('file', <FileStorage: 'DragonFire.txt' ('text/plain')>)])
    print(request.files["file"])  # <FileStorage: 'DragonFire.txt' ('text/plain')>
    my_file = request.files["file"]
    my_file.save("OldBoyEDU.txt")  # 保存文件,里面可以写完整路径+文件名
```
{% endraw %}

这样我们就成功的保存了一个名叫 "OldBoyEDU.txt" 的文件了,操作还是很简单的

9. request.获取各种路径 之 这些方法没必要记,但是要知道它存在

{% raw %}
```
    # 获取当前的url路径
    print(request.path)# /req
    # 当前url路径的上一级路径
    print(request.script_root) #
    # 当前url的全部路径
    print(request.url) # http://127.0.0.1:5000/req
    # 当前url的路径的上一级全部路径
    print(request.url_root ) # http://127.0.0.1:5000/
```
{% endraw %}

10. request.json 之 前提你得告诉是json

如果在请求中写入了 "application/json" 使用 request.json 则返回json解析数据, 否则返回 None
