---
layout:     post
title:      Djangocontenttypes组件
subtitle:   
date:       2019-01-22
author:     P
header-img: img/post-bg-js-version.jpg
catalog: true
tags:
    - python
---
# contenttypes组件

## 介绍

## 应用场景

我们在网上po一段散文诗也可以po一张旅途的风景图，文字可以被评论，图片也可以被评论。我们需要在数据库中建表存储这些数据，我们可能会设计出下面这样的表结构。

```
class Post(models.Model):
    """帖子表"""
    author = models.ForeignKey(User)
    title = models.CharField(max_length=72)


class Picture(models.Model):
    """图片表"""
    author = models.ForeignKey(User)
    image = models.ImageField()


class Comment(models.Model):
    """评论表"""
    author = models.ForeignKey(User)
    content = models.TextField()
    post = models.ForeignKey(Post, null=True, blank=True, on_delete=models.CASCADE)
    picture = models.ForeignKey(Picture, null=True, blank=True, on_delete=models.CASCADE)
```

这表结构看起来不太简洁，我们画个图来看一下：

<img src="https://img2018.cnblogs.com/blog/867021/201901/867021-20190116163037236-283539817.png" alt="" />

能用是能用，但是评论表有点冗余啊。好多列都空着呢啊！

我们优化一下，我们在评论表里不直接外键关联 文字和图片，而是存储一下关联的表名和字段，这样就好很多了。

看下图：

<img src="https://img2018.cnblogs.com/blog/867021/201901/867021-20190116163437005-317475046.png" alt="" />

那我们不妨步子再大一点，再往前走一步试试，因为表名在评论里面重复了很多次，我们完全可以把Django项目中的表名都存储在一个表里面。然后评论表里外键关联这个表就可以了。

<img src="https://img2018.cnblogs.com/blog/867021/201901/867021-20190116164158244-1452668666.png" alt="" />

```
class Comment(models.Model):
    """评论表"""
    author = models.ForeignKey(User)
    content = models.TextField()

    content_type = models.ForeignKey(ContentType)  # 外键关联django_content_type表
    object_id = models.PositiveIntegerField()  # 关联数据的主键
    content_object = GenericForeignKey('content_type', 'object_id')
```

## contenttypes使用

```
import os

if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "about_contenttype.settings")

    import django
    django.setup()

    from app01.models import Post, Picture, Comment
    from django.contrib.auth.models import User
    # 准备测试数据
    user_1 = User.objects.create_user(username='aaa', password='123')
    user_2 = User.objects.create_user(username='bbb', password='123')
    user_3 = User.objects.create_user(username='ccc', password='123')

    post_1 = Post.objects.create(author=user_1, title='Python入门教程')
    post_2 = Post.objects.create(author=user_2, title='Python进阶教程')
    post_3 = Post.objects.create(author=user_1, title='Python入土教程')

    picture_1 = Picture.objects.create(author=user_1, image='小姐姐01.jpg')
    picture_2 = Picture.objects.create(author=user_1, image='小姐姐02.jpg')
    picture_3 = Picture.objects.create(author=user_3, image='小哥哥01.jpg')

    # 给帖子创建评论数据
    comment_1 = Comment.objects.create(author=user_1, content='好文！', content_object=post_1)
    # 给图片创建评论数据
    comment_2 = Comment.objects.create(author=user_2, content='好美！', content_object=picture_1)
```

```
from django.contrib.contenttypes.fields import GenericRelation
```

```
class Post(models.Model):
    """帖子表"""
    author = models.ForeignKey(User)
    title = models.CharField(max_length=72)

    comments = GenericRelation('Comment')  # 支持反向查找评论数据（不会在数据库中创建字段）


class Picture(models.Model):
    """图片表"""
    author = models.ForeignKey(User)
    image = models.ImageField()

    comments = GenericRelation('Comment')  # 支持反向查找评论数据（不会在数据库中创建字段）
```

```
class Comment(models.Model):    """评论"""    info = models.TextField()    # talk = models.ForeignKey(to='Talk', null=True, on_delete=models.CASCADE)    # picture = models.ForeignKey(to='Picture', null=True, on_delete=models.CASCADE)    # 关联表的表名（外键 --> Django Content Type表）    table_name = models.ForeignKey(to=ContentType)    # 关联表中具体的数据id    obj_id = models.IntegerField()    # 告诉Django table_name和obj_id是一组动态关联的数据    content_object = GenericForeignKey('table_name', 'obj_id')
```

```
 
```

查询示例：

```
post_1 = Post.objects.filter(id=1).first()
comment_list = post_1.comments.all()
```
