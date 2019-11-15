---
layout:     post
title:      Linux后台运行python.sh等程序，以及查看和关闭后台运行程序操作
subtitle:   
date:       2019-06-03
author:     P
header-img: img/post-bg-miui6.jpg
catalog: true
tags:
    - python
---
1、运行.sh文件

直接用./sh 文件就可以运行，但是如果想后台运行，即使关闭当前的终端也可以运行的话，需要nohup命令和&命令。

（1）&命令

{% raw %}
```
`      功能：加在一个命令的最后，可以把这个命令放在后台执行`
```
{% endraw %}

（2）nohup命令

{% raw %}
```
`      功能：不挂断的运行命令`
```
{% endraw %}

2、查看当前后台运行的命令

有两个命令可以用，jobs和ps,区别是jobs用于查看当前终端后台运行的任务，换了终端就看不到了。而ps命令用于查看瞬间进程的动态，可以看到别的终端运行的后台进程。

（1）jobs命令

{% raw %}
```
<code class="has-numbering" onclick="mdcp.signin(event)">    功能：查看当前终端后台运行的任务

   

   jobs -l选项可显示当前终端所有任务的PID，jobs的状态可以是running，stopped，Terminated。+ 号表示当前任务，- 号表示后一个任务。</code>
```
{% endraw %}

 

（2）ps命令

{% raw %}
```
<code class="has-numbering" onclick="mdcp.signin(event)">      功能：查看当前的所有进程

      

     ps -aux | grep "test.sh"    #a:显示所有程序  u:以用户为主的格式来显示   x:显示所有程序，不以终端机来区分</code>
```
{% endraw %}

 

3、关闭当前后台运行的命令

{% raw %}
```
<code class="has-numbering" onclick="mdcp.signin(event)">  kill命令：结束进程

 （1）通过jobs命令查看jobnum，然后执行   kill %jobnum

 （2）通过ps命令查看进程号PID，然后执行  kill %PID

   如果是前台进程的话，直接执行 Ctrl+c 就可以终止了</code>
```
{% endraw %}

 

4、前后台进程的切换与控制

{% raw %}
```
<code class="has-numbering" onclick="mdcp.signin(event)"> （1）fg命令

   功能：将后台中的命令调至前台继续运行

   如果后台中有多个命令，可以先用jobs查看jobnun，然后用 fg %jobnum 将选中的命令调出。

 （2）Ctrl + z 命令

   功能：将一个正在前台执行的命令放到后台，并且处于暂停状态

 （3）bg命令

   功能：将一个在后台暂停的命令，变成在后台继续执行

   如果后台中有多个命令，可以先用jobs查看jobnum，然后用 bg %jobnum 将选中的命令调出继续执行。</code>
```
{% endraw %}

 

5、运行Python示例

{% raw %}
```
<code class="has-numbering" onclick="mdcp.signin(event)">[root@xxxxx nlp]# nohup python -u jbabs.py > out.log 2>&1 &
[1] 20087
[root@xxxxx nlp]# jobs
[1]+  Running                 nohup python -u jbabs.py > out.log 2>&1 &</code>
```
{% endraw %}

 

<li>
运行一个Python脚本，通常设置如下
$ python /data/python/server.py >python.log &
</li>

说明：     1、 > 表示把标准输出（STDOUT）重定向到 那个文件，这里重定向到了python.log     2、 & 表示在后台执行脚本这样可以到达目的，但是，我们退出shell窗口的时候，必须用exit命令来退出，否则，退出之后，该进程也会随着shell的消失而消失（退出、关闭）

<li>使用nohup(not hang up)：
$ nohup python /data/python/server.py > python.log3 2>&1 &</li>
- 说明：






1、1是标准输出（STDOUT）的文件描述符，2是标准错误（STDERR）的文件描述符     1> python.log 简化为 > python.log，表示把标准输出重定向到python.log这个文件

2、2>&1 表示把标准错误重定向到标准输出，这里&1表示标准输出 ，  为什么需要将标准错误重定向到标准输出的原因，是因为标准错误没有缓冲区，而STDOUT有。 这就会导致  commond > python.log  ，2> python.log 文件python.log被两次打开，而STDOUT和  STDERR将会竞争覆盖，这肯定不是我门想要的
3、好了，我们现在可以直接关闭shell窗口（我用的是SecureCRT，用的比较多的还有Xshell），而不用再输入exit这个命令来退出shell了

$ ps aux|grep pythontomener 1885  0.1  0.4  13120  4528 pts/0    S    15:48   0:00 python /data/python/server.pytomener 1887  0.0  0.0   5980   752 pts/0    S+   15:48   0:00 grep python

在当我们直接关闭shell窗口，再连接上服务器，查看Python的进程，发现进程还在

但是，在python运行中却查看不到输出！

因为：

使用-u参数，使得python不启用缓冲。

**kill -STOP 1234    将该进程暂停。**

**如果要让它恢复到后台，用 kill -CONT 1234 （很多在前台运行的程序这样是不行的）**

如果要恢复到前台，请在当时运行该进程的那个终端用jobs命令查询暂停的进程。

然后用 fg 〔job号〕把进程恢复到前台。

**所以改正命令，就可以正常使用了**

{% raw %}
```
`$ nohup python3 -u test.py > out.log 2>&1 &在当前shell 里查看后台任务：jobs把后台的任务拉到前台运行：　　fg %任务号 默认任务1使用 crl + z 是任务暂停并挂载到后台使用 bg %任务号 使任务在后台运行`
```
{% endraw %}
