---
layout:     post
title:      github与gitlab与git三个基佬的故事
subtitle:   
date:       2019-04-16
author:     P
header-img: img/post-bg-mma-5.jpg
catalog: true
tags:
    - python
---
<img src="https://img2018.cnblogs.com/blog/1132884/201812/1132884-20181223171046581-1823566881.png" alt="" width="550" height="314" /><img src="https://img2018.cnblogs.com/blog/1132884/201812/1132884-20181223171248364-1685454853.png" alt="" width="471" height="314" />

**我们了解了git是以个人为中心，但是人人都得数据交互呀。。python程序员每天都忙着进行py交易**

交互数据的方式

- 使用github或者码云等公有代码仓库，托管代码的地方，谁都可以看
- 公司内部使用gitlab私有仓库

github和gitlab的区别

- github国外公共仓库不安全，国内的码云代码仓库，可能会暴露自己公司代码机密，等着被开除吧。。
- 自建gitlab私有代码仓库，更加安装

<!--?xml version="1.0" encoding="UTF-8"?-->

#  安装配置gitlab 

安装gitlab的命令

```
我们是要在centos7上安装配置gitlab
建议库容服务器配置，gitlab占用资源很多，最少4G内存虚拟机

通过清华源配置gitlab，加速下载
清华大学开源镜像站
https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/
配置步骤
touch /etc/yum.repos.d/gitlab-ce.repo
下入如下内容
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
生成yum源缓存

安装gitlab-ce
sudo yum makecache
sudo yum install gitlab-ce 
gitlab-ctl reconfigure 初始化gitlab，只能执行一次
gitlab-ctl status/stop/start    启动gitlab
gitlab-ctl status
通过浏览器访问页面服务器ip，默认开启了nginx的web端口，设置初始密码，操作类似github
第一次访问会设置新密码 redhat123
登录root
密码redhat123
即可看到gitlab 
```

安装访问gitlab可能出现的问题

```
如果初始化报错，有关编码问题，修改字符编码
解决：在 ~/.bash_profile， 然后source ~/.bash_profile
export LC_ALL="zh_CN.UTF-8"
export LC_CTYPE="zh_CN.UTF-8"
```

检查gitlab安装

```
gitlab-ce一键安装后可以利用rpm -ql gitlab-ce查询其文件安装路径及相关文件路径，其默认安装路径为/opt/gitlab/、程序数据及配置文件保存路径为/var/opt/gitlab下。
相关默认位置

代码仓库保存位置：/var/opt/gitlab/git-data/repositories/
代码仓库备份位置：/var/opt/gitlab/backups/
postgresql数据及配置目录：/var/opt/gitlab/postgresql/data/
redis默认配置目录：/var/opt/gitlab/redis
gitlab主要配置文件：/etc/gitlab/gitlab.rb
```

配置gitlab服务器，便于外接访问

```
编辑/etc/gitlab/gitlab.rb
修改gitlab运行外部URL默认的访问地址

# 未修gitlab.rb配置文件中nginx配置时这个配置默认配置gitlab自带的nginx端口可以通过修改如下参数，也就访问的gitlab地址
external_url 'http://172.17.17.10:81'  

2.通过官网手册安装gitlab
https://about.gitlab.com/install/#centos-7
```

在linux服务器上配置ssh秘钥

```
ssh-keygen    一路回车
查看公钥文件，放到gitlab
cat /root/.ssh/id_rsa.pub
```

<img src="https://img2018.cnblogs.com/blog/1132884/201812/1132884-20181223182104610-203783050.png" alt="" width="698" height="391" />

-

<img src="https://img2018.cnblogs.com/blog/1132884/201812/1132884-20181223182259946-1190517669.png" alt="" width="702" height="437" />

-

<img src="https://img2018.cnblogs.com/blog/1132884/201812/1132884-20181223182444445-2029841108.png" alt="" width="699" height="396" />

-

<img src="https://img2018.cnblogs.com/blog/1132884/201812/1132884-20181223182550940-410550497.png" alt="" width="664" height="194" />

-

<img src="https://img2018.cnblogs.com/blog/1132884/201812/1132884-20181223182610252-1972117530.png" alt="" width="663" height="453" />

#  gitlab代码下载/推送实战

```
创建新的仓库，下载gitlab仓库
git clone git@192.168.119.12:root/oldboypython.git        克隆下载远端仓库
cd oldboypython    进入仓库文件夹
touch README.md    新建一个测试文件
git add README.md    提交到暂存区
git commit -m "add README    提交暂存区文件到本地仓库
git push -u origin master    推送到远端master主干仓库    origin是远程仓库地址

也可以在远端gitlab web界面修改代码，提交后，在本地pull新代码
(在git仓库中直接)    git pull

git remote show origin 查看远程服务器信息  orgin是在创建仓库时定义在.git/config配置文件中的
```
