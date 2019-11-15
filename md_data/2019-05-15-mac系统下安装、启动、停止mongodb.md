---
layout:     post
title:      mac系统下安装、启动、停止mongodb
subtitle:   
date:       2019-05-15
author:     P
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - python
---
**<strong><strong>mongodb是非关系型数据库，mysquel是关系型数据库，前者没有数据表这个说法，后者有**</strong></strong>

**一.nosqlbooster下载地址：**

[https://nosqlbooster.com/downloads](https://nosqlbooster.com/downloads)

**<strong>二. 本文主要讲解，安装包方式安装 mongodb，至于其他方式不做介绍。**</strong>

###### 下载Mongodb后，将Mongodb-3.2.5.tar.gz 复制到 /leleda002 路径下解压得到mongodb这个文件夹，（下图中的是我自己改了名字删掉了版本号）

<img src="https://images2017.cnblogs.com/blog/1086124/201801/1086124-20180106122149674-83721915.png" alt="" width="436" height="245" />

刚下载打开的文件是没有 data、etc、以及log文件夹的。只有一个bin 文件夹。

**三、文件建立。**

然后在根目录下新建 data 文件夹，里面再建一个db文件夹，就是上图中那个 usr文件夹上面的 那个data文件夹 ，里面是用来存放[数据库](https://link.jianshu.com/?t=http://lib.csdn.net/base/mysql)的。

新建一个etc文件夹，用来放文件配置。

**data/db和于存放数据文件，etc用于存放mongod.conf，log用于存放mongod.logs 错误日志。**

mongod.conf 内容如下

```
#mongodb config file
dbpath=/Users/wangxi/Documents/mongodb/data/db/
logpath=/Users/wangxi/Documents/mongodb/mongod.log
logappend = true
port = 27017
fork = true
auth = true
```

这个主要是用来配置数据库位置，和错误输出的文件位置。

**四、修改系统环境变量PATH**

把 /Users/wangxi/Documents/develop/mongodb/bin 目录加到PATH中。

（其实就是把mongodb/bin这个地址加一个快捷启动目录，找到当目录的方法，在控制台进入到该目录下，执行 pwd 便可以得到该目录）

修改环境变量的方法比较多，这里采用如下方式：

首先添加PATH：

```
echo 'export PATH=/Users/wangxi/Documents/develop/mongodb/bin:$PATH'>>~/.bash_profile 
```

如下

<img src="https://images2017.cnblogs.com/blog/1086124/201801/1086124-20180106124437503-998942444.png" alt="" />

添加完成后为使环境变量生效，可重启shell终端

或输入命令 source .bash_profile。

查看环境变量是否添加成功：

```
echo $PATH
```

如下：

<img src="https://images2017.cnblogs.com/blog/1086124/201801/1086124-20180106124545784-1072828146.png" alt="" />

环境变量添加成功。

4.5、为数据库日志文件添加操作权限。

　　新建立的data/db 通过查看是否与读写权限，如果没有的话需要添加读写权限

```
sudo chown -R  用户名 /data/db
```

<img src="https://images2017.cnblogs.com/blog/1086124/201801/1086124-20180106131924409-2045152874.png" alt="" />

**　　如何检测安装成功了呢：在控制台输入**

```
which mongod
```

会出现一个路径就代表安装成功了

**五、启动mongodb**

cmd+T 新建命令窗口，进入mongodb 的 "bin"目录，使用命令./mongod 或 mongod 启动mongoDB server，启动成功后最后一行应该是端口号，如下：

这一步是连接Mongodb的服务的

<img src="https://images2017.cnblogs.com/blog/1086124/201801/1086124-20180106124739424-739095418.png" alt="" />

###### 打开浏览器，输入localhost:27017,会出现

It looks like you are trying to access MongoDB over HTTP on the native driver port. 这样一行文字，然后可以重新打开一个终端 同样是。

5.5、新建窗口，输入 ./mongo 或 mongo , 尝试操作数据库：这个步骤是操作数据库了。不需要重新进入bin目录，新建窗口直接执行命令便可以

<img src="https://images2017.cnblogs.com/blog/1086124/201801/1086124-20180106124817737-1252581456.png" alt="" />

###### **六.要停止mongodb一定要正确的退出,不然下次再次连接数据库会出现问题.**

```
use admin;
db.shutdownServer();
```

　　**备注：如果安装成功后，以后只需要启动MongoDB服务，然后金操作数据库就行了。就相当于只需要执行上边的 5 和 6 步骤就可以了。**

**　　以上前4步骤是安装，56是连接服务器，启动数据库。**

在连接服务执行 ./mongod 或 mongod 经常会出现一些问题，接下来将本人遇到的问题在下边做一整理。

一、启动[Mac下安装mongoldb 报错 shutting down with code:100](http://blog.csdn.net/u013939918/article/details/78200946)。

```
具体错误栈：

2017-10-11T09:31:12.140+0800 I CONTROL  [initandlisten] MongoDB starting : pid=2382 port=27017 dbpath=/data/db 64-bit host=songyuxiangdeMacBook-Pro.local

2017-10-11T09:31:12.141+0800 I CONTROL  [initandlisten] db version v3.4.9
2017-10-11T09:31:12.141+0800 I CONTROL  [initandlisten] git version: 876ebee8c7dd0e2d992f36a848ff4dc50ee6603e
2017-10-11T09:31:12.141+0800 I CONTROL  [initandlisten] OpenSSL version: OpenSSL 0.9.8zh 14 Jan 2016
2017-10-11T09:31:12.141+0800 I CONTROL  [initandlisten] allocator: system
2017-10-11T09:31:12.141+0800 I CONTROL  [initandlisten] modules: none
2017-10-11T09:31:12.141+0800 I CONTROL  [initandlisten] build environment:
2017-10-11T09:31:12.141+0800 I CONTROL  [initandlisten]     distarch: x86_64
2017-10-11T09:31:12.141+0800 I CONTROL  [initandlisten]     target_arch: x86_64
2017-10-11T09:31:12.141+0800 I CONTROL  [initandlisten] options: {}
2017-10-11T09:31:12.141+0800 I STORAGE  [initandlisten] exception in initAndListen: 29 Data directory /data/db not found., terminating
2017-10-11T09:31:12.141+0800 I NETWORK  [initandlisten] shutdown: going to close listening sockets...
2017-10-11T09:31:12.141+0800 I NETWORK  [initandlisten] shutdown: going to flush diaglog...
2017-10-11T09:31:12.141+0800 I CONTROL  [initandlisten] now exiting
2017-10-11T09:31:12.141+0800 I CONTROL  [initandlisten] shutting down with code:100
```

这个是目录指定的问题。

参考我的启动命令。 

```
./mongod --dbpath ../data/db/

启动mongodb的shell客户端(command + T)

./mongo
```

有的时候按照上边的步骤执行还是报错 100，这个时候看看data/db下边是不是有一个 mongod.lock 文件，这个代表上次退出不是正常退出导致文件被锁住了，所以不能正常启动。

二、上边步骤4 环境变量配置步骤。

如果环境变量的配置出现错误，也可以理解为 路径的指定有误了，这个时候想要修改或者删除

环境变量的配置可以理解为他是将变量写在了一个文件里面

```
 vi ~/**.bash_profile**
```

```
~/.bash_profile 
这个就是环境变量的文件地址（可以这样理解）
 vi ~/.bash_profile
利用 vi 查看这个文件，也就是在终端查看这个文件
如果找不到没有权限
sudo vi ~/.bash_profile
就可以看到相应的配置
修改：
vi ~/.bash_profile
dd  要删除的代码，将光标放到要删除的那行双击dd
:wq  保存文件并推出
source ~/.bash_profile或者关闭重启shell
```

改完之后输出一下，便可以看到是否更改了。
