---
layout:     post
title:      Flask中的模板语言Jinja2及render_template的深度用法
subtitle:   
date:       2019-04-11
author:     P
header-img: img/post-bg-mma-0.png
catalog: true
tags:
    - python
---
是时候开始写个前端了,Flask中默认的模板语言是Jinja2

现在我们来一步一步的学习一下 Jinja2 捎带手把 render_template 中留下的疑问解决一下

首先我们要在后端定义几个字符串,用于传递到前端

```
STUDENT = {'name': 'Old', 'age': 38, 'gender': '中'},

STUDENT_LIST = [
    {'name': 'Old', 'age': 38, 'gender': '中'},
    {'name': 'Boy', 'age': 73, 'gender': '男'},
    {'name': 'EDU', 'age': 84, 'gender': '女'}
]

STUDENT_DICT = {
    1: {'name': 'Old', 'age': 38, 'gender': '中'},
    2: {'name': 'Boy', 'age': 73, 'gender': '男'},
    3: {'name': 'EDU', 'age': 84, 'gender': '女'},
}
```

但是前提我们要知道Jinja2模板中的流程控制:

I. Jinja2模板语言中的 for

```
{% for foo in g %}

{% endfor %}
```

II. Jinja2模板语言中的 if

```
{% if g %}

{% elif g %}
    
{% else %}
    
{% endif %}
```

接下来,我们对这几种情况分别进行传递,并在前端显示成表格

1. 使用STUDENT字典传递至前端

后端:

```
@app.route("/student")
def index():
    return render_template("student.html", student=STUDENT)
```

前端:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Old Boy EDU</title>
</head>
<body>
Welcome to Old Boy EDU
{{ student }}
<table border="1px">
    <tr>
        <td>{{ student.name }}</td>
        <td>{{ student["age"] }}</td>
        <td>{{ student.get("gender") }}</td>
    </tr>
</table>
</body>
</html>
```

结果:

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180703171612011-1803396649.png" alt="" />

从这个例子中,可以看出来,字典传入前端Jinja2 模板语言中的取值操作, 与Python中的Dict操作极为相似,并且多了一个student.name的对象操作

2. STUDENT_LIST 列表传入前端Jinja2 模板的操作:

后端:

```
@app.route("/student_list")
def student_list():
    return render_template("student_list.html", student=STUDENT_LIST)
```

前端:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Old Boy EDU</title>
</head>
<body>
Welcome to Old Boy EDU
{{ student }}
<table border="1xp">
    {% for foo in student %}
        <tr>
            <td>{{ foo }}</td>
            <td>{{ foo.name }}</td>
            <td>{{ foo.get("age") }}</td>
            <td>{{ foo["gender"] }}</td>
        </tr>
    {% endfor %}
</table>
</body>
</html>
```

结果:

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180703172051688-1580020640.png" alt="" />

这里我们可以看出如果是需要循环遍历的话,Jinja2 给我们的方案是

```
    {% for foo in student %}
        <tr>
            <td>{{ foo }}</td>
        </tr>
    {% endfor %}
```

上述代码中的 foo 就是列表中的每一个字典,再使用各种取值方式取出值即可

3.STUDENT_DICT 大字典传入前端 Jinja2 模板

后端:

```
@app.route("/student_dict")
def student_dict():
    return render_template("student_dict.html", student=STUDENT_DICT)
```

前端:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Old Boy EDU</title>
</head>
<body>
Welcome to Old Boy EDU
<table>
    {% for foo in student %}
        <tr>
            <td>{{ foo }}</td>
            <td>{{ student.get(foo).name }}</td>
            <td>{{ student[foo].get("age") }}</td>
            <td>{{ student[foo]["gender"] }}</td>
        </tr>
    {% endfor %}
</table>
</body>
</html>
```

在遍历字典的时候,foo 其实是相当于拿出了字典中的Key

结果:

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180703180525638-451491415.png" alt="" />

4.结合所有的字符串儿全部专递前端Jinja2 模板

后端:

```
@app.route("/allstudent")
def all_student():
    return render_template("all_student.html", student=STUDENT ,
                           student_list = STUDENT_LIST,
                           student_dict= STUDENT_DICT)
```

前端:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Old Boy EDU</title>
</head>
<body>
 _____________________________________
Welcome to Old Boy EDU : student
{{ student }}
<table border="1px">
    <tr>
        <td>{{ student.name }}</td>
        <td>{{ student["age"] }}</td>
        <td>{{ student.get("gender") }}</td>
    </tr>
</table>
 _____________________________________
Welcome to Old Boy EDU : student_list
{{ student_list }}
<table border="1xp">
    {% for foo in student_list %}
        <tr>
            <td>{{ foo }}</td>
            <td>{{ foo.name }}</td>
            <td>{{ foo.get("age") }}</td>
            <td>{{ foo["gender"] }}</td>
        </tr>
    {% endfor %}
</table>
 _____________________________________
Welcome to Old Boy EDU : student_dict
{{ student_dict }}
<table border="1xp">
    {% for foo in student_dict %}
        <tr>
            <td>{{ foo }}</td>
            <td>{{ student_dict.get(foo).name }}</td>
            <td>{{ student_dict[foo].get("age") }}</td>
            <td>{{ student_dict[foo]["gender"] }}</td>
        </tr>
    {% endfor %}
</table>
</body>
</html>
```

结果:

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180703181358134-1483976159.png" alt="" />

这里可以看出来,render_template中可以传递多个关键字

5.利用 **{}字典的方式传递参数

前端不变(标题4的前端代码)

后端:

```
@app.route("/allstudent")
def all_student():
    return render_template("all_student.html", **{"student":STUDENT ,
                           "student_list" : STUDENT_LIST,
                           "student_dict": STUDENT_DICT})
```

6. Jinja2 的高阶用法

Jinja2 模板语言为我们提供了很多功能接下来看一下它有什么高级的用法

6.1. safe : 此时你与HTML只差一个 safe

后端代码:

```
from flask import Flask
from flask import render_template

app = Flask(__name__)

@app.route("/")
def index():
    tag = "<input type='text' name='user' value='DragonFire'>"
    return render_template("index.html",tag=tag)


app.run("0.0.0.0",5000)
```

前端代码:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    {{ tag }}
</body>
</html>
```

如果我们直接运行代码直接访问,你会在页面看到什么呢?

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705205259594-1974861561.png" alt="" />

似乎和我们想要结果不太一样,有两种解决方案,

第一种,从前端入手

前端代码:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    {{ tag | safe}}  <!--  加上个 \  管道符,然后 safe  -->
</body>
</html>
```

还有一种方式是从后端入手

后端代码:

```
from flask import Flask
from flask import render_template
from flask import Markup  # 导入 flask 中的 Markup 模块

app = Flask(__name__)


@app.route("/")
def index():
    tag = "<input type='text' name='user' value='DragonFire'>"
    markup_tag = Markup(tag)  # Markup帮助咱们在HTML的标签上做了一层封装,让Jinja2模板语言知道这是一个安全的HTML标签
    print(markup_tag,
          type(markup_tag))  # <input type='text' name='user' value='DragonFire'> <class 'markupsafe.Markup'>
    return render_template("index.html", tag=markup_tag)


app.run("0.0.0.0", 5000, debug=True)
```

两种得到的效果是一样

6.2 在Jinja2中执行Python函数(模板中执行函数)

首先在文件中定义一个函数

后端代码:

```
from flask import Flask
from flask import render_template
from flask import Markup  # 导入 flask 中的 Markup 模块

app = Flask(__name__)

#定义一个函数,把它传递给前端
def a_b_sum(a,b):
    return a+b

@app.route("/")
def index():
    return render_template("index.html", tag=a_b_sum)


app.run("0.0.0.0", 5000, debug=True)
```

前端代码:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    {{ tag }}
    <br>
    {{ tag(99,1) }}
</body>
</html>
```

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705210805791-1956281151.png" alt="" />

看到结果就是,函数加()执行得到结果

还可以定义全局函数,无需后端传递给前端,Jinja2直接就可以执行的函数

后端代码:

```
from flask import Flask
from flask import render_template
from flask import Markup  # 导入 flask 中的 Markup 模块

app = Flask(__name__)


@app.template_global()  # 定义全局模板函数
def a_b_sum(a, b):
    return a + b


@app.template_filter()  # 定义全局模板函数
def a_b_c_sum(a, b, c):
    return a + b + c


@app.route("/")
def index():
    return render_template("index.html", tag="")


app.run("0.0.0.0", 5000, debug=True)
```

前端代码:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    {{ a_b_sum(99,1) }}
    <br>
    {{ 1 | a_b_c_sum(197,2) }}
</body>
</html>
```

两个函数的调用方式不太一样

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705211658411-355417653.png" alt="" />

尤其是@app.template_filter() 它的调用方式比较特别,这是两个Flask中的特殊装饰器

6.3 Jinja2模板复用 block

如果我们前端页面有大量重复页面,没必要每次都写,可以使用模板复用的方式复用模板

前端代码:

index.html 文件中的内容

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>Welcome OldboyEDU</h1>
    <h2>下面的内容是不一样的</h2>
    {% block content %}

    {% endblock %}
    <h2>上面的内容是不一样的,但是下面的内容是一样的</h2>
    <h1>OldboyEDU is Good</h1>
</body>
</html>
```

login.html 文件中的内容

```
{% extends "index.html" %}
{% block content %}
    <form>
        用户名:<input type="text" name="user">
        密码:<input type="text" name="pwd">
    </form>
{% endblock %}
```

home.html 文件中的内容

```
{% extends "index.html" %}
{% block content %}
    <h1>欢迎来到老男孩教育</h1>
{% endblock %}
```

后端代码:

```
from flask import Flask
from flask import render_template

app = Flask(__name__)


@app.route("/login")
def login():
    return render_template("login.html")


@app.route("/home")
def home():
    return render_template("home.html")


app.run("0.0.0.0", 5000, debug=True)
```

然后我们可以看到什么呢?

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705212959069-1120386299.png" alt="" />

大概是这样一个效果

在这两个页面中,只有 block 中的内容发生了变化,其他的位置完全一样

6.4 Jinja2模板语言的模块引用 include

login.html 文件中的内容:

```
<form>
    用户名:<input type="text" name="user">
    密码:<input type="text" name="pwd">
</form>
```

index.html 文件中的内容

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>Welcome OldboyEDU</h1>
    <h2>下面的内容是不一样的</h2>
    {% include "login.html" %}
    <h2>上面的内容是不一样的,但是下面的内容是一样的</h2>
    <h1>OldboyEDU is Good</h1>
</body>
</html>
```

后端代码:

```
from flask import Flask
from flask import render_template

app = Flask(__name__)


@app.route("/")
def index():
    return render_template("index.html")


app.run("0.0.0.0", 5000, debug=True)
```

看到的结果

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705213559471-2133009483.png" alt="" />

这就是将 login.html 当成一个模块,加载到 index.html 页面中

6.5 Jinja2模板语言中的宏定义

前端代码:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<h1>Welcome OldboyEDU</h1>

{% macro type_text(name,type) %}
    <input type="{{ type }}" name="{{ name }}" value="{{ name }}">
{% endmacro %}

<h2>在下方是使用宏来生成input标签</h2>

{{ type_text("one","text") }}
{{ type_text("two","text") }}

</body>
</html>
```

宏定义一般情况下很少应用到,但是要知道有这么个概念

第四篇,完结
