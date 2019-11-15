---
layout:     post
title:      ubuntuinstallredismongo以及监控安装
subtitle:   
date:       2019-05-30
author:     P
header-img: img/post-bg-mma-1.jpg
catalog: true
tags:
    - python
---
```
sudo apt-get updatesudo apt-get install redis-server mongodb
```

```
sudo apt-get install htopsudo apt-get install iotop
```

```
sudo redis-server redis.conf<指定配置文件启动><strong>sudo service redis-server stop
</strong>
```

**`sudo servcie redis-server start`**

**`sudo service redis-server restart`**

 

**redis error: **

```
redis.exceptions.ResponseError: MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.
```

**解决：**

**　　config set stop-writes-on-bgsave-error no ;    config set stop-writes-on-bgsave-error no**

```

```

#### mongo启动、重启和关闭命令

<li>

sudo service mongod start

</li>
<li>

sudo service mongod restart

</li>
<li>

sudo service mongod stop

</li>

```
配置文件后台启动Mongodbnohup sudo mongod --config /etc/mongodb.conf &更改mongodb --path 后可以用一下命令解决 权限问题sudo chown -Rh mongodb:mongodb /renpeng/mongodb*sudo chown -R mongodb:mongodb /renpeng/mongodb/*# chown [-R] [用户名称:组名称] [文件或目录]-c或-change：作用与-v相似，但只传回修改的部分  -f或quiet或silent：不显示错误信息  -h或no-dereference：只对符号链接的文件做修改，而不更改其他任何相关文件 -R或-recursive：递归处理，将指定目录下的所有文件及子目录一并处理  -v或verbose：显示指令执行过程  dereference：作用和-h刚好相反  help：显示在线说明  reference=<参考文件或目录>：把指定文件或目录的所有者与所属组，统统设置成和参考文件或目录的所有者与所属组相同  version：显示版本信息关闭Mongodb.shutdownServer()
```
