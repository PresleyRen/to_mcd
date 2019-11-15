---
layout:     post
title:      Flask中的蓝图(BluePrint)
subtitle:   
date:       2019-03-17
author:     P
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - python
---
蓝图,听起来就是一个很宏伟的东西

在Flask中的蓝图 blueprint 也是非常宏伟的

它的作用就是将 功能 与 主服务 分开怎么理解呢?

比如说,你有一个客户管理系统,最开始的时候,只有一个查看客户列表的功能,后来你又加入了一个添加客户的功能(add_user)模块, 然后又加入了一个删除客户的功能(del_user)模块,然后又加入了一个修改客户的功能(up_user)模块,在这个系统中,就可以将

查看客户,修改客户,添加客户,删除客户的四个功能做成蓝图加入到客户管理系统中,本篇最后会做一个这样的例子,但是首先我们要搞清楚什么是蓝图 blueprint

1.初识Flask蓝图(blueprint)

创建一个项目然后将目录结构做成:

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705135617382-1292374835.png" alt="" />

{% raw %}
```
from flask import Blueprint  # 导入 Flask 中的蓝图 Blueprint 模块

sv = Blueprint("sv", __name__)  # 实例化一个蓝图(Blueprint)对象


@sv.route("/svlist")  # 这里添加路由和视图函数的时候与在Flask对象中添加是一样的
def view_list():
    return "svlist_view_list"
```
{% endraw %}

manager.py 文件中的内容

{% raw %}
```
from flask import Flask

# 导入此前写好的蓝图模块
from student_view import s_view

app = Flask(__name__)  # type:Flask

# 在Flask对象中注册蓝图模块中的蓝图对象 s_view 中的 sv
app.register_blueprint(s_view.sv)

app.run("0.0.0.0",5000)
# 现在Flask对象中并没有写任何的路由和视图函数
```
{% endraw %}

开启服务,然后访问 http://127.0.0.1:5000/svlist 查看结果

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705135755354-470496391.png" alt="" />

很明显,我们没有在Flask对象中添加路由,但是我们注册了有路由和视图函数的sv蓝图对象

2.如何理解蓝图呢?

其实我们可以理解成一个没有run方法的Flask对象,这个理论虽然有很多的漏洞,但是对于刚接触蓝图的你来说,就这么样理解,没有错

下面来看一下,在实例化蓝图的时候可以传递的参数都有什么,你就能完全理解了

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705141156302-345619059.png" alt="" />

这是目录结构

{% raw %}
```
from flask import Blueprint  # 导入 Flask 中的蓝图 Blueprint 模块
from flask import render_template

sv = Blueprint("sv",
               __name__,
               template_folder="sv_template",  # 每个蓝图都可以为自己独立出一套template模板文件夹,如果不写则共享项目目录中的templates
               static_folder="sv_static"  # 静态文件目录也是可以独立出来的
               )  # 实例化一个蓝图(Blueprint)对象


@sv.route("/svlist")
def view_list():
    return render_template("svlist.html")
```
{% endraw %}

{% raw %}
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    Hello ! I am sv_template
    <img src="/sv_static/img.png">
</body>
</html> 
```
{% endraw %}

打开页面看结果

127.0.0.1:5000/svlist

从这个例子中我们总结出:

只要Blueprint被 Flask 注册了,就一定会生效

坑来了!坑来了!

蓝图内部的视图函数及route不要出现重复,否则~你们自己试试吧

3.使用蓝图,做一个增删改查用户

要有一个文件存放我们的原始数据

{% raw %}
```
STUDENT = [
    {'id': 1, 'name': 'Old', 'age': 38, 'gender': '中'},
    {'id': 2, 'name': 'Boy', 'age': 73, 'gender': '男'},
    {'id': 3, 'name': 'EDU', 'age': 84, 'gender': '女'}
]
```
{% endraw %}

然后我们根据以上内容进行增删改查

3.1 使用蓝图进行web应用搭建:

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705145839336-1943333214.png" alt="" />

__init__.py 文件中的内容:

{% raw %}
```
from flask import Flask


def create_app():
    app = Flask(__name__)

    return app
```
{% endraw %}

这个文件我们会修改函数 create_app中的代码

manager.py 文件中的内容

{% raw %}
```
from student import create_app

flask_app = create_app()

flask_app.run("0.0.0.0",5000)
```
{% endraw %}

通过这种方式启动 Flask 程序

3.2 使用Flask蓝图,查看学生信息

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705153716822-1307854308.png" alt="" />

{% raw %}
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>学生列表</title>
</head>
<body>
<table border="3xp">
    <thead>
    <tr>
        <td>ID</td>
        <td>name</td>
        <td>age</td>
        <td>gender</td>
        <td>options</td>
    </tr>
    </thead>
    <tbody>
    {% for foo in student %}
        <tr>
            <td>{{ foo.id }}</td>
            <td>{{ foo["name"] }}</td>
            <td>{{ foo.get("age") }}</td>
            <td>{{ foo.gender }}</td>
            <td> <a href="/s_update/{{ foo.id }}">修改</a> | <a href="/s_del?id={{ foo.id }}">删除</a> </td>
        </tr>
    {% endfor %}
    </tbody>
</table>
<a href="/s_add"> 添加学生 </a>
</body>
</html>
```
{% endraw %}

{% raw %}
```
from flask import Blueprint
from flask import render_template
from student_data import STUDENT

ss_blueprint = Blueprint("ss_b", __name__, template_folder="html", static_folder="static")


@ss_blueprint.route("/s_list")
def s_list():
    return render_template("s_list.html", student=STUDENT)
```
{% endraw %}

{% raw %}
```
from flask import Flask
from student_select import stu_select


def create_app():
    app = Flask(__name__)  # type:Flask

    app.register_blueprint(stu_select.ss_blueprint)

    return app
```
{% endraw %}

赶紧运行一下manager.py 来访问一下,我们的成果

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705154232051-1657616413.png" alt="" />

什么链接都不要点,因为点啥都不好使,之后咱们一个一个的做

3.3. 使用Flask蓝图,添加一个学生

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705154342878-514832894.png" alt="" />

{% raw %}
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>学生列表</title>
</head>
<body>
<form method="post">
    ID:<input type="text" name="id"> <br>
    姓名:<input type="text" name="name"><br>
    年龄:<input type="text" name="age"><br>
    性别:<input type="text" name="gender"><br>
    <input type="submit" value="添加学生">
</form>

</body>
</html>
```
{% endraw %}

{% raw %}
```
from flask import Blueprint
from flask import redirect
from flask import request
from flask import render_template
from student_data import STUDENT

s_add = Blueprint("s_add", __name__, template_folder="html", static_folder="static") # type:Blueprint


@s_add.route("/s_add",methods=["GET","POST"])
def s_add_view():
    if request.method == "POST":
        stu_dic = {
            "id": request.form["id"],
            "name": request.form["name"],
            "age": request.form["age"],
            "gender": request.form["gender"]
        }

        STUDENT.append(stu_dic)

        return redirect("/s_list")

    return render_template("s_add.html")
```
{% endraw %}

这里面我们让他添加完一个学生,就返回到s_list查看学生列表

{% raw %}
```
from flask import Flask
from student_select import stu_select
from student_add import stu_add


def create_app():
    app = Flask(__name__)  # type:Flask

    app.register_blueprint(stu_select.ss_blueprint)
    app.register_blueprint(stu_add.s_add)

    return app
```
{% endraw %}

如果你要是重新启动服务了,那么你刚刚添加的学生信息就没有了

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705154855593-53199194.png" alt="" />

添加完成之后

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705154914562-2105987817.png" alt="" />

添加学生的Blueprint已经做完了

3.4. 使用Flask蓝图,修改学生信息

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705161441223-146949071.png" alt="" />

{% raw %}
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>学生列表</title>
</head>
<body>
<form method="post">
    <input type="text" name="id" hidden value="{{ student.id }}"><br>
    姓名:<input type="text" name="name" value="{{ student.name }}"><br>
    年龄:<input type="text" name="age" value="{{ student.age }}"><br>
    性别:<input type="text" name="gender" value="{{ student.gender }}"><br>
    <input type="submit" value="修改信息">
</form>

</body>
</html>
```
{% endraw %}

{% raw %}
```
from flask import Blueprint
from flask import render_template
from flask import redirect
from flask import request
from student_data import STUDENT

s_update = Blueprint("s_update", __name__, template_folder="html", static_folder="static")


@s_update.route("/s_update/<int:nid>",methods=["GET","POST"])
def s_update_view(nid):
    if request.method == "POST":
        stu_id = int(request.form["id"])
        stu_dic = {
            "id": stu_id,
            "name": request.form["name"],
            "age": request.form["age"],
            "gender": request.form["gender"]
        }

        for index,stu in enumerate(STUDENT):
            if stu["id"] == stu_id:
                STUDENT[index] = stu_dic

        return redirect("/s_list")

    for stu in STUDENT:
        if stu["id"] == nid :
            return render_template("s_update.html", student=stu)

    return render_template("s_update.html", student="")
```
{% endraw %}

{% raw %}
```
from flask import Flask
from student_select import stu_select
from student_add import stu_add
from student_update import stu_update


def create_app():
    app = Flask(__name__)  # type:Flask

    app.register_blueprint(stu_select.ss_blueprint)
    app.register_blueprint(stu_add.s_add)
    app.register_blueprint(stu_update.s_update)

    return app
```
{% endraw %}

试一下结果:

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705161908819-653078831.png" alt="" />

<img src="https://images2018.cnblogs.com/blog/1122946/201807/1122946-20180705161853827-1080110915.png" alt="" />

4.蓝图目录:

其实学会了蓝图,我们的Flask项目目录结构也就随之出来了,那么Flask的蓝图目录结构应该是什么样子的呢?

<img src="https://img2018.cnblogs.com/blog/1122946/201902/1122946-20190208201505414-1233630998.png" alt="" />

如图,这就是我们建立好的一个目录结构,一层一层的看一下,首先是app目录,它就是我们的主应用程序目录了在这里面有一个__init__.py这个文件里面的内容如下

{% raw %}
```
 1 from flask import Flask
 2 from .views.auto import auto_bp
 3 from .views.motor import motor_bp
 4 
 5 
 6 def create_app():
 7     my_app = Flask(__name__) # type:Flask
 8 
 9     my_app.register_blueprint(auto_bp)
10     my_app.register_blueprint(motor_bp)
11 
12     return my_app
```
{% endraw %}

由此见得__init__.py就是构建app的一个函数,并且将views中的似乎是蓝图的东西注册进去了

接下来看static目录,这个目录从字面意思就可以理解了,就是我们的static静态文件存放目录了

然后就是templates目录,模板存放目录

views目录,主角终于登场了,这里存放的就是视图函数文件,也就是我们Blueprint,每一个文件就是一个Blueprint

{% raw %}
```
1 from flask import Blueprint
2 
3 auto_bp = Blueprint("auto",__name__)
4 
5 @auto_bp.route("/auto")
6 def auto_func():
7     return "my_app.auto"
```
{% endraw %}

{% raw %}
```
1 from flask import Blueprint
2 
3 motor_bp = Blueprint("motor",__name__)
4 
5 @motor_bp.route("/motor")
6 def motor_func():
7     return "my_app.motor"
```
{% endraw %}

这样目录结构就完成了,接下来就是关键性的一个文件manager.py项目的启动文件

{% raw %}
```
1 from app import create_app
2 my_app = create_app()
3 
4 if __name__ == '__main__':
5     my_app.run()
```
{% endraw %}

以上就是我们Flask小型应用的项目结构目录了,要牢记哦

第九篇,完结
