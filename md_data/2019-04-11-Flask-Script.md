---
layout:     post
title:      Flask-Script
subtitle:   
date:       2019-04-11
author:     P
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - python
---
其实本章就是为下一章做的铺垫啦,但是也要认真学习哦

Flask-Script 从字面意思上来看就是 Flask 的脚本

是的,熟悉Django的同学是否还记得Django的启动命令呢? python manager.py runserver 大概是这样对吧

其实Flask也可以做到,基于 Flask-Script 就可以了 - 但是你还是得有一个项目就是[**第十四章**](https://www.cnblogs.com/presleyren/p/10686116.html)的项目 **[点击下载](https://pan.baidu.com/s/123TyVXOFvh5P7V8MbyMfDg)**

### 1.安装 Flask-Script

```
pip install Flask-Script
```

### 2.将 Flask-Script 加入到 Flask 项目中

```
 1 import MyApp
 2 # 导入 Flask-Script 中的 Manager
 3 from flask_script import Manager
 4 
 5 app = MyApp.create_app()
 6 # 让app支持 Manager
 7 manager = Manager(app)
 8 
 9 if __name__ == '__main__':
10     #app.run()
11     # 替换原有的app.run(),然后大功告成了
12     manager.run()
```

### 3.使用命令启动 Flask 项目

```
python manager.py runserver
```

<img src="https://img2018.cnblogs.com/blog/1122946/201902/1122946-20190212163434616-849307789.png" alt="" />

### 4.启动项目并更改配置参数(监听IP地址,监听端口)

```
python manager.py runserver -h 0.0.0.0 -p 9527
```

<img src="https://img2018.cnblogs.com/blog/1122946/201902/1122946-20190212163635071-724007567.png" alt="" />

### 5.高级操作 - 自定制脚本命令

5.1.方式一 : @manager.command

```
 1 import MyApp
 2 # 导入 Flask-Script 中的 Manager
 3 from flask_script import Manager
 4 
 5 app = MyApp.create_app()
 6 # 让app支持 Manager
 7 manager = Manager(app) # type:Manager
 8 
 9 @manager.command
10 def DragonFire(arg):
11     print(arg)
12 
13 if __name__ == '__main__':
14     #app.run()
15     # 替换原有的app.run(),然后大功告成了
16     manager.run()
```

```
python manager.py DragonFire 666
```

<img src="https://img2018.cnblogs.com/blog/1122946/201902/1122946-20190212164132650-1912565267.png" alt="" />

5.2.方式二 : @manager.opation("-短指令","--长指令",dest="变量名")

```
 1 import MyApp
 2 # 导入 Flask-Script 中的 Manager
 3 from flask_script import Manager
 4 
 5 app = MyApp.create_app()
 6 # 让app支持 Manager
 7 manager = Manager(app) # type:Manager
 8 
 9 @manager.command
10 def DragonFire(arg):
11     print(arg)
12 
13 @manager.option("-n","--name",dest="name")
14 @manager.option("-s","--say",dest="say")
15 def talk(name,say):
16     print(f"{name}你可真{say}")
17 
18 if __name__ == '__main__':
19     #app.run()
20     # 替换原有的app.run(),然后大功告成了
21     manager.run()
```

```
python manager.py talk -n 赵丽颖 -s 漂亮
python manager.py talk --name DragonFire --say NB-Class
```

<img src="https://img2018.cnblogs.com/blog/1122946/201902/1122946-20190212164705293-1525668161.png" alt="" />

Flask-Script 完结~

后续更精彩哦
