---
layout:     post
title:      Flask中内置的Session
subtitle:   
date:       2019-04-11
author:     P
header-img: img/post-bg-BJJ.jpg
catalog: true
tags:
    - python
---
Flask中的Session非常的奇怪,他会将你的SessionID存放在客户端的Cookie中,使用起来也非常的奇怪

1. Flask 中 session 是需要 secret_key 的

```
from flask import session
app = Flask(__name__)
app.secret_key = "DragonFire"
```

2. session 要这样用

```
@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        if request.form["username"] == USER["username"] and request.form["password"] == USER["password"]:
            session["user"] = USER["username"]
            return redirect("/student_list")
        return render_template("login.html", msg="用户名密码错误")

    return render_template("login.html", msg=None)  # 如果前端Jinja2模板中使用了msg,这里就算是传递None也要出现msg
```

3. cookies 中的 session 是什么

cookies 中 session 存储的是通过 secret_key 加密后的 key , 通过这个 key 从flask程序的内存中找到用户对应的session信息

4. 怎么用 session 进行验证呢?

```
@app.route("/student_list")
def student():
    if session.get("user"):
        return render_template("student_list.html", student=STUDENT_DICT)

    return redirect("/login")
```

如果这个你要是看不明白的,我只能从基础给你讲了

第六篇,完结
