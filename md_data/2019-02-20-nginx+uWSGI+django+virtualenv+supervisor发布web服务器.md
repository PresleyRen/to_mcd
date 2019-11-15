---
layout:     post
title:      nginx+uWSGI+django+virtualenv+supervisor发布web服务器
subtitle:   
date:       2019-02-20
author:     P
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - python
---
#  导论

{% raw %}
```
WSGI是Web服务器网关接口。它是一个规范，描述了Web服务器如何与Web应用程序通信，以及Web应用程序如何链接在一起以处理一个请求，（接收请求，处理请求，响应请求）
基于wsgi运行的框架有bottle,DJango,Flask,用于解析动态HTTP请求
支持WSGI的服务器
    wsgiref
        python自带的web服务器
    Gunicorn
        用于linux的 python wsgi Http服务器，常用于各种django，flask结合部署服务器。
    mode_wsgi
        实现了Apache与wsgi应用程序的结合
    uWSGI
        C语言开发，快速，自我修复，开发人员友好的WSGI服务器，用于Python Web应用程序的专业部署和开发。

在部署python程序web应用程序时，可以根据性能的需求，选择合适的wsgi server，不同的wsgi server区别在于并发支持上，有单线程，多进程，多线程，协程的区别，其功能还是近似，无非是请求路由，执行对应的函数，返回处理结果。




Django部署

Django的主要部署平台是 WSGI，这是用于Web服务器和应用程序的Python标准。

Django的 startproject管理命令设置一个简单的默认WSGI配置，可以根据需要为您的项目进行调整，并指示任何符合WSGI的应用程序服务器使用。

application 
使用WSGI部署的关键概念是应用程序服务器用于与代码通信的 application 可调用。它通常在服务器可访问的Python模块中作为名为 application 的对象提供。

startproject 命令创建包含这样的 application 可调用的文件 <project_name>/wsgi.py. ，它被Django的开发服务器和生产WSGI部署使用。
WSGI服务器从其配置中获取 application 可调用的路径。 Django的内置服务器，即 runserver 命令，从 WSGI_APPLICATION 设置读取它。
```
{% endraw %}

{% raw %}
```
1 首先nginx 是对外的服务接口，外部浏览器通过url访问nginx,

2nginx 接收到浏览器发送过来的http请求，将包进行解析，分析url，如果是静态文件请求就直接访问用户给nginx配置的静态文件目录，直接返回用户请求的静态文件，

如果不是静态文件，而是一个动态的请求，那么nginx就将请求转发给uwsgi,uwsgi 接收到请求之后将包进行处理，处理成wsgi可以接受的格式，并发给wsgi,wsgi 根据请求调用应用程序的某个文件，某个文件的某个函数，最后处理完将返回值再次交给wsgi,wsgi将返回值进行打包，打包成uwsgi能够接收的格式，uwsgi接收wsgi 发送的请求，并转发给nginx,nginx最终将返回值返回给浏览器。

3要知道第一级的nginx并不是必须的，uwsgi完全可以完成整个的和浏览器交互的流程，但是要考虑到某些情况

1 安全问题，程序不能直接被浏览器访问到，而是通过nginx,nginx只开放某个接口，uwsgi本身是内网接口，这样运维人员在nginx上加上安全性的限制，可以达到保护程序的作用。

2负载均衡问题，一个uwsgi很可能不够用，即使开了多个work也是不行，毕竟一台机器的cpu和内存都是有限的，有了nginx做代理，一个nginx可以代理多台uwsgi完成uwsgi的负载均衡。

3静态文件问题，用django或是uwsgi这种东西来负责静态文件的处理是很浪费的行为，而且他们本身对文件的处理也不如nginx好，所以整个静态文件的处理都直接由nginx完成，静态文件的访问完全不去经过uwsgi以及其后面的东西。
```
{% endraw %}

{% raw %}
```
1.单机启动django项目，性能低，默认使用wsgiref模块，性能低的wsgi协议

python3 manager.py runserver 0.0.0.0:8000   > wsgiref模块中

2.高并发启动django，django是没有这个功能的，而uWSGI模块，遵循uwsgi协议，支持多进程处理django请求

uwsgi  通过他，启动你的django，而不再是python3 manager.py runserver 0.0.0.0:8000


3.公司中一般用 nginx + uwsgi + django + virtualenv  + supervisord（进程管理工具）


搭建笔记：
    1.确定依赖组件是否安装
    yum install zlib-devel bzip2-devel pcre-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel
    
    

    
    
nginx 正向代理，反向代理的概念




用户阿段，去访问mycrm.com:80 ，他想直接从80端口，找到hello视图，也就是mycrm.com:80/hello 
实现手段就是，阿段去访问 mycrm.com:80 这个nginx服务，并且让nginx，把hello这个请求，丢给后端的 uwsgi+django程序处理

1.基础环境准备好
yum groupinstall "Development tools"
yum install zlib-devel bzip2-devel pcre-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel

2.准备好python3环境

3.准备好virtualenv 

4.安装uWSGI
    1.激活虚拟环境
    source /opt/all_venv/venv2/bin/activate
    
    2.安装uWSGI
    (venv2) [root@s13linux ~ 05:18:21]$pip3 install uwsgi
    
    3.检查uwsgi版本
        (venv) [root@slave 192.168.11.64 /opt]$uwsgi --version
        2.0.17.1
        #检查uwsgi python版本
        uwsgi --python-version
    
    4.运行一个简单的uwsgi服务器
        1.创建一个test.py文件，写入内容
        def application(env, start_response):
            start_response('200 OK', [('Content-Type','text/html')])
            return [b"Hello World"] # python3
            
        2.然后用uwsgi命令启动
        uwsgi --http :8000 --wsgi-file test.py
            参数解释
            http :8000: 使用http协议，端口8000
            wsgi-file test.py: 加载指定的文件，test.py
        
    5.用uwsgi运行你的django项目（测试使用）
        1.准备好mysite，自己写好MTV视图函数  /hello 
    
    先确保你在项目文件夹下，例如/opt/mysite/底下
    
    uwsgi --http :8088 --module mysite.wsgi --py-autoreload=1 
        参数解析
            --http 启动在8088端口，--module 指定项目文件夹路径  --py-autoreload是热加载程序
    
    6.配置nginx反向代理uwsgi+django！！！！（此步重要！！！）
        1.首先kill杀掉nginx进程
        2.配置nginx.conf，通过此步才能生效！！
            填入重要两个参数，根据自己目录结构配置，uwsgi_pass通过这个参数，nginx才能转发请求给后端0.0.0.0:9000的应用
            include  /opt/nginx112/conf/uwsgi_params;
            uwsgi_pass 0.0.0.0:9000;
        --------------------------分割线--------------------------------------------------------
             server {
                    listen       80;
                    server_name  mycrm.com;
                    location / {
                        include  /opt/nginx112/conf/uwsgi_params;
                        uwsgi_pass 0.0.0.0:9000;
                        root   html;
                        index  index.html index.htm;
                        #deny 10.0.0.1;
        }
        
        配置nginx.conf之后，启动nginx服务，等待配置启动uwsgi+django 
    
    7.配置supervisor进程管理工具
        1.通过python2的包管理工具easy_install安装
        yum install python-setuptools
        easy_install supervisor
        
        2.通过命令生成supervisor的配支文件
        echo_supervisord_conf > /etc/supervisord.conf
        
        3.写入/etc/supervisord.conf配置信息(参数根据自己环境填写)
        [program:my_crm]
        command=/opt/all_venv/venv2/bin/uwsgi --uwsgi 0.0.0.0:9000 --chdir=/opt/s13crm --home=/opt/all_venv/venv2/ --module=s13crm.wsgi
        directory=/opt/s13crm
        startsecs=0
        stopwaitsecs=0
        autostart=true
        autorestart=true
    
    8.启动supervi服务，（同时启动uwsgi+django服务）
        最后启动supervisor，完成uWSGI启动django，nginx反向代理
        supervisord -c /etc/supervisord.conf #启动supervisor
        supervisorctl -c /etxc/supervisord.conf restart my  #重启my项目
        supervisorctl -c /etc/supervisord.conf [start|stop|restart] [program-name|all]

    9.此时访问网站mycrm.com ，查看是否可以通过80端口，访问到django应用，完成项目发布。
        由于nginx的高并发性能，配合uwsgi的多进程性能，可以达到一个线上的django应用发布！！！
        
                                                                        谢谢大家！
    
内容博客地址：
https://www.cnblogs.com/pyyu/p/9481344.html
    
    
    

13期课程笔记
```
{% endraw %}

{% raw %}
```
http://nginx.org/en/
```
{% endraw %}

{% raw %}
```
http://nginx.org/en/docs/http/ngx_http_uwsgi_module.html
```
{% endraw %}

大家都学过了django，用django写了各种功能，写了bbs项目，写了路飞学城。

咱们都知道django是一个web框架，方便我们快速开发web程序，http请求的动态数据就是由web框架来提供处理的。

前面超哥也对nginx简单的介绍了，本文将nginx、WSGI、uwsgi、uWSGI、django这几个关系梳理一下。

{% raw %}
```
wsgi    全称web server gateway interface，wsgi不是服务器，也不是python模块，只是一种协议，描述web server如何和web application通信的规则。运行在wsgi上的web框架有bottle，flask，django
```
{% endraw %}

{% raw %}
```
uwsgi    和wsgi一样是通信协议，是uWSGI服务器的单独协议，用于定义传输信息的类型
```
{% endraw %}

{% raw %}
```
uWSGI    是一个web服务器，实现了WSGI协议，uwsgi协议。a
```
{% endraw %}

{% raw %}
```
nginx    web服务器，更加安全，更好的处理处理静态资源，缓存功能，负载均衡，因此nginx的强劲性能，配合uWSGI服务器会更加安全，性能有保障。
```
{% endraw %}

{% raw %}
```
django 高级的python web框架，用于快速开发，解决web开发的大部分麻烦，程序员可以更专注业务逻辑，无须重新造轮子
```
{% endraw %}

### 逻辑图

<img src="https://img2018.cnblogs.com/blog/1132884/201811/1132884-20181117155443429-1430917061.png" alt="" />

web服务器

{% raw %}
```
传统的c/s架构，请求的过程是
客户端 > 服务器 
服务器 > 客户端
服务器就是：1.接收请求 2.处理请求 3.返回响应
```
{% endraw %}

web框架层

{% raw %}
```
HTTP的动态数据交给web框架，例如django遵循MTV模式处理请求。
HTTp协议使用url定位资源，urls.py将路由请求交给views视图处理，然后返回一个结果，完成一次请求。
web框架使用者只需要处理业务的逻辑即可。
```
{% endraw %}

如果将一次通信转化为对话的过程

WSGI：好的nginx，我马上设置环境变量，然后把请求交给django

Django：谢谢WSGI，我处理完请求马上给你响应结果

WSGI：好的，我在等着

Django：搞定啦，麻烦wsgi吧响应结果传递给nginx

WSGI：太棒了，nginx，响应结果请收好，已经按照要求传递给你了

# Django Nginx+uwsgi 安装配置

在前面的章节中我们使用 **python manage.py runserver** 来运行服务器。这只适用测试环境中使用。

正式发布的服务，需要一个可以稳定而持续的服务器。

#### 基础开发环境配置

{% raw %}
```
yum groupinstall "Development tools"
yum install zlib-devel bzip2-devel pcre-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel
```
{% endraw %}

#### 提前安装好python3环境

{% raw %}
```
https://www.cnblogs.com/pyyu/p/7402145.html
```
{% endraw %}

#### virtualenv

{% raw %}
```
请确保你的虚拟环境正常工作https://www.cnblogs.com/pyyu/p/9015317.html
```
{% endraw %}

#### 安装django1.11

{% raw %}
```
pip3 install django==1.11#创建django项目mysitedjango-admin startproject mysite#创建app01python3 manage.py startapp app01
```
{% endraw %}

mysite/settings.py

{% raw %}
```
#settings.py设置
ALLOWED_HOSTS = ['*']
install app01
```
{% endraw %}

mysite/urls.py

{% raw %}
```
from app01 import views
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^hello_django/', views.hello),
]
```
{% endraw %}

app01/views.py

{% raw %}
```
from django.shortcuts import render,HttpResponse

# Create your views here.
def hello(request):
    print('request is :',request)
    return HttpResponse('django is ok ')
```
{% endraw %}

#### 安装uWSGI

{% raw %}
```
进入虚拟环境venv，安装uwsgi
(venv) [root@slave 192.168.11.64 /opt]$pip3 install uwsgi检查uwsgi版本(venv) [root@slave 192.168.11.64 /opt]$uwsgi --version2.0.17.1#检查uwsgi python版本uwsgi --python-version
```
{% endraw %}

运行简单的uWSGI

{% raw %}
```
#启动一个python
uwsgi --http :8000 --wsgi-file test.py
```
{% endraw %}

- `http :8000`: 使用http协议，端口8000
- `wsgi-file test.py`: 加载指定的文件，test.py

{% raw %}
```
#test.py
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"] # python3
```
{% endraw %}

uWsgi热加载python程序

{% raw %}
```
在启动命令后面加上参数
uwsgi --http :8088 --module mysite.wsgi --py-autoreload=1 #发布命令command= /home/venv/bin/uwsgi --uwsgi 0.0.0.0:8000 --chdir /opt/mysite --home=/home/venv --module mysite.wsgi
```
{% endraw %}

{% raw %}
```
#此时修改django代码，uWSGI会自动加载django程序，页面生效
```
{% endraw %}

运行django程序

{% raw %}
```
#mysite/wsgi.py  确保找到这个文件uwsgi --http :8000 --module mysite.wsgi
```
{% endraw %}

- `module mysite.wsgi`: 加载指定的wsgi模块

uwsgi配置文件

{% raw %}
```
uwsgi支持ini、xml等多种配置方式，本文以 ini 为例， 在/etc/目录下新建uwsgi_nginx.ini，添加如下配置：

# mysite_uwsgi.ini file
[uwsgi]

# Django-related settings
# the base directory (full path)
chdir           = /opt/mysite
# Django's wsgi file
module          = mysite.wsgi
# the virtualenv (full path)
home            = /opt/venv
# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 1
# the socket (use the full path to be safe
socket          = 0.0.0.0:8000
# ... with appropriate permissions - may be needed
# chmod-socket    = 664
# clear environment on exit
vacuum          = true
```
{% endraw %}

#### 指定配置文件启动命令

{% raw %}
```
uwsgi --ini  /etc/uwsgi_nginx.ini
```
{% endraw %}

#### 配置nginx结合uWSGI

配置nginx.conf

{% raw %}
```
worker_processes  1;
error_log  logs/error.log;
pid        logs/nginx.pid;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  logs/access.log  main;
    sendfile        on;
    keepalive_timeout  65;　　#nginx反向代理uwsgi
    server {
        listen       80;
        server_name  192.168.11.64;
        location / {　　　　  #nginx自带ngx_http_uwsgi_module模块，起到nginx和uwsgi交互作用         #通过uwsgi_pass设置服务器地址和协议，讲动态请求转发给uwsgi处理
         include  /opt/nginx1-12/conf/uwsgi_params;
         uwsgi_pass 0.0.0.0:8000;
            root   html;
            index  index.html index.htm;
        }　　　　  #nginx处理静态页面资源　　　　  location /static{　　　　　　　　alias /opt/nginx1-12/static;　　　         }　　　　　#nginx处理媒体资源　　　　　location /media{　　　　　　　　alias /opt/nginx1-12/media;　　
```
{% endraw %}

{% raw %}
```
**         }****<em id="__mceDel">        error_page   500 502 503 504  /50x.html;**</em>
```
{% endraw %}

{% raw %}
```
<em id="__mceDel"><em id="__mceDel">        location = /50x.html {
            root   html;
        }
    }
}</em></em>
```
{% endraw %}

配置完启动nginx

# supervisor

这里超哥要配置基于virtualenv的supervisor

由于supervisor在python3下无法使用，因此只能用python2去下载！！！！！！

{% raw %}
```
#注意此时已经退出虚拟环境了！！！！！yum install python-setuptoolseasy_install supervisor
```
{% endraw %}

通过命令生成supervisor的配支文件

{% raw %}
```
echo_supervisord_conf > /etc/supervisord.conf
```
{% endraw %}

然后再/etc/supervisord.conf末尾添加上如下代码！！！！！！

{% raw %}
```
supervisord.conf配置文件参数解释
[program:xx]是被管理的进程配置参数，xx是进程的名称
[program:xx]
command=/opt/apache-tomcat-8.0.35/bin/catalina.sh run  ; 程序启动命令
autostart=true       ; 在supervisord启动的时候也自动启动
startsecs=10         ; 启动10秒后没有异常退出，就表示进程正常启动了，默认为1秒
autorestart=true     ; 程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外杀死后才重启
startretries=3       ; 启动失败自动重试次数，默认是3
user=tomcat          ; 用哪个用户启动进程，默认是root
priority=999         ; 进程启动优先级，默认999，值小的优先启动
redirect_stderr=true ; 把stderr重定向到stdout，默认false
stdout_logfile_maxbytes=20MB  ; stdout 日志文件大小，默认50MB
stdout_logfile_backups = 20   ; stdout 日志文件备份数，默认是10
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile=/opt/apache-tomcat-8.0.35/logs/catalina.out
stopasgroup=false     ;默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
killasgroup=false     ;默认为false，向进程组发送kill信号，包括子进程
```
{% endraw %}

{% raw %}
```
[program:my]
#command=/opt/venv/bin/uwsgi --ini  /etc/uwsgi_nginx.ini  #这里是结合virtualenv的命令 和supervisor的精髓！！！！command= /home/venv/bin/uwsgi --uwsgi 0.0.0.0:8000 --chdir /opt/mysite --home=/home/venv --module mysite.wsgi#--home指的是虚拟环境目录  --module找到 mysite/wsgi.py   
```
{% endraw %}

最后启动supervisor，完成uWSGI启动django，nginx反向代理

{% raw %}
```
supervisord -c /etc/supervisord.conf #启动supervisorsupervisorctl -c /etxc/supervisord.conf restart my  #重启my项目`supervisorctl -c ``/etc/supervisord``.conf [start|stop|restart] [program-name|all]`
```
{% endraw %}

{% raw %}
```
一、添加好配置文件后

二、更新新的配置到supervisord    

supervisorctl update
三、重新启动配置中的所有程序

supervisorctl reload
四、启动某个进程(program_name=你配置中写的程序名称)

supervisorctl start program_name
五、查看正在守候的进程

supervisorctl
六、停止某一进程 (program_name=你配置中写的程序名称)

pervisorctl stop program_name
七、重启某一进程 (program_name=你配置中写的程序名称)

supervisorctl restart program_name
八、停止全部进程

supervisorctl stop all
注意：显示用stop停止掉的进程，用reload或者update都不会自动重启。
```
{% endraw %}

#  django的静态文件与nginx配置

mysite/settings.py

{% raw %}
```
STATIC_ROOT='/opt/nginx1-12/static'
STATIC_URL = '/static/'
STATICFILES_DIRS=[
    os.path.join(BASE_DIR,"static"),
]
```
{% endraw %}

上述的参数STATIC_ROOT用在哪？

通过python3 manage.py collectstatic 收集所有你使用的静态文件保存到STATIC_ROOT！

{% raw %}
```
STATIC_ROOT 文件夹 是用来将所有STATICFILES_DIRS中所有文件夹中的文件，以及各app中static中的文件都复制过来
# 把这些文件放到一起是为了用nginx等部署的时候更方便
```
{% endraw %}

** **

参考文档：http://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/tutorials/Django_and_nginx.html
