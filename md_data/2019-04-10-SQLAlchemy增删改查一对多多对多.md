---
layout:     post
title:      SQLAlchemy增删改查一对多多对多
subtitle:   
date:       2019-04-10
author:     P
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - python
---
好久没有更新Blog了

今天来聊一聊 Python 的 ORM 框架 SQLAlchemy 有些同学已经听说过这个框架的大名了,也听说了 SQLAlchemy 没有 Django 的 Models 好用

我在这里官宣辟谣一下啊, Models 紧紧只是配置和使用比较简单(这特么就是废话),因为他是Django自带的ORM框架,也正是因为是Django原生的,所以兼容性远远不如SQLAlchemy

真正算得上全面的ORM框架必然是我们的SQLAlchemy ORM框架,它可以在任何使用SQL查询时使用

当然了,无论是使用什么ORM框架,都是为了方便不熟练数据库的同学使用的,我个人还是比较推崇原生 SQL ,也建议每位同学攻克 SQL 的难关

废话不多说,我们来看一下 SQLAlchemy 如何使用:

### 1.创建数据表

```
 1 # ORM中的数据表是什么呢?
 2 # Object Relation Mapping
 3 # Object - Table 通过 Object 去操纵数据表
 4 # 从而引出了我们的第一步创建数据表 - 创建Object
 5 # 1. 创建Object
 6 # class User(object):
 7 #     pass
 8 
 9 # 2. 让Object与数据表产生某种关系 也就是让Object与数据表格式极度相似
10 # 导入官宣基础模型
11 from sqlalchemy.ext.declarative import declarative_base
12 # 实例化官宣模型 - Base 就是 ORM 模型
13 Base = declarative_base()
14 # 当前的这个Object继承了Base也就是代表了Object继承了ORM的模型
15 class User(Base):  # 相当于 Django Models中的 Model
16     # 为Table创建名称
17     __tablename__ = "user"
18     # 创建ID数据字段 , 那么ID是不是一个数据列呢? 也就是说创建ID字段 == 创建ID数据列
19     from sqlalchemy import Column,Integer,String
20     # id = Column(数据类型,索引,主键,外键,等等)
21     # int == Integer
22     id = Column(Integer,primary_key=True,autoincrement=True)
23     # str == char(长度) == String(长度)
24     name = Column(String(32),index=True)
25 
26 # 3.去数据库中创建数据表? or 先连接数据库?
27 # 3.去连接数据库 创建数据引擎
28 from sqlalchemy import create_engine
29 # 创建的数据库引擎
30 engine = create_engine("mysql+pymysql://root:DragonFire@127.0.0.1:3306/dragon?charset=utf8")
31 
32 # Base 自动检索所有继承Base的ORM 对象 并且创建所有的数据表
33 Base.metadata.create_all(engine)
```

### 2.增删改查操作

2.1.增加数据

```
 1 #insert 为数据表增加数据
 2 # insert One 增加一行数据
 3 # insert into user(name) values ("DragonFire")
 4 # 在ORM中的操作:
 5 # 1.首先导入之间做好的ORM 对象 User
 6 from my_create_table import User
 7 # 2.使用Users ORM模型创建一条数据
 8 user1 = User(name="DragonFire")
 9 # 数据已经创建完了,但是需要写入到数据库中啊,怎么写入呢?
10 # 3.写入数据库:
11 # 首先打开数据库会话 , 说白了就是创建了一个操纵数据库的窗口
12 # 导入 sqlalchemy.orm 中的 sessionmaker
13 from sqlalchemy.orm import sessionmaker
14 # 导入之前创建好的 create_engine
15 from my_create_table import engine
16 # 创建 sessionmaker 会话对象,将数据库引擎 engine 交给 sessionmaker
17 Session = sessionmaker(engine)
18 # 打开会话对象 Session
19 db_session = Session()
20 # 在db_session会话中添加一条 UserORM模型创建的数据
21 db_session.add(user1)
22 # 使用 db_session 会话提交 , 这里的提交是指将db_session中的所有指令一次性提交
23 db_session.commit()
24 
25 # 当然也你也可很任性的提交多条数据
26 # 方法一:
27 user2 = User(name="Dragon")
28 user3 = User(name="Fire")
29 db_session.add(user2)
30 db_session.add(user3)
31 db_session.commit()
32 # 之前说过commit是将db_session中的所有指令一次性提交,现在的db_session中至少有两条指令user2和user3
33 db_session.close()
34 #关闭会话
35 
36 # 如果说你觉得方法一很麻烦,那么方法二一定非常非常适合你
37 # 方法二:
38 user_list = [
39     User(name="Dragon1"),
40     User(name="Dragon2"),
41     User(name="Dragon3")
42 ]
43 db_session.add_all(user_list)
44 db_session.commit()
45 
46 db_session.close()
```

2.2.查询数据

```
 1 # ORM操作查询数据
 2 # 有了刚才Insert增加数据的经验,那么查询之前的准备工作,就不用再重复了吧
 3 # 回想一下刚才Insert时我们的操作
 4 from my_create_table import User, engine
 5 from sqlalchemy.orm import sessionmaker
 6 
 7 Session = sessionmaker(engine)
 8 db_session = Session()
 9 
10 # 1. select * from user 查询user表中的所有数据
11 # 语法是这样的 使用 db_session 会话 执行User表 query(User) 取出全部数据 all()
12 user_all_list = db_session.query(User).all()
13 print(user_all_list)  # [<my_create_table.User object at 0x0000016D7C4BCDD8>]
14 # 如何查看user_all_list其中的数据呢? 循环呗
15 for i in user_all_list:
16     print(i.id, i.name)  # ORM对象 直接使用调用属性的方法 拿出对应字段的值
17 
18 db_session.close()
19 #关闭会话
20 
21 # 2. select * from user where id >= 20
22 # 语法是这样的 使用 db_session 会话 执行User表 query(User) 筛选内容User.id >=20 的数据全部取出 all()
23 user_all_list = db_session.query(User).filter(User.id >= 20).all()
24 print(user_all_list)
25 for i in user_all_list:
26     print(i.id, i.name)
27 
28 db_session.close()
29 #关闭会话
30 
31 # 3. 除了取出全部还可以只取出一条
32 user = db_session.query(User).filter(User.id >= 20).first()
33 print(user.id, user.name)
34 db_session.close()
35 #关闭会话
36 
37 # 4. 乌龙 之 忘了取出数据.......
38 wulong1 = db_session.query(User).filter(User.id >= 20)
39 print(wulong1)
40 #SELECT user.id AS user_id, user.name AS user_name
41 #FROM user
42 #WHERE user.id >= %(id_1)s
43 # Fuck我忘了取出数据了!!!!!!! 哎? wulong1给我显示了原生SQL语句,因祸得福了
44 wulong2 = db_session.query(User)
45 print(wulong2)
46 #SELECT user.id AS user_id, user.name AS user_name
47 #FROM user
48 # Fuck我又忘了取出数据了!!!!!!! 哎? wulong2给我显示了原生SQL语句,因祸得福了
49 db_session.close()
50 #关闭会话
```

2.3.修改数据

```
 1 # ORM更新数据
 2 # 无论是更新还是删除,首先要做的事情,就应该是查询吧
 3 # 根据之前原有的经验,接下来是不是要导入ORM对象了,是不是要创建db_session会话了
 4 from my_create_table import User,engine
 5 from sqlalchemy.orm import sessionmaker
 6 Session = sessionmaker(engine)
 7 db_session = Session()
 8 
 9 # UPDATE user SET name="NBDragon" WHERE id=20 更新一条数据
10 # 语法是这样的 :
11 # 使用 db_session 执行User表 query(User) 筛选 User.id = 20 的数据 filter(User.id == 20)
12 # 将name字段的值改为NBDragon update({"name":"NBDragon"})
13 res = db_session.query(User).filter(User.id == 20).update({"name":"NBDragon"})
14 print(res) # 1 res就是我们当前这句更新语句所更新的行数
15 # 注意注意注意
16 # 这里一定要将db_session中的执行语句进行提交,因为你这是要对数据中的数据进行操作
17 # 数据库中 增 改 删 都是操作,也就是说执行以上三种操作的时候一定要commit
18 db_session.commit()
19 db_session.close()
20 #关闭会话
21 
22 # 更新多条
23 res = db_session.query(User).filter(User.id <= 20).update({"name":"NBDragon"})
24 print(res) # 6 res就是我们当前这句更新语句所更新的行数
25 db_session.commit()
26 db_session.close()
27 #关闭会话
```

2.4.删除数据

```
 1 # ORM 删除一条多条数据
 2 # 老规矩
 3 # 导入 ORM 创建会话
 4 from my_create_table import User,engine
 5 from sqlalchemy.orm import sessionmaker
 6 Session = sessionmaker(engine)
 7 db_session = Session()
 8 
 9 # DELETE FROM `user` WHERE id=20
10 res = db_session.query(User).filter(User.id==20).delete()
11 print(res)
12 # 是删除操作吧,没错吧,那你想什么呢?commit吧
13 db_session.commit()
14 
15 db_session.close()
16 #关闭会话
```

2.5.搞基!高级版查询操作

```
  1 # 高级版查询操作,厉害了哦
  2 #老规矩
  3 from my_create_table import User,engine
  4 from sqlalchemy.orm import sessionmaker
  5 
  6 Session = sessionmaker(engine)
  7 db_session = Session()
  8 
  9 # 查询数据表操作
 10 # and or
 11 from sqlalchemy.sql import and_ , or_
 12 ret = db_session.query(User).filter(and_(User.id > 3, User.name == 'DragonFire')).all()
 13 ret = db_session.query(User).filter(or_(User.id < 2, User.name == 'DragonFire')).all()
 14 
 15 # 查询所有数据
 16 r1 = db_session.query(User).all()
 17 
 18 # 查询数据 指定查询数据列 加入别名
 19 r2 = db_session.query(User.name.label('username'), User.id).first()
 20 print(r2.id,r2.username) # 15 NBDragon
 21 
 22 # 表达式筛选条件
 23 r3 = db_session.query(User).filter(User.name == "DragonFire").all()
 24 
 25 # 原生SQL筛选条件
 26 r4 = db_session.query(User).filter_by(name='DragonFire').all()
 27 r5 = db_session.query(User).filter_by(name='DragonFire').first()
 28 
 29 # 字符串匹配方式筛选条件 并使用 order_by进行排序
 30 r6 = db_session.query(User).filter(text("id<:value and name=:name")).params(value=224, name='DragonFire').order_by(User.id).all()
 31 
 32 #原生SQL查询
 33 r7 = db_session.query(User).from_statement(text("SELECT * FROM User where name=:name")).params(name='DragonFire').all()
 34 
 35 # 筛选查询列
 36 # query的时候我们不在使用User ORM对象,而是使用User.name来对内容进行选取
 37 user_list = db_session.query(User.name).all()
 38 print(user_list)
 39 for row in user_list:
 40     print(row.name)
 41 
 42 # 别名映射  name as nick
 43 user_list = db_session.query(User.name.label("nick")).all()
 44 print(user_list)
 45 for row in user_list:
 46     print(row.nick) # 这里要写别名了
 47 
 48 # 筛选条件格式
 49 user_list = db_session.query(User).filter(User.name == "DragonFire").all()
 50 user_list = db_session.query(User).filter(User.name == "DragonFire").first()
 51 user_list = db_session.query(User).filter_by(name="DragonFire").first()
 52 for row in user_list:
 53     print(row.nick)
 54 
 55 # 复杂查询
 56 from sqlalchemy.sql import text
 57 user_list = db_session.query(User).filter(text("id<:value and name=:name")).params(value=3,name="DragonFire")
 58 
 59 # 查询语句
 60 from sqlalchemy.sql import text
 61 user_list = db_session.query(User).filter(text("select * from User id<:value and name=:name")).params(value=3,name="DragonFire")
 62 
 63 # 排序 :
 64 user_list = db_session.query(User).order_by(User.id).all()
 65 user_list = db_session.query(User).order_by(User.id.desc()).all()
 66 for row in user_list:
 67     print(row.name,row.id)
 68 
 69 #其他查询条件
 70 """
 71 ret = session.query(User).filter_by(name='DragonFire').all()
 72 ret = session.query(User).filter(User.id > 1, User.name == 'DragonFire').all()
 73 ret = session.query(User).filter(User.id.between(1, 3), User.name == 'DragonFire').all() # between 大于1小于3的
 74 ret = session.query(User).filter(User.id.in_([1,3,4])).all() # in_([1,3,4]) 只查询id等于1,3,4的
 75 ret = session.query(User).filter(~User.id.in_([1,3,4])).all() # ~xxxx.in_([1,3,4]) 查询不等于1,3,4的
 76 ret = session.query(User).filter(User.id.in_(session.query(User.id).filter_by(name='DragonFire'))).all() 子查询
 77 from sqlalchemy import and_, or_
 78 ret = session.query(User).filter(and_(User.id > 3, User.name == 'DragonFire')).all()
 79 ret = session.query(User).filter(or_(User.id < 2, User.name == 'DragonFire')).all()
 80 ret = session.query(User).filter(
 81     or_(
 82         User.id < 2,
 83         and_(User.name == 'eric', User.id > 3),
 84         User.extra != ""
 85     )).all()
 86 # select * from User where id<2 or (name="eric" and id>3) or extra != "" 
 87 
 88 # 通配符
 89 ret = db_session.query(User).filter(User.name.like('e%')).all()
 90 ret = db_session.query(User).filter(~User.name.like('e%')).all()
 91 
 92 # 限制
 93 ret = db_session.query(User)[1:2]
 94 
 95 # 排序
 96 ret = db_session.query(User).order_by(User.name.desc()).all()
 97 ret = db_session.query(User).order_by(User.name.desc(), User.id.asc()).all()
 98 
 99 # 分组
100 from sqlalchemy.sql import func
101 
102 ret = db_session.query(User).group_by(User.extra).all()
103 ret = db_session.query(
104     func.max(User.id),
105     func.sum(User.id),
106     func.min(User.id)).group_by(User.name).all()
107 
108 ret = db_session.query(
109     func.max(User.id),
110     func.sum(User.id),
111     func.min(User.id)).group_by(User.name).having(func.min(User.id) >2).all()
112 """
113 
114 # 关闭连接
115 db_session.close()
```

2.6.高级修改数据操作

```
 1 #高级版更新操作
 2 from my_create_table import User,engine
 3 from sqlalchemy.orm import sessionmaker
 4 
 5 Session = sessionmaker(engine)
 6 db_session = Session()
 7 
 8 #直接修改
 9 db_session.query(User).filter(User.id > 0).update({"name" : "099"})
10 
11 #在原有值基础上添加 - 1
12 db_session.query(User).filter(User.id > 0).update({User.name: User.name + "099"}, synchronize_session=False)
13 
14 #在原有值基础上添加 - 2
15 db_session.query(User).filter(User.id > 0).update({"age": User.age + 1}, synchronize_session="evaluate")
16 db_session.commit()
```

### 3.一对多的操作 : ForeignKey

3.1.创建数据表及关系relationship:

```
 1 from sqlalchemy.ext.declarative import declarative_base
 2 
 3 Base = declarative_base()
 4 
 5 # 这次我们要多导入一个 ForeignKey 字段了,外键关联对了
 6 from sqlalchemy import Column,Integer,String,ForeignKey
 7 # 还要从orm 中导入一个 relationship 关系映射
 8 from sqlalchemy.orm import relationship
 9 
10 class ClassTable(Base):
11     __tablename__="classtable"
12     id = Column(Integer,primary_key=True)
13     name = Column(String(32),index=True)
14 
15 class Student(Base):
16     __tablename__="student"
17     id=Column(Integer,primary_key=True)
18     name = Column(String(32),index=True)
19 
20     # 关联字段,让class_id 与 class 的 id 进行关联,主外键关系(这里的ForeignKey一定要是表名.id不是对象名)
21     class_id = Column(Integer,ForeignKey("classtable.id"))
22 
23     # 将student 与 classtable 创建关系 这个不是字段,只是关系,backref是反向关联的关键字
24     to_class = relationship("ClassTable",backref = "stu2class")
25 
26 from sqlalchemy import create_engine
27 engine = create_engine("mysql+pymysql://root:DragonFire@127.0.0.1:3306/dragon?charset=utf8")
28 
29 Base.metadata.create_all(engine)
```

3.2.基于relationship增加数据

```
 1 from my_ForeignKey import Student, ClassTable,engine
 2 # 创建连接
 3 from sqlalchemy.orm import sessionmaker
 4 # 创建数据表操作对象 sessionmaker
 5 DB_session = sessionmaker(engine)
 6 db_session = DB_session()
 7 
 8 # 增加数据
 9 # 1.简单增加数据
10 # 添加两个班级:
11 # db_session.add_all([
12 #     ClassTable(name="OldBoyS1"),
13 #     ClassTable(name="OldBoyS2")
14 # ])
15 # db_session.commit()
16 # 添加一个学生 DragonFire 班级是 OldBoyS1
17 # 查询要添加到的班级
18 # class_obj = db_session.query(ClassTable).filter(ClassTable.name == "OldBoyS1").first()
19 # 创建学生
20 # stu = Student(name="DragonFire",class_id = class_obj.id)
21 # db_session.add(stu)
22 # db_session.commit()
23 
24 # 2. relationship版 添加数据
25 # 通过关系列 to_class 可以做到两件事
26 # 第一件事 在ClassTable表中添加一条数据
27 # 第二件事 在Student表中添加一条数据并将刚刚添加的ClassTable的数据id填写在Student的class_id中
28 # stu_cla = Student(name="DragonFire",to_class=ClassTable(name="OldBoyS1"))
29 # print(stu_cla.name,stu_cla.class_id)
30 # db_session.add(stu_cla)
31 # db_session.commit()
32 
33 # 3.relationship版 反向添加数据
34 # 首先建立ClassTable数据
35 class_obj = ClassTable(name="OldBoyS2")
36 # 通过class_obj中的反向关联字段backref - stu2class
37 # 向 Student 数据表中添加 2条数据 并将 2条数据的class_id 写成 class_obj的id
38 # class_obj.stu2class = [Student(name="BMW"),Student(name="Audi")]
39 # db_session.add(class_obj)
40 # db_session.commit()
41 
42 # 关闭连接
43 db_session.close()
```

3.3.基于relationship查询数据

```
 1 from my_ForeignKey import Student, ClassTable,engine
 2 
 3 from sqlalchemy.orm import sessionmaker
 4 DB_session = sessionmaker(engine)
 5 db_session = DB_session()
 6 
 7 # 1.查询所有数据,并显示班级名称,连表查询
 8 student_list = db_session.query(Student).all()
 9 for row in student_list:
10     # row.to_class.name 通过Student对象中的关系字段relationship to_class 获取关联 ClassTable中的name
11     print(row.name,row.to_class.name,row.class_id)
12 
13 # 2.反向查询
14 class_list = db_session.query(ClassTable).all()
15 for row in class_list:
16     for row2 in row.stu2class:
17         print(row.name,row2.name)
18 # row.stu2class 通过 backref 中的 stu2class 反向关联到 Student 表中根据ID获取name
19 
20 
21 db_session.close()
```

3.4.更新数据

```
 1 from my_ForeignKey import Student, ClassTable,engine
 2 
 3 from sqlalchemy.orm import sessionmaker
 4 DB_session = sessionmaker(engine)
 5 db_session = DB_session()
 6 
 7 # 更新
 8 class_info = db_session.query(ClassTable).filter(ClassTable.name=="OldBoyS1").first()
 9 db_session.query(Student).filter(Student.class_id == class_info.id).update({"name":"NBDragon"})
10 db_session.commit()
11 
12 db_session.close()
```

3.5.删除数据

```
 1 from my_ForeignKey import Student, ClassTable,engine
 2 
 3 from sqlalchemy.orm import sessionmaker
 4 DB_session = sessionmaker(engine)
 5 db_session = DB_session()
 6 
 7 # 删除
 8 class_info = db_session.query(ClassTable).filter(ClassTable.name=="OldBoyS1").first()
 9 db_session.query(Student).filter(Student.class_id == class_info.id).delete()
10 db_session.commit()
11 
12 db_session.close()
```

### 4.多对多 : ManyToMany

4.1.创建表及关系

```
 1 from sqlalchemy.ext.declarative import declarative_base
 2 
 3 Base = declarative_base()
 4 
 5 from sqlalchemy import Column,Integer,String,ForeignKey
 6 from sqlalchemy.orm import relationship
 7 
 8 class Hotel(Base):
 9     __tablename__="hotel"
10     id=Column(Integer,primary_key=True)
11     girl_id = Column(Integer,ForeignKey("girl.id"))
12     boy_id = Column(Integer,ForeignKey("boy.id"))
13 
14 class Girl(Base):
15     __tablename__="girl"
16     id=Column(Integer,primary_key=True)
17     name = Column(String(32),index=True)
18 
19     #创建关系
20     boys = relationship("Boy",secondary="hotel",backref="girl2boy")
21 
22 
23 class Boy(Base):
24     __tablename__="boy"
25     id=Column(Integer,primary_key=True)
26     name = Column(String(32),index=True)
27 
28 
29 from sqlalchemy import create_engine
30 engine = create_engine("mysql+pymysql://root:DragonFire@127.0.0.1:3306/dragon?charset=utf8")
31 
32 Base.metadata.create_all(engine)
```

4.2.基于relationship增加数据

```
 1 from my_M2M import Girl,Boy,Hotel,engine
 2 
 3 # 创建连接
 4 from sqlalchemy.orm import sessionmaker
 5 # 创建数据表操作对象 sessionmaker
 6 DB_session = sessionmaker(engine)
 7 db_session = DB_session()
 8 
 9 # 1.通过Boy添加Girl和Hotel数据
10 boy = Boy(name="DragonFire")
11 boy.girl2boy = [Girl(name="赵丽颖"),Girl(name="Angelababy")]
12 db_session.add(boy)
13 db_session.commit()
14 
15 # 2.通过Girl添加Boy和Hotel数据
16 girl = Girl(name="珊珊")
17 girl.boys = [Boy(name="Dragon")]
18 db_session.add(girl)
19 db_session.commit()
```

4.3.基于relationship查询数据

```
 1 from my_M2M import Girl,Boy,Hotel,engine
 2 
 3 # 创建连接
 4 from sqlalchemy.orm import sessionmaker
 5 # 创建数据表操作对象 sessionmaker
 6 DB_session = sessionmaker(engine)
 7 db_session = DB_session()
 8 
 9 # 1.通过Boy查询约会过的所有Girl
10 hotel = db_session.query(Boy).all()
11 for row in hotel:
12     for row2 in row.girl2boy:
13         print(row.name,row2.name)
14 
15 # 2.通过Girl查询约会过的所有Boy
16 hotel = db_session.query(Girl).all()
17 for row in hotel:
18     for row2 in row.boys:
19         print(row.name,row2.name)
```
