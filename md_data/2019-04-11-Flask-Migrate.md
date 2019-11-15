---
layout:     post
title:      Flask-Migrate
subtitle:   
date:       2019-04-11
author:     P
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - python
---
终于到了Flask-Migrate,之前在学习Flask-SQLAlchemy的时候,有的同学就提过类似的问题,Flask支持 makemigration / migrate 吗?

答案在这里该诉你,如果你同时拥有两个三方组件 Flask-Script 和 Flask-Migrate 那么就支持这样的动作

首先你要有几个准备工作

[**第十五章的知识回顾**](https://www.cnblogs.com/presleyren/p/10692907.html)

**[第十五章的项目下载](https://pan.baidu.com/s/1WqdINYFPt3r-CEqOOC3Gog)**

废话不多说,直接进入正题

### 1.安装 Flask-Migrate

```
pip install Flask-Migrate
```

### 2.将 Flask-Migrate 加入到 Flask 项目中 - **PS: 注意了 Flask-Migrate 是要依赖 Flask-Script 组件的**

```
 1 import MyApp
 2 # 导入 Flask-Script 中的 Manager
 3 from flask_script import Manager
 4 
 5 # 导入 Flask-Migrate 中的 Migrate 和 MigrateCommand
 6 # 这两个东西说白了就是想在 Flask-Script 中添加几个命令和指令而已
 7 from flask_migrate import Migrate,MigrateCommand
 8 
 9 app = MyApp.create_app()
10 # 让app支持 Manager
11 manager = Manager(app) # type:Manager
12 
13 # Migrate 既然是数据库迁移,那么就得告诉他数据库在哪里
14 # 并且告诉他要支持那个app
15 Migrate(app,MyApp.db)
16 # 现在就要告诉manager 有新的指令了,这个新指令在MigrateCommand 中存着呢
17 manager.add_command("db",MigrateCommand) # 当你的命令中出现 db 指令,则去MigrateCommand中寻找对应关系
18 """
19 数据库迁移指令:
20 python manager.py db init 
21 python manager.py db migrate   # Django中的 makemigration
22 python manager.py db upgrade  # Django中的 migrate
23 """
24 
25 
26 @manager.command
27 def DragonFire(arg):
28     print(arg)
29 
30 @manager.option("-n","--name",dest="name")
31 @manager.option("-s","--say",dest="say")
32 def talk(name,say):
33     print(f"{name}你可真{say}")
34 
35 if __name__ == '__main__':
36     #app.run()
37     # 替换原有的app.run(),然后大功告成了
38     manager.run()
```

### 3.执行数据库初始化指令

```
python manager.py db init
```

<img src="https://img2018.cnblogs.com/blog/1122946/201902/1122946-20190212170518784-240870796.png" alt="" />

此时你会发现你的项目目录中出现了一个好玩儿的东西

<img src="https://img2018.cnblogs.com/blog/1122946/201902/1122946-20190212170557810-1492485776.png" alt="" />

接下来的操作就和Django中一样了,在这里就不做演示了

本章结束
