---
layout:     post
title:      Django之ORM
subtitle:   
date:       2019-01-22
author:     P
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - python
---
## 目录

<li>[ORM介绍](#_label0)
<ul>
- [ORM概念](#_label0_0)
- [ORM由来](#_label0_1)
- [ORM的优势](#_label0_2)
- [ORM的劣势](#_label0_3)
- [ORM总结](#_label0_4)

- [Django项目使用MySQL数据库](#_label1_0)
- [Model](#_label1_1)
- [快速入门 ](#_label1_2)
- [字段 ](#_label1_3)
- [自定义字段](#_label1_4)
- [字段参数](#_label1_5)
- [Model Meta参数](#_label1_6)
- [多表关系和参数](#_label1_7)
- [ORM操作](#_label1_8)

# Object Relational Mapping(ORM)

## ORM介绍

### ORM概念

对象关系映射（Object Relational Mapping，简称ORM）模式是一种为了解决面向对象与关系数据库存在的互不匹配的现象的技术。

简单的说，ORM是通过使用描述对象和数据库之间映射的元数据，将程序中的对象自动持久化到关系数据库中。

ORM在业务逻辑层和数据库层之间充当了桥梁的作用。

### ORM由来

让我们从O/R开始。字母O起源于"对象"(Object)，而R则来自于"关系"(Relational)。

几乎所有的软件开发过程中都会涉及到对象和关系数据库。在用户层面和业务逻辑层面，我们是面向对象的。当对象的信息发生变化的时候，我们就需要把对象的信息保存在关系数据库中。

按照之前的方式来进行开发就会出现程序员会在自己的业务逻辑代码中夹杂很多SQL语句用来增加、读取、修改、删除相关数据，而这些代码通常都是极其相似或者重复的。

### ORM的优势

ORM解决的主要问题是对象和关系的映射。它通常将一个类和一张表一一对应，类的每个实例对应表中的一条记录，类的每个属性对应表中的每个字段。 

ORM提供了对数据库的映射，不用直接编写SQL代码，只需操作对象就能对数据库操作数据。

让软件开发人员专注于业务逻辑的处理，提高了开发效率。

### ORM的劣势

ORM的缺点是会在一定程度上牺牲程序的执行效率。

ORM的操作是有限的，也就是ORM定义好的操作是可以完成的，一些复杂的查询操作是完成不了。

ORM用多了SQL语句就不会写了，关系数据库相关技能退化...

### ORM总结

ORM只是一种工具，工具确实能解决一些重复，简单的劳动。这是不可否认的。

但我们不能指望某个工具能一劳永逸地解决所有问题，一些特殊问题还是需要特殊处理的。

但是在整个软件开发过程中需要特殊处理的情况应该都是很少的，否则所谓的工具也就失去了它存在的意义。

## Django中的ORM

### Django项目使用MySQL数据库

1. 在Django项目的settings.py文件中，配置数据库连接信息：
<td class="gutter">12345678910</td><td class="code">`DATABASES ``=` `{``    ``"default"``: {``        ``"ENGINE"``: ``"django.db.backends.mysql"``,``        ``"NAME"``: ``"你的数据库名称"``,  ``# 需要自己手动创建数据库``        ``"USER"``: ``"数据库用户名"``,``        ``"PASSWORD"``: ``"数据库密码"``,``        ``"HOST"``: ``"数据库IP"``,``        ``"POST"``: ``3306``    ``}``}`</td>

2. 在与Django项目同名的目录下的__init__.py文件中写如下代码，告诉Django使用pymysql模块连接MySQL数据库:
<td class="gutter">123</td><td class="code">`import` `pymysql` `pymysql.install_as_MySQLdb()`</td>

注：数据库迁移的时候出现一个警告

```
WARNINGS: 
?: (mysql.W002) MySQL Strict Mode is not set for database connection 'default'
HINT: MySQL's Strict Mode fixes many data integrity problems in MySQL, such as data truncation upon insertion, by escalating warnings into errors. It is strongly recommended you activate it.
```

在配置中多加一个OPTIONS参数：[Django官网解释](https://docs.djangoproject.com/en/1.11/ref/databases/#setting-sql-mode)

```
 'OPTIONS': {
    'init_command': "SET sql_mode='STRICT_TRANS_TABLES'"},
```

### Model

在Django中model是你数据的单一、明确的信息来源。它包含了你存储的数据的重要字段和行为。通常，一个模型（model）映射到一个数据库表。

基本情况：

- 每个模型都是一个Python类，它是django.db.models.Model的子类。
- 模型的每个属性都代表一个数据库字段。
- 综上所述，Django为您提供了一个自动生成的数据库访问API，详询[官方文档链接](https://docs.djangoproject.com/en/1.11/topics/db/queries/)。

<img src="https://images2018.cnblogs.com/blog/867021/201803/867021-20180325235218756-104285201.png" alt="" />

### 快速入门 

下面这个例子定义了一个 **Person** 模型，包含 **first_name **和 **last_name**。
<td class="gutter">12345</td><td class="code">`from` `django.db ``import` `models` `class` `Person(models.Model):``    ``first_name ``=` `models.CharField(max_length``=``30``)``    ``last_name ``=` `models.CharField(max_length``=``30``)`</td>

**first_name **和 **last_name** 是模型的字段。每个字段被指定为一个类属性，每个属性映射到一个数据库列。

上面的 **Person** 模型将会像这样创建一个数据库表：
<td class="gutter">12345</td><td class="code">`CREATE` `TABLE` `myapp_person (``    ``"id"` `serial ``NOT` `NULL` `PRIMARY` `KEY``,``    ``"first_name"` `varchar``(30) ``NOT` `NULL``,``    ``"last_name"` `varchar``(30) ``NOT` `NULL``);`</td>

一些说明：

- 表myapp_person的名称是自动生成的，如果你要自定义表名，需要在model的Meta类中指定 db_table 参数，强烈建议使用小写表名，特别是使用MySQL作为数据库时。
- id字段是自动添加的，如果你想要指定自定义主键，只需在其中一个字段中指定 primary_key=True 即可。如果Django发现你已经明确地设置了Field.primary_key，它将不会添加自动ID列。
- 本示例中的CREATE TABLE SQL使用PostgreSQL语法进行格式化，但值得注意的是，Django会根据配置文件中指定的数据库类型来生成相应的SQL语句。
- Django支持MySQL5.5及更高版本。

### 字段 

**常用字段 **

**AutoField**

自增的整形字段，必填参数primary_key=True，则成为数据库的主键。无该字段时，django自动创建。

一个model不能有两个AutoField字段。

**IntegerField**

一个整数类型。数值的范围是 -2147483648 ~ 2147483647。

**CharField**

字符类型，必须提供max_length参数。max_length表示字符的长度。

**DateField**

日期类型，日期格式为YYYY-MM-DD，相当于Python中的datetime.date的实例。

参数：

- auto_now：每次修改时修改为当前日期时间。
- auto_now_add：新创建对象时自动添加当前日期时间。

auto_now和auto_now_add和default参数是互斥的，不能同时设置。

**DatetimeField**

日期时间字段，格式为YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ]，相当于Python中的datetime.datetime的实例。

字段类型，详情可点击查询[官网](https://docs.djangoproject.com/en/1.11/ref/models/fields/#field-types)。

```
    AutoField(Field)
        - int自增列，必须填入参数 primary_key=True

    BigAutoField(AutoField)
        - bigint自增列，必须填入参数 primary_key=True

        注：当model中如果没有自增列，则自动会创建一个列名为id的列
        from django.db import models

        class UserInfo(models.Model):
            # 自动创建一个列名为id的且为自增的整数列
            username = models.CharField(max_length=32)

        class Group(models.Model):
            # 自定义自增列
            nid = models.AutoField(primary_key=True)
            name = models.CharField(max_length=32)

    SmallIntegerField(IntegerField):
        - 小整数 -32768 ～ 32767

    PositiveSmallIntegerField(PositiveIntegerRelDbTypeMixin, IntegerField)
        - 正小整数 0 ～ 32767

    IntegerField(Field)
        - 整数列(有符号的) -2147483648 ～ 2147483647

    PositiveIntegerField(PositiveIntegerRelDbTypeMixin, IntegerField)
        - 正整数 0 ～ 2147483647

    BigIntegerField(IntegerField):
        - 长整型(有符号的) -9223372036854775808 ～ 9223372036854775807

    BooleanField(Field)
        - 布尔值类型

    NullBooleanField(Field):
        - 可以为空的布尔值

    CharField(Field)
        - 字符类型
        - 必须提供max_length参数， max_length表示字符长度

    TextField(Field)
        - 文本类型

    EmailField(CharField)：
        - 字符串类型，Django Admin以及ModelForm中提供验证机制

    IPAddressField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供验证 IPV4 机制

    GenericIPAddressField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供验证 Ipv4和Ipv6
        - 参数：
            protocol，用于指定Ipv4或Ipv6， 'both',"ipv4","ipv6"
            unpack_ipv4， 如果指定为True，则输入::ffff:192.0.2.1时候，可解析为192.0.2.1，开启此功能，需要protocol="both"

    URLField(CharField)
        - 字符串类型，Django Admin以及ModelForm中提供验证 URL

    SlugField(CharField)
        - 字符串类型，Django Admin以及ModelForm中提供验证支持 字母、数字、下划线、连接符（减号）

    CommaSeparatedIntegerField(CharField)
        - 字符串类型，格式必须为逗号分割的数字

    UUIDField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供对UUID格式的验证

    FilePathField(Field)
        - 字符串，Django Admin以及ModelForm中提供读取文件夹下文件的功能
        - 参数：
                path,                      文件夹路径
                match=None,                正则匹配
                recursive=False,           递归下面的文件夹
                allow_files=True,          允许文件
                allow_folders=False,       允许文件夹

    FileField(Field)
        - 字符串，路径保存在数据库，文件上传到指定目录
        - 参数：
            upload_to = ""      上传文件的保存路径
            storage = None      存储组件，默认django.core.files.storage.FileSystemStorage

    ImageField(FileField)
        - 字符串，路径保存在数据库，文件上传到指定目录
        - 参数：
            upload_to = ""      上传文件的保存路径
            storage = None      存储组件，默认django.core.files.storage.FileSystemStorage
            width_field=None,   上传图片的高度保存的数据库字段名（字符串）
            height_field=None   上传图片的宽度保存的数据库字段名（字符串）

    DateTimeField(DateField)
        - 日期+时间格式 YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ]

    DateField(DateTimeCheckMixin, Field)
        - 日期格式      YYYY-MM-DD

    TimeField(DateTimeCheckMixin, Field)
        - 时间格式      HH:MM[:ss[.uuuuuu]]

    DurationField(Field)
        - 长整数，时间间隔，数据库中按照bigint存储，ORM中获取的值为datetime.timedelta类型

    FloatField(Field)
        - 浮点型

    DecimalField(Field)
        - 10进制小数
        - 参数：
            max_digits，小数总长度
            decimal_places，小数位长度

    BinaryField(Field)
        - 二进制类型
```

### 自定义字段

自定义一个二进制字段，以及Django字段与数据库字段类型的对应关系。

```
class UnsignedIntegerField(models.IntegerField):
    def db_type(self, connection):
        return 'integer UNSIGNED'

# PS: 返回值为字段在数据库中的属性。
# Django字段与数据库字段类型对应关系如下：
    'AutoField': 'integer AUTO_INCREMENT',
    'BigAutoField': 'bigint AUTO_INCREMENT',
    'BinaryField': 'longblob',
    'BooleanField': 'bool',
    'CharField': 'varchar(%(max_length)s)',
    'CommaSeparatedIntegerField': 'varchar(%(max_length)s)',
    'DateField': 'date',
    'DateTimeField': 'datetime',
    'DecimalField': 'numeric(%(max_digits)s, %(decimal_places)s)',
    'DurationField': 'bigint',
    'FileField': 'varchar(%(max_length)s)',
    'FilePathField': 'varchar(%(max_length)s)',
    'FloatField': 'double precision',
    'IntegerField': 'integer',
    'BigIntegerField': 'bigint',
    'IPAddressField': 'char(15)',
    'GenericIPAddressField': 'char(39)',
    'NullBooleanField': 'bool',
    'OneToOneField': 'integer',
    'PositiveIntegerField': 'integer UNSIGNED',
    'PositiveSmallIntegerField': 'smallint UNSIGNED',
    'SlugField': 'varchar(%(max_length)s)',
    'SmallIntegerField': 'smallint',
    'TextField': 'longtext',
    'TimeField': 'time',
    'UUIDField': 'char(32)',
```

自定义一个char类型字段：
<td class="gutter">12345678910111213</td><td class="code">`class` `MyCharField(models.Field):``    ``"""``    ``自定义的char类型的字段类``    ``"""``    ``def` `__init__(``self``, max_length, ``*``args, ``*``*``kwargs):``        ``self``.max_length ``=` `max_length``        ``super``(MyCharField, ``self``).__init__(max_length``=``max_length, ``*``args, ``*``*``kwargs)` `    ``def` `db_type(``self``, connection):``        ``"""``        ``限定生成数据库表的字段类型为char，长度为max_length指定的值``        ``"""``        ``return` `'char(%s)'` `%` `self``.max_length`</td>
<td class="gutter">12345</td><td class="code">`class` `Class(models.Model):``    ``id` `=` `models.AutoField(primary_key``=``True``)``    ``title ``=` `models.CharField(max_length``=``25``)``    ``# 使用自定义的char类型的字段``    ``cname ``=` `MyCharField(max_length``=``25``)`</td>

创建的表结构：

<img src="https://images2017.cnblogs.com/blog/867021/201801/867021-20180119164437990-1369210170.png" alt="" />

### 字段参数

字段参数，详情可点击查看[官网](https://docs.djangoproject.com/en/1.11/ref/models/fields/#field-options)。
<td class="gutter">12345678910111213141516171819202122232425262728293031323334353637383940</td><td class="code">`    ``null                数据库中字段是否可以为空``    ``db_column           数据库中字段的列名``    ``default             数据库中字段的默认值``    ``primary_key         数据库中字段是否为主键``    ``db_index            数据库中字段是否可以建立索引``    ``unique              数据库中字段是否可以建立唯一索引``    ``unique_for_date     数据库中字段【日期】部分是否可以建立唯一索引``    ``unique_for_month    数据库中字段【月】部分是否可以建立唯一索引``    ``unique_for_year     数据库中字段【年】部分是否可以建立唯一索引` `    ``verbose_name        Admin中显示的字段名称``    ``blank               Admin中是否允许用户输入为空``    ``editable            Admin中是否可以编辑``    ``help_text           Admin中该字段的提示信息``    ``choices             Admin中显示选择框的内容，用不变动的数据放在内存中从而避免跨表操作``                        ``如：gf ``=` `models.IntegerField(choices``=``[(``0``, ``'何穗'``),(``1``, ``'大表姐'``),],default``=``1``)` `    ``error_messages      自定义错误信息（字典类型），从而定制想要显示的错误信息；``                        ``字典健：null, blank, invalid, invalid_choice, unique, ``and` `unique_for_date``                        ``如：{``'null'``: ``"不能为空."``, ``'invalid'``: ``'格式错误'``}` `    ``validators          自定义错误验证（列表类型），从而定制想要的验证规则``                        ``from` `django.core.validators ``import` `RegexValidator``                        ``from` `django.core.validators ``import` `EmailValidator,URLValidator,DecimalValidator,\``                        ``MaxLengthValidator,MinLengthValidator,MaxValueValidator,MinValueValidator``                        ``如：``                            ``test ``=` `models.CharField(``                                ``max_length``=``32``,``                                ``error_messages``=``{``                                    ``'c1'``: ``'优先错信息1'``,``                                    ``'c2'``: ``'优先错信息2'``,``                                    ``'c3'``: ``'优先错信息3'``,``                                ``},``                                ``validators``=``[``                                    ``RegexValidator(regex``=``'root_\d+'``, message``=``'错误了'``, code``=``'c1'``),``                                    ``RegexValidator(regex``=``'root_112233\d+'``, message``=``'又错误了'``, code``=``'c2'``),``                                    ``EmailValidator(message``=``'又错误了'``, code``=``'c3'``), ]``                            ``)` `字段参数`</td>

### Model Meta参数

这个不是很常用，如果你有特殊需要可以使用。详情点击查看[官网](https://docs.djangoproject.com/en/1.11/ref/models/options/#model-meta-options)。
<td class="gutter">123456789101112131415161718192021</td><td class="code">`class` `UserInfo(models.Model):``    ``nid ``=` `models.AutoField(primary_key``=``True``)``    ``username ``=` `models.CharField(max_length``=``32``)` `    ``class` `Meta:``        ``# 数据库中生成的表名称 默认 app名称 + 下划线 + 类名``        ``db_table ``=` `"table_name"` `        ``# admin中显示的表名称``        ``verbose_name ``=` `'个人信息'` `        ``# verbose_name加s``        ``verbose_name_plural ``=` `'所有用户信息'` `        ``# 联合索引  ``        ``index_together ``=` `[``            ``(``"pub_date"``, ``"deadline"``),   ``# 应为两个存在的字段``        ``]` `        ``# 联合唯一索引``        ``unique_together ``=` `((``"driver"``, ``"restaurant"``),)   ``# 应为两个存在的字段`</td>

### 多表关系和参数
<td class="gutter">123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108</td><td class="code">`ForeignKey(ForeignObject) ``# ForeignObject(RelatedField)``    ``to,                 ``# 要进行关联的表名``    ``to_field``=``None``,      ``# 要关联的表中的字段名称``    ``on_delete``=``None``,     ``# 当删除关联表中的数据时，当前表与其关联的行的行为``                        ``-` `models.CASCADE，删除关联数据，与之关联也删除``                        ``-` `models.DO_NOTHING，删除关联数据，引发错误IntegrityError``                        ``-` `models.PROTECT，删除关联数据，引发错误ProtectedError``                        ``-` `models.SET_NULL，删除关联数据，与之关联的值设置为null（前提FK字段需要设置为可空）``                        ``-` `models.SET_DEFAULT，删除关联数据，与之关联的值设置为默认值（前提FK字段需要设置默认值）``                        ``-` `models.``SET``，删除关联数据，``                               ``a. 与之关联的值设置为指定值，设置：models.``SET``(值)``                               ``b. 与之关联的值设置为可执行对象的返回值，设置：models.``SET``(可执行对象)` `                                    ``def` `func():``                                        ``return` `10` `                                    ``class` `MyModel(models.Model):``                                        ``user ``=` `models.ForeignKey(``                                            ``to``=``"User"``,``                                            ``to_field``=``"id"``                                            ``on_delete``=``models.``SET``(func),)``    ``related_name``=``None``,          ``# 反向操作时，使用的字段名，用于代替 【表名_set】 如： obj.表名_set.all()``    ``related_query_name``=``None``,    ``# 反向操作时，使用的连接前缀，用于替换【表名】     如： models.UserGroup.objects.filter(表名__字段名=1).values('表名__字段名')``    ``limit_choices_to``=``None``,      ``# 在Admin或ModelForm中显示关联数据时，提供的条件：``                                ``# 如：``                        ``-` `limit_choices_to``=``{``'nid__gt'``: ``5``}``                        ``-` `limit_choices_to``=``lambda` `: {``'nid__gt'``: ``5``}` `                        ``from` `django.db.models ``import` `Q``                        ``-` `limit_choices_to``=``Q(nid__gt``=``10``)``                        ``-` `limit_choices_to``=``Q(nid``=``8``) | Q(nid__gt``=``10``)``                        ``-` `limit_choices_to``=``lambda` `: Q(Q(nid``=``8``) | Q(nid__gt``=``10``)) & Q(caption``=``'root'``)``    ``db_constraint``=``True`          `# 是否在数据库中创建外键约束``    ``parent_link``=``False`           `# 在Admin中是否显示关联数据`  `OneToOneField(ForeignKey)``    ``to,                 ``# 要进行关联的表名``    ``to_field``=``None`       `# 要关联的表中的字段名称``    ``on_delete``=``None``,     ``# 当删除关联表中的数据时，当前表与其关联的行的行为` `                        ``###### 对于一对一 ######``                        ``# 1. 一对一其实就是 一对多 + 唯一索引``                        ``# 2.当两个类之间有继承关系时，默认会创建一个一对一字段``                        ``# 如下会在A表中额外增加一个c_ptr_id列且唯一：``                                ``class` `C(models.Model):``                                    ``nid ``=` `models.AutoField(primary_key``=``True``)``                                    ``part ``=` `models.CharField(max_length``=``12``)` `                                ``class` `A(C):``                                    ``id` `=` `models.AutoField(primary_key``=``True``)``                                    ``code ``=` `models.CharField(max_length``=``1``)` `ManyToManyField(RelatedField)``    ``to,                         ``# 要进行关联的表名``    ``related_name``=``None``,          ``# 反向操作时，使用的字段名，用于代替 【表名_set】 如： obj.表名_set.all()``    ``related_query_name``=``None``,    ``# 反向操作时，使用的连接前缀，用于替换【表名】     如： models.UserGroup.objects.filter(表名__字段名=1).values('表名__字段名')``    ``limit_choices_to``=``None``,      ``# 在Admin或ModelForm中显示关联数据时，提供的条件：``                                ``# 如：``                                    ``-` `limit_choices_to``=``{``'nid__gt'``: ``5``}``                                    ``-` `limit_choices_to``=``lambda` `: {``'nid__gt'``: ``5``}` `                                    ``from` `django.db.models ``import` `Q``                                    ``-` `limit_choices_to``=``Q(nid__gt``=``10``)``                                    ``-` `limit_choices_to``=``Q(nid``=``8``) | Q(nid__gt``=``10``)``                                    ``-` `limit_choices_to``=``lambda` `: Q(Q(nid``=``8``) | Q(nid__gt``=``10``)) & Q(caption``=``'root'``)``    ``symmetrical``=``None``,           ``# 仅用于多对多自关联时，symmetrical用于指定内部是否创建反向操作的字段``                                ``# 做如下操作时，不同的symmetrical会有不同的可选字段``                                    ``models.BB.objects.``filter``(...)` `                                    ``# 可选字段有：code, id, m1``                                        ``class` `BB(models.Model):` `                                        ``code ``=` `models.CharField(max_length``=``12``)``                                        ``m1 ``=` `models.ManyToManyField(``'self'``,symmetrical``=``True``)` `                                    ``# 可选字段有: bb, code, id, m1``                                        ``class` `BB(models.Model):` `                                        ``code ``=` `models.CharField(max_length``=``12``)``                                        ``m1 ``=` `models.ManyToManyField(``'self'``,symmetrical``=``False``)` `    ``through``=``None``,               ``# 自定义第三张表时，使用字段用于指定关系表``    ``through_fields``=``None``,        ``# 自定义第三张表时，使用字段用于指定关系表中那些字段做多对多关系表``                                    ``from` `django.db ``import` `models` `                                    ``class` `Person(models.Model):``                                        ``name ``=` `models.CharField(max_length``=``50``)` `                                    ``class` `Group(models.Model):``                                        ``name ``=` `models.CharField(max_length``=``128``)``                                        ``members ``=` `models.ManyToManyField(``                                            ``Person,``                                            ``through``=``'Membership'``,``                                            ``through_fields``=``(``'group'``, ``'person'``),``                                        ``)` `                                    ``class` `Membership(models.Model):``                                        ``group ``=` `models.ForeignKey(Group, on_delete``=``models.CASCADE)``                                        ``person ``=` `models.ForeignKey(Person, on_delete``=``models.CASCADE)``                                        ``inviter ``=` `models.ForeignKey(``                                            ``Person,``                                            ``on_delete``=``models.CASCADE,``                                            ``related_name``=``"membership_invites"``,``                                        ``)``                                        ``invite_reason ``=` `models.CharField(max_length``=``64``)``    ``db_constraint``=``True``,         ``# 是否在数据库中创建外键约束``    ``db_table``=``None``,              ``# 默认创建第三张表时，数据库中表的名称`</td>

### ORM操作

基本操作
<td class="gutter">12345678910111213141516171819202122</td><td class="code">`# 增``models.Tb1.objects.create(c1``=``'xx'``, c2``=``'oo'``)   ``# 增加一条数据，可以接受字典类型数据 **kwargs``obj ``=` `models.Tb1(c1``=``'xx'``, c2``=``'oo'``)``obj.save()`  `# 查``models.Tb1.objects.get(``id``=``123``)  ``# 获取单条数据，不存在则报错（不建议）``models.Tb1.objects.``all``()  ``# 获取全部``models.Tb1.objects.``filter``(name``=``'seven'``)  ``# 获取指定条件的数据``models.Tb1.objects.exclude(name``=``'seven'``)  ``# 去除指定条件的数据`  `# 删``# models.Tb1.objects.filter(name='seven').delete()  # 删除指定条件的数据`  `# 改``models.Tb1.objects.``filter``(name``=``'seven'``).update(gender``=``'0'``)   ``# 将指定条件的数据更新，均支持 **kwargs``obj ``=` `models.Tb1.objects.get(``id``=``1``)``obj.c1 ``=` `'111'``obj.save()   ``# 修改单条数据`</td>

进阶操作

```
# 获取个数
#
# models.Tb1.objects.filter(name='seven').count()

# 大于，小于
#
# models.Tb1.objects.filter(id__gt=1)              # 获取id大于1的值
# models.Tb1.objects.filter(id__gte=1)              # 获取id大于等于1的值
# models.Tb1.objects.filter(id__lt=10)             # 获取id小于10的值
# models.Tb1.objects.filter(id__lte=10)             # 获取id小于10的值
# models.Tb1.objects.filter(id__lt=10, id__gt=1)   # 获取id大于1 且 小于10的值

# 成员判断in
#
# models.Tb1.objects.filter(id__in=[11, 22, 33])   # 获取id等于11、22、33的数据
# models.Tb1.objects.exclude(id__in=[11, 22, 33])  # not in

# 是否为空 isnull
# Entry.objects.filter(pub_date__isnull=True)

# 包括contains
#
# models.Tb1.objects.filter(name__contains="ven")
# models.Tb1.objects.filter(name__icontains="ven") # icontains大小写不敏感
# models.Tb1.objects.exclude(name__icontains="ven")

# 范围range
#
# models.Tb1.objects.filter(id__range=[1, 2])   # 范围bettwen and

# 其他类似
#
# startswith，istartswith, endswith, iendswith,

# 排序order by
#
# models.Tb1.objects.filter(name='seven').order_by('id')    # asc
# models.Tb1.objects.filter(name='seven').order_by('-id')   # desc

# 分组group by
#
# from django.db.models import Count, Min, Max, Sum
# models.Tb1.objects.filter(c1=1).values('id').annotate(c=Count('num'))
# SELECT "app01_tb1"."id", COUNT("app01_tb1"."num") AS "c" FROM "app01_tb1" WHERE "app01_tb1"."c1" = 1 GROUP BY "app01_tb1"."id"

# limit 、offset
#
# models.Tb1.objects.all()[10:20]

# regex正则匹配，iregex 不区分大小写
#
# Entry.objects.get(title__regex=r'^(An?|The) +')
# Entry.objects.get(title__iregex=r'^(an?|the) +')

# date
#
# Entry.objects.filter(pub_date__date=datetime.date(2005, 1, 1))
# Entry.objects.filter(pub_date__date__gt=datetime.date(2005, 1, 1))

# year
#
# Entry.objects.filter(pub_date__year=2005)
# Entry.objects.filter(pub_date__year__gte=2005)

# month
#
# Entry.objects.filter(pub_date__month=12)
# Entry.objects.filter(pub_date__month__gte=6)

# day
#
# Entry.objects.filter(pub_date__day=3)
# Entry.objects.filter(pub_date__day__gte=3)

# week_day
#
# Entry.objects.filter(pub_date__week_day=2)
# Entry.objects.filter(pub_date__week_day__gte=2)

# hour
#
# Event.objects.filter(timestamp__hour=23)
# Event.objects.filter(time__hour=5)
# Event.objects.filter(timestamp__hour__gte=12)

# minute
#
# Event.objects.filter(timestamp__minute=29)
# Event.objects.filter(time__minute=46)
# Event.objects.filter(timestamp__minute__gte=29)

# second
#
# Event.objects.filter(timestamp__second=31)
# Event.objects.filter(time__second=2)
# Event.objects.filter(timestamp__second__gte=31)
```

高级操作

```
# extra
# 在QuerySet的基础上继续执行子语句
# extra(self, select=None, where=None, params=None, tables=None, order_by=None, select_params=None)

# select和select_params是一组，where和params是一组，tables用来设置from哪个表
# Entry.objects.extra(select={'new_id': "select col from sometable where othercol > %s"}, select_params=(1,))
# Entry.objects.extra(where=['headline=%s'], params=['Lennon'])
# Entry.objects.extra(where=["foo='a' OR bar = 'a'", "baz = 'a'"])
# Entry.objects.extra(select={'new_id': "select id from tb where id > %s"}, select_params=(1,), order_by=['-nid'])

举个例子：
models.UserInfo.objects.extra(
                    select={'newid':'select count(1) from app01_usertype where id>%s'},
                    select_params=[1,],
                    where = ['age>%s'],
                    params=[18,],
                    order_by=['-age'],
                    tables=['app01_usertype']
                )
                """
                select 
                    app01_userinfo.id,
                    (select count(1) from app01_usertype where id>1) as newid
                from app01_userinfo,app01_usertype
                where 
                    app01_userinfo.age > 18
                order by 
                    app01_userinfo.age desc
                """


# 执行原生SQL
# 更高灵活度的方式执行原生SQL语句
# from django.db import connection, connections
# cursor = connection.cursor()  # cursor = connections['default'].cursor()
# cursor.execute("""SELECT * from auth_user where id = %s""", [1])
# row = cursor.fetchone()
```

QuerySet相关方法

```
##################################################################
# PUBLIC METHODS THAT ALTER ATTRIBUTES AND RETURN A NEW QUERYSET #
##################################################################

def all(self)
    # 获取所有的数据对象

def filter(self, *args, **kwargs)
    # 条件查询
    # 条件可以是：参数，字典，Q

def exclude(self, *args, **kwargs)
    # 条件查询
    # 条件可以是：参数，字典，Q

def select_related(self, *fields)
    性能相关：表之间进行join连表操作，一次性获取关联的数据。

    总结：
    1. select_related主要针一对一和多对一关系进行优化。
    2. select_related使用SQL的JOIN语句进行优化，通过减少SQL查询的次数来进行优化、提高性能。

def prefetch_related(self, *lookups)
    性能相关：多表连表操作时速度会慢，使用其执行多次SQL查询在Python代码中实现连表操作。

    总结：
    1. 对于多对多字段（ManyToManyField）和一对多字段，可以使用prefetch_related()来进行优化。
    2. prefetch_related()的优化方式是分别查询每个表，然后用Python处理他们之间的关系。

def annotate(self, *args, **kwargs)
    # 用于实现聚合group by查询

    from django.db.models import Count, Avg, Max, Min, Sum

    v = models.UserInfo.objects.values('u_id').annotate(uid=Count('u_id'))
    # SELECT u_id, COUNT(ui) AS `uid` FROM UserInfo GROUP BY u_id

    v = models.UserInfo.objects.values('u_id').annotate(uid=Count('u_id')).filter(uid__gt=1)
    # SELECT u_id, COUNT(ui_id) AS `uid` FROM UserInfo GROUP BY u_id having count(u_id) > 1

    v = models.UserInfo.objects.values('u_id').annotate(uid=Count('u_id',distinct=True)).filter(uid__gt=1)
    # SELECT u_id, COUNT( DISTINCT ui_id) AS `uid` FROM UserInfo GROUP BY u_id having count(u_id) > 1

def distinct(self, *field_names)
    # 用于distinct去重
    models.UserInfo.objects.values('nid').distinct()
    # select distinct nid from userinfo

    注：只有在PostgreSQL中才能使用distinct进行去重

def order_by(self, *field_names)
    # 用于排序
    models.UserInfo.objects.all().order_by('-id','age')

def extra(self, select=None, where=None, params=None, tables=None, order_by=None, select_params=None)
    # 构造额外的查询条件或者映射，如：子查询

    Entry.objects.extra(select={'new_id': "select col from sometable where othercol > %s"}, select_params=(1,))
    Entry.objects.extra(where=['headline=%s'], params=['Lennon'])
    Entry.objects.extra(where=["foo='a' OR bar = 'a'", "baz = 'a'"])
    Entry.objects.extra(select={'new_id': "select id from tb where id > %s"}, select_params=(1,), order_by=['-nid'])

 def reverse(self):
    # 倒序
    models.UserInfo.objects.all().order_by('-nid').reverse()
    # 注：如果存在order_by，reverse则是倒序，如果多个排序则一一倒序


 def defer(self, *fields):
    models.UserInfo.objects.defer('username','id')
    或
    models.UserInfo.objects.filter(...).defer('username','id')
    #映射中排除某列数据

 def only(self, *fields):
    #仅取某个表中的数据
     models.UserInfo.objects.only('username','id')
     或
     models.UserInfo.objects.filter(...).only('username','id')

 def using(self, alias):
     指定使用的数据库，参数为别名（setting中的设置）


##################################################
# PUBLIC METHODS THAT RETURN A QUERYSET SUBCLASS #
##################################################

def raw(self, raw_query, params=None, translations=None, using=None):
    # 执行原生SQL
    models.UserInfo.objects.raw('select * from userinfo')

    # 如果SQL是其他表时，必须将名字设置为当前UserInfo对象的主键列名
    models.UserInfo.objects.raw('select id as nid from 其他表')

    # 为原生SQL设置参数
    models.UserInfo.objects.raw('select id as nid from userinfo where nid>%s', params=[12,])

    # 将获取的到列名转换为指定列名
    name_map = {'first': 'first_name', 'last': 'last_name', 'bd': 'birth_date', 'pk': 'id'}
    Person.objects.raw('SELECT * FROM some_other_table', translations=name_map)

    # 指定数据库
    models.UserInfo.objects.raw('select * from userinfo', using="default")

    ################### 原生SQL ###################
    from django.db import connection, connections
    cursor = connection.cursor()  # cursor = connections['default'].cursor()
    cursor.execute("""SELECT * from auth_user where id = %s""", [1])
    row = cursor.fetchone() # fetchall()/fetchmany(..)


def values(self, *fields):
    # 获取每行数据为字典格式

def values_list(self, *fields, **kwargs):
    # 获取每行数据为元祖

def dates(self, field_name, kind, order='ASC'):
    # 根据时间进行某一部分进行去重查找并截取指定内容
    # kind只能是："year"（年）, "month"（年-月）, "day"（年-月-日）
    # order只能是："ASC"  "DESC"
    # 并获取转换后的时间
        - year : 年-01-01
        - month: 年-月-01
        - day  : 年-月-日

    models.DatePlus.objects.dates('ctime','day','DESC')

def datetimes(self, field_name, kind, order='ASC', tzinfo=None):
    # 根据时间进行某一部分进行去重查找并截取指定内容，将时间转换为指定时区时间
    # kind只能是 "year", "month", "day", "hour", "minute", "second"
    # order只能是："ASC"  "DESC"
    # tzinfo时区对象
    models.DDD.objects.datetimes('ctime','hour',tzinfo=pytz.UTC)
    models.DDD.objects.datetimes('ctime','hour',tzinfo=pytz.timezone('Asia/Shanghai'))

    """
    pip3 install pytz
    import pytz
    pytz.all_timezones
    pytz.timezone(Asia/Shanghai)
    """

def none(self):
    # 空QuerySet对象


####################################
# METHODS THAT DO DATABASE QUERIES #
####################################

def aggregate(self, *args, **kwargs):
   # 聚合函数，获取字典类型聚合结果
   from django.db.models import Count, Avg, Max, Min, Sum
   result = models.UserInfo.objects.aggregate(k=Count('u_id', distinct=True), n=Count('nid'))
   ===> {'k': 3, 'n': 4}

def count(self):
   # 获取个数

def get(self, *args, **kwargs):
   # 获取单个对象

def create(self, **kwargs):
   # 创建对象

def bulk_create(self, objs, batch_size=None):
    # 批量插入
    # batch_size表示一次插入的个数
    objs = [
        models.DDD(name='r11'),
        models.DDD(name='r22')
    ]
    models.DDD.objects.bulk_create(objs, 10)

def get_or_create(self, defaults=None, **kwargs):
    # 如果存在，则获取，否则，创建
    # defaults 指定创建时，其他字段的值
    obj, created = models.UserInfo.objects.get_or_create(username='root1', defaults={'email': '1111111','u_id': 2, 't_id': 2})

def update_or_create(self, defaults=None, **kwargs):
    # 如果存在，则更新，否则，创建
    # defaults 指定创建时或更新时的其他字段
    obj, created = models.UserInfo.objects.update_or_create(username='root1', defaults={'email': '1111111','u_id': 2, 't_id': 1})

def first(self):
   # 获取第一个

def last(self):
   # 获取最后一个

def in_bulk(self, id_list=None):
   # 根据主键ID进行查找
   id_list = [11,21,31]
   models.DDD.objects.in_bulk(id_list)

def delete(self):
   # 删除

def update(self, **kwargs):
    # 更新

def exists(self):
   # 是否有结果

其他操作
```
