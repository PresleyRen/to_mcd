---
layout:     post
title:      Flask-SQLAlchemy
subtitle:   
date:       2019-04-10
author:     P
header-img: img/post-bg-ioses.jpg
catalog: true
tags:
    - python
---
前不久刚刚认识过了SQLAlchemy,**[点击这里复习一下](https://www.cnblogs.com/presleyren/p/10686099.html)**

当 Flask 与 SQLAlchemy 发生火花会怎么样呢?

Flask-SQLAlchemy就这么诞生了

首先要先安装一下Flask-SQLAlchemy这个模块

pip install Flask-SQLAlchemy

然后你要下载一个干净的Flask项目 **[点击下载](https://pan.baidu.com/s/1H26uMUI5gRm4yqkqyAWY1A)**

接下来基于这个Flask项目,我们要加入Flask-SQLAlchemy让项目变得生动起来

### 1.加入Flask-SQLAlchemy第三方组件

{% raw %}
```
 1 from flask import Flask
 2 
 3 # 导入Flask-SQLAlchemy中的SQLAlchemy
 4 from flask_sqlalchemy import SQLAlchemy
 5 
 6 # 实例化SQLAlchemy
 7 db = SQLAlchemy()
 8 # PS : 实例化SQLAlchemy的代码必须要在引入蓝图之前
 9 
10 from .views.users import user
11 
12 
13 def create_app():
14     app = Flask(__name__)
15 
16     # 初始化App配置 这个app配置就厉害了,专门针对 SQLAlchemy 进行配置
17     # SQLALCHEMY_DATABASE_URI 配置 SQLAlchemy 的链接字符串儿
18     app.config["SQLALCHEMY_DATABASE_URI"] = "mysql+pymysql://root:DragonFire@127.0.0.1:3306/dragon?charset=utf8"
19     # SQLALCHEMY_POOL_SIZE 配置 SQLAlchemy 的连接池大小
20     app.config["SQLALCHEMY_POOL_SIZE"] = 5
21     # SQLALCHEMY_POOL_TIMEOUT 配置 SQLAlchemy 的连接超时时间
22     app.config["SQLALCHEMY_POOL_TIMEOUT"] = 15
23     app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
24 
25     # 初始化SQLAlchemy , 本质就是将以上的配置读取出来
26     db.init_app(app)
27 
28     app.register_blueprint(user)
29 
30     return app
```
{% endraw %}

### 2.建立models.py ORM模型文件

{% raw %}
```
 1 from MyApp import db
 2 
 3 Base = db.Model # 这句话你是否还记的?
 4 # from sqlalchemy.ext.declarative import declarative_base
 5 # Base = declarative_base()
 6 # 每一次我们在创建数据表的时候都要做这样一件事
 7 # 然而Flask-SQLAlchemy已经为我们把 Base 封装好了
 8 
 9 # 建立User数据表
10 class Users(Base): # Base实际上就是 db.Model
11     __tablename__ = "users"
12     __table_args__ = {"useexisting": True}
13     # 在SQLAlchemy 中我们是导入了Column和数据类型 Integer 在这里
14     # 就和db.Model一样,已经封装好了
15     id = db.Column(db.Integer,primary_key=True)
16     username = db.Column(db.String(32))
17     password = db.Column(db.String(32))
18 
19 
20 if __name__ == '__main__':
21     from MyApp import create_app
22     app = create_app()
23     # 这里你要回顾一下Flask应该上下文管理了
24     # 离线脚本:
25     with app.app_context():
26         db.drop_all()
27         db.create_all()
```
{% endraw %}

### 3.登录视图函数的应用

{% raw %}
```
 1 from flask import Blueprint, request, render_template
 2 
 3 user = Blueprint("user", __name__)
 4 
 5 from MyApp.models import Users
 6 from MyApp import db
 7 
 8 @user.route("/login",methods=["POST","GET"])
 9 def user_login():
10     if request.method == "POST":
11         username = request.form.get("username")
12         password = request.form.get("password")
13 
14         # 还记不记得我们的
15         # from sqlalchemy.orm import sessionmaker
16         # Session = sessionmaker(engine)
17         # db_sesson = Session()
18         # 现在不用了,因为 Flask-SQLAlchemy 也已经为我们做好会话打开的工作
19         # 我们在这里做个弊:
20         db.session.add(Users(username=username,password=password))
21         db.session.commit()
22 
23         # 然后再查询,捏哈哈哈哈哈
24         user_info = Users.query.filter(Users.username == username and User.password == password).first()
25         print(user_info.username)
26         if user_info:
27             return f"登录成功{user_info.username}"
28 
29     return render_template("login.html")
```
{% endraw %}

其实Flask-SQLAlchemy比起SQLAlchemy更加的简单自如,用法几乎一模一样,就是在配置和启动上需要注意与Flask的配合就好啦

到这里Flask-SQLAlchemy告一段落
