---
layout:     post
title:      DjangoRestFramework
subtitle:   
date:       2019-04-02
author:     P
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
    - python
---
## 目录

- [一. 什么是RESTful ](#_label0)
- [二. RESTful API设计](#_label1)
- [三. 基于Django实现](#_label2)
<li>[四. 基于Django Rest Framework框架实现](#_label3)
<ul>
- [1. 基本流程](#_label3_0)
- [2.  认证和授权](#_label3_1)
- [3. 用户访问次数/频率限制](#_label3_2)
- [4. 版本](#_label3_3)
- [5. 解析器（parser） ](#_label3_4)
- [6. 序列化](#_label3_5)
- [7. 分页](#_label3_6)
- [8. 路由系统](#_label3_7)
- [9. 视图](#_label3_8)
- [10. 渲染器](#_label3_9)

## 一. 什么是RESTful 

- REST与技术无关，代表的是一种软件架构风格，REST是Representational State Transfer的简称，中文翻译为表征状态转移
- REST从资源的角度类审视整个网络，它将分布在网络中某个节点的资源通过URL进行标识，客户端应用通过URL来获取资源的表征，获得这些表征致使这些应用转变状态
- 所有的数据，不过是通过网络获取的还是操作（增删改查）的数据，都是资源，将一切数据视为资源是REST区别与其他架构风格的最本质属性
- 对于REST这种面向资源的架构风格，有人提出一种全新的结构理念，即：面向资源架构（ROA：Resource Oriented Architecture）

## 二. RESTful API设计

- API与用户的通信协议，总是使用[HTTPs协议](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)。
<li>域名 
<ul>
- https://api.example.com                         尽量将API部署在专用域名（会存在跨域问题）
- https://example.org/api/                        API很简单

- URL，如：https://api.example.com/v1/
- 请求头                                                  跨域时，引发发送多次请求

- https://api.example.com/v1/zoos
- https://api.example.com/v1/animals
- https://api.example.com/v1/employees

- GET      ：从服务器取出资源（一项或多项）
- POST    ：在服务器新建一个资源
- PUT      ：在服务器更新资源（客户端提供改变后的完整资源）
- PATCH  ：在服务器更新资源（客户端提供改变的属性）
- DELETE ：从服务器删除资源

- https://api.example.com/v1/zoos?limit=10：指定返回记录的数量
- https://api.example.com/v1/zoos?offset=10：指定返回记录的开始位置
- https://api.example.com/v1/zoos?page=2&per_page=100：指定第几页，以及每页的记录数
- https://api.example.com/v1/zoos?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序
- https://api.example.com/v1/zoos?animal_type_id=1：指定筛选条件

```
200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。
400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。

更多看这里：http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html
```
<td class="gutter">123</td><td class="code">`{``    ``error: ``"Invalid API key"``}`</td>
<td class="gutter">123456</td><td class="code">`GET ``/``collection：返回资源对象的列表（数组）``GET ``/``collection``/``resource：返回单个资源对象``POST ``/``collection：返回新生成的资源对象``PUT ``/``collection``/``resource：返回完整的资源对象``PATCH ``/``collection``/``resource：返回完整的资源对象``DELETE ``/``collection``/``resource：返回一个空文档`</td>
<td class="gutter">123456</td><td class="code">`{``"link"``: {``  ``"rel"``:   ``"collection https://www.example.com/zoos"``,``  ``"href"``:  ``"https://api.example.com/zoos"``,``  ``"title"``: ``"List of zoos"``,``  ``"type"``:  ``"application/vnd.yourformat+json"``}}`</td>

　　摘自：http://www.ruanyifeng.com/blog/2014/05/restful_api.html 

## 三. 基于Django实现

路由系统：
<td class="gutter">123</td><td class="code">`urlpatterns ``=` `[``    ``url(r``'^users'``, Users.as_view()),``]`</td>

CBV视图：
<td class="gutter">1234567891011121314151617</td><td class="code">`from` `django.views ``import` `View``from` `django.http ``import` `JsonResponse` `class` `Users(View):``    ``def` `get(``self``, request, ``*``args, ``*``*``kwargs):``        ``result ``=` `{``            ``'status'``: ``True``,``            ``'data'``: ``'response data'``        ``}``        ``return` `JsonResponse(result, status``=``200``)` `    ``def` `post(``self``, request, ``*``args, ``*``*``kwargs):``        ``result ``=` `{``            ``'status'``: ``True``,``            ``'data'``: ``'response data'``        ``}``        ``return` `JsonResponse(result, status``=``200``) `</td>

## 四. 基于Django Rest Framework框架实现

### **1. 基本流程**

url.py
<td class="gutter">123456</td><td class="code">`from` `django.conf.urls ``import` `url, include``from` `web.views.s1_api ``import` `TestView` `urlpatterns ``=` `[``    ``url(r``'^test/'``, TestView.as_view()),``]`</td>

views.py
<td class="gutter">123456789101112131415161718192021</td><td class="code">`from` `rest_framework.views ``import` `APIView``from` `rest_framework.response ``import` `Response`  `class` `TestView(APIView):``    ``def` `dispatch(``self``, request, ``*``args, ``*``*``kwargs):``        ``"""``        ``请求到来之后，都要执行dispatch方法，dispatch方法根据请求方式不同触发 get/post/put等方法``        ` `        ``注意：APIView中的dispatch方法有好多好多的功能``        ``"""``        ``return` `super``().dispatch(request, ``*``args, ``*``*``kwargs)` `    ``def` `get(``self``, request, ``*``args, ``*``*``kwargs):``        ``return` `Response(``'GET请求，响应内容'``)` `    ``def` `post(``self``, request, ``*``args, ``*``*``kwargs):``        ``return` `Response(``'POST请求，响应内容'``)` `    ``def` `put(``self``, request, ``*``args, ``*``*``kwargs):``        ``return` `Response(``'PUT请求，响应内容'``)`</td>

上述是rest framework框架基本流程，重要的功能是在APIView的dispatch中触发。

### **2.  认证和授权**

a. 用户url传入的token认证

```
from django.conf.urls import url, include
from web.viewsimport TestView

urlpatterns = [
    url(r'^test/', TestView.as_view()),
]
```

```
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.authentication import BaseAuthentication
from rest_framework.request import Request
from rest_framework import exceptions

token_list = [
    'sfsfss123kuf3j123',
    'asijnfowerkkf9812',
]


class TestAuthentication(BaseAuthentication):
    def authenticate(self, request):
        """
        用户认证，如果验证成功后返回元组： (用户,用户Token)
        :param request: 
        :return: 
            None,表示跳过该验证；
                如果跳过了所有认证，默认用户和Token和使用配置文件进行设置
                self._authenticator = None
                if api_settings.UNAUTHENTICATED_USER:
                    self.user = api_settings.UNAUTHENTICATED_USER()
                else:
                    self.user = None
        
                if api_settings.UNAUTHENTICATED_TOKEN:
                    self.auth = api_settings.UNAUTHENTICATED_TOKEN()
                else:
                    self.auth = None
            (user,token)表示验证通过并设置用户名和Token；
            AuthenticationFailed异常
        """
        val = request.query_params.get('token')
        if val not in token_list:
            raise exceptions.AuthenticationFailed("用户认证失败")

        return ('登录用户', '用户token')

    def authenticate_header(self, request):
        """
        Return a string to be used as the value of the `WWW-Authenticate`
        header in a `401 Unauthenticated` response, or `None` if the
        authentication scheme should return `403 Permission Denied` responses.
        """
        # 验证失败时，返回的响应头WWW-Authenticate对应的值
        pass


class TestView(APIView):
    authentication_classes = [TestAuthentication, ]
    permission_classes = []

    def get(self, request, *args, **kwargs):
        print(request.user)
        print(request.auth)
        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

```
from django.conf.urls import url, include
from web.viewsimport TestView

urlpatterns = [
    url(r'^test/', TestView.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.authentication import BaseAuthentication
from rest_framework.request import Request
from rest_framework import exceptions

token_list = [
    'sfsfss123kuf3j123',
    'asijnfowerkkf9812',
]


class TestAuthentication(BaseAuthentication):
    def authenticate(self, request):
        """
        用户认证，如果验证成功后返回元组： (用户,用户Token)
        :param request: 
        :return: 
            None,表示跳过该验证；
                如果跳过了所有认证，默认用户和Token和使用配置文件进行设置
                self._authenticator = None
                if api_settings.UNAUTHENTICATED_USER:
                    self.user = api_settings.UNAUTHENTICATED_USER()
                else:
                    self.user = None
        
                if api_settings.UNAUTHENTICATED_TOKEN:
                    self.auth = api_settings.UNAUTHENTICATED_TOKEN()
                else:
                    self.auth = None
            (user,token)表示验证通过并设置用户名和Token；
            AuthenticationFailed异常
        """
        import base64
        auth = request.META.get('HTTP_AUTHORIZATION', b'')
        if auth:
            auth = auth.encode('utf-8')
        auth = auth.split()
        if not auth or auth[0].lower() != b'basic':
            raise exceptions.AuthenticationFailed('验证失败')
        if len(auth) != 2:
            raise exceptions.AuthenticationFailed('验证失败')
        username, part, password = base64.b64decode(auth[1]).decode('utf-8').partition(':')
        if username == 'alex' and password == '123':
            return ('登录用户', '用户token')
        else:
            raise exceptions.AuthenticationFailed('用户名或密码错误')

    def authenticate_header(self, request):
        """
        Return a string to be used as the value of the `WWW-Authenticate`
        header in a `401 Unauthenticated` response, or `None` if the
        authentication scheme should return `403 Permission Denied` responses.
        """
        return 'Basic realm=api'


class TestView(APIView):
    authentication_classes = [TestAuthentication, ]
    permission_classes = []

    def get(self, request, *args, **kwargs):
        print(request.user)
        print(request.auth)
        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

**c.** 多个认证规则

```
from django.conf.urls import url, include
from web.views.s2_auth import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.authentication import BaseAuthentication
from rest_framework.request import Request
from rest_framework import exceptions

token_list = [
    'sfsfss123kuf3j123',
    'asijnfowerkkf9812',
]


class Test1Authentication(BaseAuthentication):
    def authenticate(self, request):
        """
        用户认证，如果验证成功后返回元组： (用户,用户Token)
        :param request: 
        :return: 
            None,表示跳过该验证；
                如果跳过了所有认证，默认用户和Token和使用配置文件进行设置
                self._authenticator = None
                if api_settings.UNAUTHENTICATED_USER:
                    self.user = api_settings.UNAUTHENTICATED_USER() # 默认值为：匿名用户
                else:
                    self.user = None

                if api_settings.UNAUTHENTICATED_TOKEN:
                    self.auth = api_settings.UNAUTHENTICATED_TOKEN()# 默认值为：None
                else:
                    self.auth = None
            (user,token)表示验证通过并设置用户名和Token；
            AuthenticationFailed异常
        """
        import base64
        auth = request.META.get('HTTP_AUTHORIZATION', b'')
        if auth:
            auth = auth.encode('utf-8')
        else:
            return None
        print(auth,'xxxx')
        auth = auth.split()
        if not auth or auth[0].lower() != b'basic':
            raise exceptions.AuthenticationFailed('验证失败')
        if len(auth) != 2:
            raise exceptions.AuthenticationFailed('验证失败')
        username, part, password = base64.b64decode(auth[1]).decode('utf-8').partition(':')
        if username == 'alex' and password == '123':
            return ('登录用户', '用户token')
        else:
            raise exceptions.AuthenticationFailed('用户名或密码错误')

    def authenticate_header(self, request):
        """
        Return a string to be used as the value of the `WWW-Authenticate`
        header in a `401 Unauthenticated` response, or `None` if the
        authentication scheme should return `403 Permission Denied` responses.
        """
        # return 'Basic realm=api'
        pass

class Test2Authentication(BaseAuthentication):
    def authenticate(self, request):
        """
        用户认证，如果验证成功后返回元组： (用户,用户Token)
        :param request: 
        :return: 
            None,表示跳过该验证；
                如果跳过了所有认证，默认用户和Token和使用配置文件进行设置
                self._authenticator = None
                if api_settings.UNAUTHENTICATED_USER:
                    self.user = api_settings.UNAUTHENTICATED_USER() # 默认值为：匿名用户
                else:
                    self.user = None
        
                if api_settings.UNAUTHENTICATED_TOKEN:
                    self.auth = api_settings.UNAUTHENTICATED_TOKEN()# 默认值为：None
                else:
                    self.auth = None
            (user,token)表示验证通过并设置用户名和Token；
            AuthenticationFailed异常
        """
        val = request.query_params.get('token')
        if val not in token_list:
            raise exceptions.AuthenticationFailed("用户认证失败")

        return ('登录用户', '用户token')

    def authenticate_header(self, request):
        """
        Return a string to be used as the value of the `WWW-Authenticate`
        header in a `401 Unauthenticated` response, or `None` if the
        authentication scheme should return `403 Permission Denied` responses.
        """
        pass


class TestView(APIView):
    authentication_classes = [Test1Authentication, Test2Authentication]
    permission_classes = []

    def get(self, request, *args, **kwargs):
        print(request.user)
        print(request.auth)
        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

**d.** 认证和权限

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.authentication import BaseAuthentication
from rest_framework.permissions import BasePermission

from rest_framework.request import Request
from rest_framework import exceptions

token_list = [
    'sfsfss123kuf3j123',
    'asijnfowerkkf9812',
]


class TestAuthentication(BaseAuthentication):
    def authenticate(self, request):
        """
        用户认证，如果验证成功后返回元组： (用户,用户Token)
        :param request: 
        :return: 
            None,表示跳过该验证；
                如果跳过了所有认证，默认用户和Token和使用配置文件进行设置
                self._authenticator = None
                if api_settings.UNAUTHENTICATED_USER:
                    self.user = api_settings.UNAUTHENTICATED_USER() # 默认值为：匿名用户
                else:
                    self.user = None
        
                if api_settings.UNAUTHENTICATED_TOKEN:
                    self.auth = api_settings.UNAUTHENTICATED_TOKEN()# 默认值为：None
                else:
                    self.auth = None
            (user,token)表示验证通过并设置用户名和Token；
            AuthenticationFailed异常
        """
        val = request.query_params.get('token')
        if val not in token_list:
            raise exceptions.AuthenticationFailed("用户认证失败")

        return ('登录用户', '用户token')

    def authenticate_header(self, request):
        """
        Return a string to be used as the value of the `WWW-Authenticate`
        header in a `401 Unauthenticated` response, or `None` if the
        authentication scheme should return `403 Permission Denied` responses.
        """
        pass


class TestPermission(BasePermission):
    message = "权限验证失败"

    def has_permission(self, request, view):
        """
        判断是否有权限访问当前请求
        Return `True` if permission is granted, `False` otherwise.
        :param request: 
        :param view: 
        :return: True有权限；False无权限
        """
        if request.user == "管理员":
            return True

    # GenericAPIView中get_object时调用
    def has_object_permission(self, request, view, obj):
        """
        视图继承GenericAPIView，并在其中使用get_object时获取对象时，触发单独对象权限验证
        Return `True` if permission is granted, `False` otherwise.
        :param request: 
        :param view: 
        :param obj: 
        :return: True有权限；False无权限
        """
        if request.user == "管理员":
            return True


class TestView(APIView):
    # 认证的动作是由request.user触发
    authentication_classes = [TestAuthentication, ]

    # 权限
    # 循环执行所有的权限
    permission_classes = [TestPermission, ]

    def get(self, request, *args, **kwargs):
        # self.dispatch
        print(request.user)
        print(request.auth)
        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

**e.** 全局使用

上述操作中均是对单独视图进行特殊配置，如果想要对全局进行配置，则需要再配置文件中写入即可。

```
REST_FRAMEWORK = {
    'UNAUTHENTICATED_USER': None,
    'UNAUTHENTICATED_TOKEN': None,
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "web.utils.TestAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "web.utils.TestPermission",
    ],
}
```

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response

class TestView(APIView):

    def get(self, request, *args, **kwargs):
        # self.dispatch
        print(request.user)
        print(request.auth)
        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

### **3. 用户访问次数/频率限制**

a. 基于用户IP限制访问频率

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import time
from rest_framework.views import APIView
from rest_framework.response import Response

from rest_framework import exceptions
from rest_framework.throttling import BaseThrottle
from rest_framework.settings import api_settings

# 保存访问记录
RECORD = {
    '用户IP': [12312139, 12312135, 12312133, ]
}


class TestThrottle(BaseThrottle):
    ctime = time.time

    def get_ident(self, request):
        """
        根据用户IP和代理IP，当做请求者的唯一IP
        Identify the machine making the request by parsing HTTP_X_FORWARDED_FOR
        if present and number of proxies is > 0. If not use all of
        HTTP_X_FORWARDED_FOR if it is available, if not use REMOTE_ADDR.
        """
        xff = request.META.get('HTTP_X_FORWARDED_FOR')
        remote_addr = request.META.get('REMOTE_ADDR')
        num_proxies = api_settings.NUM_PROXIES

        if num_proxies is not None:
            if num_proxies == 0 or xff is None:
                return remote_addr
            addrs = xff.split(',')
            client_addr = addrs[-min(num_proxies, len(addrs))]
            return client_addr.strip()

        return ''.join(xff.split()) if xff else remote_addr

    def allow_request(self, request, view):
        """
        是否仍然在允许范围内
        Return `True` if the request should be allowed, `False` otherwise.
        :param request: 
        :param view: 
        :return: True，表示可以通过；False表示已超过限制，不允许访问
        """
        # 获取用户唯一标识（如：IP）

        # 允许一分钟访问10次
        num_request = 10
        time_request = 60

        now = self.ctime()
        ident = self.get_ident(request)
        self.ident = ident
        if ident not in RECORD:
            RECORD[ident] = [now, ]
            return True
        history = RECORD[ident]
        while history and history[-1] <= now - time_request:
            history.pop()
        if len(history) < num_request:
            history.insert(0, now)
            return True

    def wait(self):
        """
        多少秒后可以允许继续访问
        Optionally, return a recommended number of seconds to wait before
        the next request.
        """
        last_time = RECORD[self.ident][0]
        now = self.ctime()
        return int(60 + last_time - now)


class TestView(APIView):
    throttle_classes = [TestThrottle, ]

    def get(self, request, *args, **kwargs):
        # self.dispatch
        print(request.user)
        print(request.auth)
        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')

    def throttled(self, request, wait):
        """
        访问次数被限制时，定制错误信息
        """

        class Throttled(exceptions.Throttled):
            default_detail = '请求被限制.'
            extra_detail_singular = '请 {wait} 秒之后再重试.'
            extra_detail_plural = '请 {wait} 秒之后再重试.'

        raise Throttled(wait)
```

```
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_RATES': {
        'test_scope': '10/m',
    },
}
```

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response

from rest_framework import exceptions
from rest_framework.throttling import SimpleRateThrottle


class TestThrottle(SimpleRateThrottle):

    # 配置文件定义的显示频率的Key
    scope = "test_scope"

    def get_cache_key(self, request, view):
        """
        Should return a unique cache-key which can be used for throttling.
        Must be overridden.

        May return `None` if the request should not be throttled.
        """
        if not request.user:
            ident = self.get_ident(request)
        else:
            ident = request.user

        return self.cache_format % {
            'scope': self.scope,
            'ident': ident
        }


class TestView(APIView):
    throttle_classes = [TestThrottle, ]

    def get(self, request, *args, **kwargs):
        # self.dispatch
        print(request.user)
        print(request.auth)
        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')

    def throttled(self, request, wait):
        """
        访问次数被限制时，定制错误信息
        """

        class Throttled(exceptions.Throttled):
            default_detail = '请求被限制.'
            extra_detail_singular = '请 {wait} 秒之后再重试.'
            extra_detail_plural = '请 {wait} 秒之后再重试.'

        raise Throttled(wait)
```

c. view中限制请求频率

```
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_RATES': {
        'xxxxxx': '10/m',
    },
}
```

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response

from rest_framework import exceptions
from rest_framework.throttling import ScopedRateThrottle


# 继承 ScopedRateThrottle
class TestThrottle(ScopedRateThrottle):

    def get_cache_key(self, request, view):
        """
        Should return a unique cache-key which can be used for throttling.
        Must be overridden.

        May return `None` if the request should not be throttled.
        """
        if not request.user:
            ident = self.get_ident(request)
        else:
            ident = request.user

        return self.cache_format % {
            'scope': self.scope,
            'ident': ident
        }


class TestView(APIView):
    throttle_classes = [TestThrottle, ]

    # 在settings中获取 xxxxxx 对应的频率限制值
    throttle_scope = "xxxxxx"

    def get(self, request, *args, **kwargs):
        # self.dispatch
        print(request.user)
        print(request.auth)
        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')

    def throttled(self, request, wait):
        """
        访问次数被限制时，定制错误信息
        """

        class Throttled(exceptions.Throttled):
            default_detail = '请求被限制.'
            extra_detail_singular = '请 {wait} 秒之后再重试.'
            extra_detail_plural = '请 {wait} 秒之后再重试.'

        raise Throttled(wait)
```

d. 匿名时用IP限制+登录时用Token限制

```
REST_FRAMEWORK = {
    'UNAUTHENTICATED_USER': None,
    'UNAUTHENTICATED_TOKEN': None,
    'DEFAULT_THROTTLE_RATES': {
        'luffy_anon': '10/m',
        'luffy_user': '20/m',
    },
}
```

```
from django.conf.urls import url, include
from web.views.s3_throttling import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response

from rest_framework.throttling import SimpleRateThrottle


class LuffyAnonRateThrottle(SimpleRateThrottle):
    """
    匿名用户，根据IP进行限制
    """
    scope = "luffy_anon"

    def get_cache_key(self, request, view):
        # 用户已登录，则跳过 匿名频率限制
        if request.user:
            return None

        return self.cache_format % {
            'scope': self.scope,
            'ident': self.get_ident(request)
        }


class LuffyUserRateThrottle(SimpleRateThrottle):
    """
    登录用户，根据用户token限制
    """
    scope = "luffy_user"

    def get_ident(self, request):
        """
        认证成功时：request.user是用户对象；request.auth是token对象
        :param request: 
        :return: 
        """
        # return request.auth.token
        return "user_token"

    def get_cache_key(self, request, view):
        """
        获取缓存key
        :param request: 
        :param view: 
        :return: 
        """
        # 未登录用户，则跳过 Token限制
        if not request.user:
            return None

        return self.cache_format % {
            'scope': self.scope,
            'ident': self.get_ident(request)
        }


class TestView(APIView):
    throttle_classes = [LuffyUserRateThrottle, LuffyAnonRateThrottle, ]

    def get(self, request, *args, **kwargs):
        # self.dispatch
        print(request.user)
        print(request.auth)
        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

e. 全局使用

```
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'api.utils.throttles.throttles.LuffyAnonRateThrottle',
        'api.utils.throttles.throttles.LuffyUserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '10/day',
        'user': '10/day',
        'luffy_anon': '10/m',
        'luffy_user': '20/m',
    },
}
```

### **4. 版本**

a. 基于url的get传参方式

**如：/users?version=v1**

```
REST_FRAMEWORK = {
    'DEFAULT_VERSION': 'v1',            # 默认版本
    'ALLOWED_VERSIONS': ['v1', 'v2'],   # 允许的版本
    'VERSION_PARAM': 'version'          # URL中获取值的key
}
```

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view(),name='test'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.versioning import QueryParameterVersioning


class TestView(APIView):
    versioning_class = QueryParameterVersioning

    def get(self, request, *args, **kwargs):

        # 获取版本
        print(request.version)
        # 获取版本管理的类
        print(request.versioning_scheme)

        # 反向生成URL
        reverse_url = request.versioning_scheme.reverse('test', request=request)
        print(reverse_url)

        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

**如：/v1/users/**

```
REST_FRAMEWORK = {
    'DEFAULT_VERSION': 'v1',            # 默认版本
    'ALLOWED_VERSIONS': ['v1', 'v2'],   # 允许的版本
    'VERSION_PARAM': 'version'          # URL中获取值的key
}
```

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^(?P<version>[v1|v2]+)/test/', TestView.as_view(), name='test'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.versioning import URLPathVersioning


class TestView(APIView):
    versioning_class = URLPathVersioning

    def get(self, request, *args, **kwargs):
        # 获取版本
        print(request.version)
        # 获取版本管理的类
        print(request.versioning_scheme)

        # 反向生成URL
        reverse_url = request.versioning_scheme.reverse('test', request=request)
        print(reverse_url)

        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

c. 基于 accept 请求头方式

**如：Accept: application/json; version=1.0**

```
REST_FRAMEWORK = {
    'DEFAULT_VERSION': 'v1',            # 默认版本
    'ALLOWED_VERSIONS': ['v1', 'v2'],   # 允许的版本
    'VERSION_PARAM': 'version'          # URL中获取值的key
}
```

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view(), name='test'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.versioning import AcceptHeaderVersioning


class TestView(APIView):
    versioning_class = AcceptHeaderVersioning

    def get(self, request, *args, **kwargs):
        # 获取版本 HTTP_ACCEPT头
        print(request.version)
        # 获取版本管理的类
        print(request.versioning_scheme)
        # 反向生成URL
        reverse_url = request.versioning_scheme.reverse('test', request=request)
        print(reverse_url)

        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

d. 基于主机名方法

**如：v1.example.com**

```
ALLOWED_HOSTS = ['*']
REST_FRAMEWORK = {
    'DEFAULT_VERSION': 'v1',  # 默认版本
    'ALLOWED_VERSIONS': ['v1', 'v2'],  # 允许的版本
    'VERSION_PARAM': 'version'  # URL中获取值的key
}
```

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view(), name='test'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.versioning import HostNameVersioning


class TestView(APIView):
    versioning_class = HostNameVersioning

    def get(self, request, *args, **kwargs):
        # 获取版本
        print(request.version)
        # 获取版本管理的类
        print(request.versioning_scheme)
        # 反向生成URL
        reverse_url = request.versioning_scheme.reverse('test', request=request)
        print(reverse_url)

        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

e. 基于django路由系统的namespace

**如：example.com/v1/users/**

```
REST_FRAMEWORK = {
    'DEFAULT_VERSION': 'v1',  # 默认版本
    'ALLOWED_VERSIONS': ['v1', 'v2'],  # 允许的版本
    'VERSION_PARAM': 'version'  # URL中获取值的key
}
```

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^v1/', ([
                      url(r'test/', TestView.as_view(), name='test'),
                  ], None, 'v1')),
    url(r'^v2/', ([
                      url(r'test/', TestView.as_view(), name='test'),
                  ], None, 'v2')),

]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.versioning import NamespaceVersioning


class TestView(APIView):
    versioning_class = NamespaceVersioning

    def get(self, request, *args, **kwargs):
        # 获取版本
        print(request.version)
        # 获取版本管理的类
        print(request.versioning_scheme)
        # 反向生成URL
        reverse_url = request.versioning_scheme.reverse('test', request=request)
        print(reverse_url)

        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

f. 全局使用

```
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS':"rest_framework.versioning.URLPathVersioning",
    'DEFAULT_VERSION': 'v1',
    'ALLOWED_VERSIONS': ['v1', 'v2'],
    'VERSION_PARAM': 'version' 
}
```

### 5. 解析器（parser） 

根据请求头 content-type 选择对应的解析器就请求体内容进行处理。

a. 仅处理请求头content-type为application/json的请求体

```
from django.conf.urls import url, include
from web.views.s5_parser import TestView

urlpatterns = [
    url(r'test/', TestView.as_view(), name='test'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.request import Request
from rest_framework.parsers import JSONParser


class TestView(APIView):
    parser_classes = [JSONParser, ]

    def post(self, request, *args, **kwargs):
        print(request.content_type)

        # 获取请求的值，并使用对应的JSONParser进行处理
        print(request.data)

        # application/x-www-form-urlencoded 或 multipart/form-data时，request.POST中才有值
        print(request.POST)
        print(request.FILES)

        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'test/', TestView.as_view(), name='test'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.request import Request
from rest_framework.parsers import FormParser


class TestView(APIView):
    parser_classes = [FormParser, ]

    def post(self, request, *args, **kwargs):
        print(request.content_type)

        # 获取请求的值，并使用对应的JSONParser进行处理
        print(request.data)

        # application/x-www-form-urlencoded 或 multipart/form-data时，request.POST中才有值
        print(request.POST)
        print(request.FILES)

        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

c. 仅处理请求头content-type为multipart/form-data的请求体

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'test/', TestView.as_view(), name='test'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.request import Request
from rest_framework.parsers import MultiPartParser


class TestView(APIView):
    parser_classes = [MultiPartParser, ]

    def post(self, request, *args, **kwargs):
        print(request.content_type)

        # 获取请求的值，并使用对应的JSONParser进行处理
        print(request.data)
        # application/x-www-form-urlencoded 或 multipart/form-data时，request.POST中才有值
        print(request.POST)
        print(request.FILES)
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="http://127.0.0.1:8000/test/" method="post" enctype="multipart/form-data">
    <input type="text" name="user" />
    <input type="file" name="img">

    <input type="submit" value="提交">

</form>
</body>
</html>
```

d. 仅上传文件

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'test/(?P<filename>[^/]+)', TestView.as_view(), name='test'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.request import Request
from rest_framework.parsers import FileUploadParser


class TestView(APIView):
    parser_classes = [FileUploadParser, ]

    def post(self, request, filename, *args, **kwargs):
        print(filename)
        print(request.content_type)

        # 获取请求的值，并使用对应的JSONParser进行处理
        print(request.data)
        # application/x-www-form-urlencoded 或 multipart/form-data时，request.POST中才有值
        print(request.POST)
        print(request.FILES)
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="http://127.0.0.1:8000/test/f1.numbers" method="post" enctype="multipart/form-data">
    <input type="text" name="user" />
    <input type="file" name="img">

    <input type="submit" value="提交">

</form>
</body>
</html>
```

e. 同时多个Parser

当同时使用多个parser时，rest framework会根据请求头content-type自动进行比对，并使用对应parser

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'test/', TestView.as_view(), name='test'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.request import Request
from rest_framework.parsers import JSONParser, FormParser, MultiPartParser


class TestView(APIView):
    parser_classes = [JSONParser, FormParser, MultiPartParser, ]

    def post(self, request, *args, **kwargs):
        print(request.content_type)

        # 获取请求的值，并使用对应的JSONParser进行处理
        print(request.data)
        # application/x-www-form-urlencoded 或 multipart/form-data时，request.POST中才有值
        print(request.POST)
        print(request.FILES)
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

f. 全局使用

```
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES':[
        'rest_framework.parsers.JSONParser'
        'rest_framework.parsers.FormParser'
        'rest_framework.parsers.MultiPartParser'
    ]

}
```

```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'test/', TestView.as_view(), name='test'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response


class TestView(APIView):
    def post(self, request, *args, **kwargs):
        print(request.content_type)

        # 获取请求的值，并使用对应的JSONParser进行处理
        print(request.data)
        # application/x-www-form-urlencoded 或 multipart/form-data时，request.POST中才有值
        print(request.POST)
        print(request.FILES)
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```

**注意：个别特殊的值可以通过Django的request对象 request._request 来进行获取**

### 6. 序列化

序列化用于对用户请求数据进行验证和数据进行序列化。

a. 自定义字段

```
from django.conf.urls import url, include
from web.views.s6_serializers import TestView

urlpatterns = [
    url(r'test/', TestView.as_view(), name='test'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import serializers
from .. import models


class PasswordValidator(object):
    def __init__(self, base):
        self.base = base

    def __call__(self, value):
        if value != self.base:
            message = 'This field must be %s.' % self.base
            raise serializers.ValidationError(message)

    def set_context(self, serializer_field):
        """
        This hook is called by the serializer instance,
        prior to the validation call being made.
        """
        # 执行验证之前调用,serializer_fields是当前字段对象
        pass


class UserSerializer(serializers.Serializer):
    ut_title = serializers.CharField(source='ut.title')
    user = serializers.CharField(min_length=6)
    pwd = serializers.CharField(error_messages={'required': '密码不能为空'}, validators=[PasswordValidator('666')])


class TestView(APIView):
    def get(self, request, *args, **kwargs):

        # 序列化，将数据库查询字段序列化为字典
        data_list = models.UserInfo.objects.all()
        ser = UserSerializer(instance=data_list, many=True)
        # 或
        # obj = models.UserInfo.objects.all().first()
        # ser = UserSerializer(instance=obj, many=False)
        return Response(ser.data)

    def post(self, request, *args, **kwargs):
        # 验证，对请求发来的数据进行验证
        ser = UserSerializer(data=request.data)
        if ser.is_valid():
            print(ser.validated_data)
        else:
            print(ser.errors)

        return Response('POST请求，响应内容')
```

```
from django.conf.urls import url, include
from web.views.s6_serializers import TestView

urlpatterns = [
    url(r'test/', TestView.as_view(), name='test'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import serializers
from .. import models


class PasswordValidator(object):
    def __init__(self, base):
        self.base = str(base)

    def __call__(self, value):
        if value != self.base:
            message = 'This field must be %s.' % self.base
            raise serializers.ValidationError(message)

    def set_context(self, serializer_field):
        """
        This hook is called by the serializer instance,
        prior to the validation call being made.
        """
        # 执行验证之前调用,serializer_fields是当前字段对象
        pass

class ModelUserSerializer(serializers.ModelSerializer):

    user = serializers.CharField(max_length=32)

    class Meta:
        model = models.UserInfo
        fields = "__all__"
        # fields = ['user', 'pwd', 'ut']
        depth = 2
        extra_kwargs = {'user': {'min_length': 6}, 'pwd': {'validators': [PasswordValidator(666), ]}}
        # read_only_fields = ['user']


class TestView(APIView):
    def get(self, request, *args, **kwargs):

        # 序列化，将数据库查询字段序列化为字典
        data_list = models.UserInfo.objects.all()
        ser = ModelUserSerializer(instance=data_list, many=True)
        # 或
        # obj = models.UserInfo.objects.all().first()
        # ser = UserSerializer(instance=obj, many=False)
        return Response(ser.data)

    def post(self, request, *args, **kwargs):
        # 验证，对请求发来的数据进行验证
        print(request.data)
        ser = ModelUserSerializer(data=request.data)
        if ser.is_valid():
            print(ser.validated_data)
        else:
            print(ser.errors)

        return Response('POST请求，响应内容')
```

c. 生成URL

```
from django.conf.urls import url, include
from web.views.s6_serializers import TestView

urlpatterns = [
    url(r'test/', TestView.as_view(), name='test'),
    url(r'detail/(?P<pk>\d+)/', TestView.as_view(), name='detail'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import serializers
from .. import models


class PasswordValidator(object):
    def __init__(self, base):
        self.base = str(base)

    def __call__(self, value):
        if value != self.base:
            message = 'This field must be %s.' % self.base
            raise serializers.ValidationError(message)

    def set_context(self, serializer_field):
        """
        This hook is called by the serializer instance,
        prior to the validation call being made.
        """
        # 执行验证之前调用,serializer_fields是当前字段对象
        pass


class ModelUserSerializer(serializers.ModelSerializer):
    ut = serializers.HyperlinkedIdentityField(view_name='detail')
    class Meta:
        model = models.UserInfo
        fields = "__all__"

        extra_kwargs = {
            'user': {'min_length': 6},
            'pwd': {'validators': [PasswordValidator(666),]},
        }



class TestView(APIView):
    def get(self, request, *args, **kwargs):

        # 序列化，将数据库查询字段序列化为字典
        data_list = models.UserInfo.objects.all()
        ser = ModelUserSerializer(instance=data_list, many=True, context={'request': request})
        # 或
        # obj = models.UserInfo.objects.all().first()
        # ser = UserSerializer(instance=obj, many=False)
        return Response(ser.data)

    def post(self, request, *args, **kwargs):
        # 验证，对请求发来的数据进行验证
        print(request.data)
        ser = ModelUserSerializer(data=request.data)
        if ser.is_valid():
            print(ser.validated_data)
        else:
            print(ser.errors)

        return Response('POST请求，响应内容')
```

### 7. 分页

a. 根据页码进行分页

```
from django.conf.urls import url, include
from rest_framework import routers
from web.views import s9_pagination

urlpatterns = [
    url(r'^test/', s9_pagination.UserViewSet.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework import serializers
from .. import models

from rest_framework.pagination import PageNumberPagination


class StandardResultsSetPagination(PageNumberPagination):
    # 默认每页显示的数据条数
    page_size = 1
    # 获取URL参数中设置的每页显示数据条数
    page_size_query_param = 'page_size'

    # 获取URL参数中传入的页码key
    page_query_param = 'page'

    # 最大支持的每页显示的数据条数
    max_page_size = 1


class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.UserInfo
        fields = "__all__"


class UserViewSet(APIView):
    def get(self, request, *args, **kwargs):
        user_list = models.UserInfo.objects.all().order_by('-id')

        # 实例化分页对象，获取数据库中的分页数据
        paginator = StandardResultsSetPagination()
        page_user_list = paginator.paginate_queryset(user_list, self.request, view=self)

        # 序列化对象
        serializer = UserSerializer(page_user_list, many=True)

        # 生成分页和数据
        response = paginator.get_paginated_response(serializer.data)
        return response
```

```
from django.conf.urls import url, include
from web.views import s9_pagination

urlpatterns = [
    url(r'^test/', s9_pagination.UserViewSet.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework import serializers
from .. import models

from rest_framework.pagination import PageNumberPagination,LimitOffsetPagination


class StandardResultsSetPagination(LimitOffsetPagination):
    # 默认每页显示的数据条数
    default_limit = 10
    # URL中传入的显示数据条数的参数
    limit_query_param = 'limit'
    # URL中传入的数据位置的参数
    offset_query_param = 'offset'
    # 最大每页显得条数
    max_limit = None

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.UserInfo
        fields = "__all__"


class UserViewSet(APIView):
    def get(self, request, *args, **kwargs):
        user_list = models.UserInfo.objects.all().order_by('-id')

        # 实例化分页对象，获取数据库中的分页数据
        paginator = StandardResultsSetPagination()
        page_user_list = paginator.paginate_queryset(user_list, self.request, view=self)

        # 序列化对象
        serializer = UserSerializer(page_user_list, many=True)

        # 生成分页和数据
        response = paginator.get_paginated_response(serializer.data)
        return response
```

c. 游标分页

```
from django.conf.urls import url, include
from web.views import s9_pagination

urlpatterns = [
    url(r'^test/', s9_pagination.UserViewSet.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework import serializers
from .. import models

from rest_framework.pagination import PageNumberPagination, LimitOffsetPagination, CursorPagination


class StandardResultsSetPagination(CursorPagination):
    # URL传入的游标参数
    cursor_query_param = 'cursor'
    # 默认每页显示的数据条数
    page_size = 2
    # URL传入的每页显示条数的参数
    page_size_query_param = 'page_size'
    # 每页显示数据最大条数
    max_page_size = 1000

    # 根据ID从大到小排列
    ordering = "id"



class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.UserInfo
        fields = "__all__"


class UserViewSet(APIView):
    def get(self, request, *args, **kwargs):
        user_list = models.UserInfo.objects.all().order_by('-id')

        # 实例化分页对象，获取数据库中的分页数据
        paginator = StandardResultsSetPagination()
        page_user_list = paginator.paginate_queryset(user_list, self.request, view=self)

        # 序列化对象
        serializer = UserSerializer(page_user_list, many=True)

        # 生成分页和数据
        response = paginator.get_paginated_response(serializer.data)
        return response
```

### 8. 路由系统

a. 自定义路由

```
from django.conf.urls import url, include
from web.views import s11_render

urlpatterns = [
    url(r'^test/$', s11_render.TestView.as_view()),
    url(r'^test\.(?P<format>[a-z0-9]+)$', s11_render.TestView.as_view()),
    url(r'^test/(?P<pk>[^/.]+)/$', s11_render.TestView.as_view()),
    url(r'^test/(?P<pk>[^/.]+)\.(?P<format>[a-z0-9]+)$', s11_render.TestView.as_view())
]
```

```
from rest_framework.views import APIView
from rest_framework.response import Response
from .. import models


class TestView(APIView):
    def get(self, request, *args, **kwargs):
        print(kwargs)
        print(self.renderer_classes)
        return Response('...')
```

```
from django.conf.urls import url, include
from web.views import s10_generic

urlpatterns = [
    url(r'^test/$', s10_generic.UserViewSet.as_view({'get': 'list', 'post': 'create'})),
    url(r'^test/(?P<pk>\d+)/$', s10_generic.UserViewSet.as_view(
        {'get': 'retrieve', 'put': 'update', 'patch': 'partial_update', 'delete': 'destroy'})),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.viewsets import ModelViewSet
from rest_framework import serializers
from .. import models


class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.UserInfo
        fields = "__all__"


class UserViewSet(ModelViewSet):
    queryset = models.UserInfo.objects.all()
    serializer_class = UserSerializer
```

c. 全自动路由

```
from django.conf.urls import url, include
from rest_framework import routers
from web.views import s10_generic


router = routers.DefaultRouter()
router.register(r'users', s10_generic.UserViewSet)

urlpatterns = [
    url(r'^', include(router.urls)),
]
```

```
from rest_framework.viewsets import ModelViewSet
from rest_framework import serializers
from .. import models


class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.UserInfo
        fields = "__all__"


class UserViewSet(ModelViewSet):
    queryset = models.UserInfo.objects.all()
    serializer_class = UserSerializer
```

### 9. 视图

a. GenericViewSet

```
from django.conf.urls import url, include
from web.views.s7_viewset import TestView

urlpatterns = [
    url(r'test/', TestView.as_view({'get':'list'}), name='test'),
    url(r'detail/(?P<pk>\d+)/', TestView.as_view({'get':'list'}), name='xxxx'),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework import viewsets
from rest_framework.response import Response


class TestView(viewsets.GenericViewSet):
    def list(self, request, *args, **kwargs):
        return Response('...')

    def add(self, request, *args, **kwargs):
        pass

    def delete(self, request, *args, **kwargs):
        pass

    def edit(self, request, *args, **kwargs):
        pass
```

```
from django.conf.urls import url, include
from web.views import s10_generic

urlpatterns = [
    url(r'^test/$', s10_generic.UserViewSet.as_view({'get': 'list', 'post': 'create'})),
    url(r'^test/(?P<pk>\d+)/$', s10_generic.UserViewSet.as_view(
        {'get': 'retrieve', 'put': 'update', 'patch': 'partial_update', 'delete': 'destroy'})),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.viewsets import ModelViewSet
from rest_framework import serializers
from .. import models


class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.UserInfo
        fields = "__all__"


class UserViewSet(ModelViewSet):
    queryset = models.UserInfo.objects.all()
    serializer_class = UserSerializer
```

c. ModelViewSet（rest framework路由）

```
from django.conf.urls import url, include
from rest_framework import routers
from app01 import views

router = routers.DefaultRouter()
router.register(r'users', views.UserViewSet)
router.register(r'groups', views.GroupViewSet)

# Wire up our API using automatic URL routing.
# Additionally, we include login URLs for the browsable API.
urlpatterns = [
    url(r'^', include(router.urls)),
]
```

```
from rest_framework import viewsets
from rest_framework import serializers


class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = models.User
        fields = ('url', 'username', 'email', 'groups')


class GroupSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = models.Group
        fields = ('url', 'name')
        
class UserViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows users to be viewed or edited.
    """
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer


class GroupViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows groups to be viewed or edited.
    """
    queryset = Group.objects.all()
    serializer_class = GroupSerializer
```

### 10. 渲染器

 根据 用户请求URL 或 用户可接受的类型，筛选出合适的 渲染组件。	用户请求URL：

- http://127.0.0.1:8000/test/?format=json
- http://127.0.0.1:8000/test.json









	用户请求头：

-     	Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8









a. json

访问URL：

- http://127.0.0.1:8000/test/?format=json
- http://127.0.0.1:8000/test.json
- http://127.0.0.1:8000/test/ 









```
from django.conf.urls import url, include
from web.views import s11_render

urlpatterns = [
    url(r'^test/$', s11_render.TestView.as_view()),
    url(r'^test\.(?P<format>[a-z0-9]+)', s11_render.TestView.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import serializers

from rest_framework.renderers import JSONRenderer

from .. import models


class TestSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.UserInfo
        fields = "__all__"


class TestView(APIView):
    renderer_classes = [JSONRenderer, ]

    def get(self, request, *args, **kwargs):
        user_list = models.UserInfo.objects.all()
        ser = TestSerializer(instance=user_list, many=True)
        return Response(ser.data)
```

访问URL：

- http://127.0.0.1:8000/test/?format=admin
- http://127.0.0.1:8000/test.admin
- http://127.0.0.1:8000/test/ 

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import serializers

from rest_framework.renderers import AdminRenderer

from .. import models


class TestSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.UserInfo
        fields = "__all__"


class TestView(APIView):
    renderer_classes = [AdminRenderer, ]

    def get(self, request, *args, **kwargs):
        user_list = models.UserInfo.objects.all()
        ser = TestSerializer(instance=user_list, many=True)
        return Response(ser.data)
```

c. Form表单

访问URL：

- http://127.0.0.1:8000/test/?format=form
- http://127.0.0.1:8000/test.form
- http://127.0.0.1:8000/test/ 

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import serializers

from rest_framework.renderers import JSONRenderer
from rest_framework.renderers import AdminRenderer
from rest_framework.renderers import HTMLFormRenderer

from .. import models


class TestSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.UserInfo
        fields = "__all__"


class TestView(APIView):
    renderer_classes = [HTMLFormRenderer, ]

    def get(self, request, *args, **kwargs):
        user_list = models.UserInfo.objects.all().first()
        ser = TestSerializer(instance=user_list, many=False)
        return Response(ser.data)
```

d. 自定义显示模板

访问URL：

- http://127.0.0.1:8000/test/?format=html
- http://127.0.0.1:8000/test.html
- http://127.0.0.1:8000/test/ 

```
from django.conf.urls import url, include
from web.views import s11_render

urlpatterns = [
    url(r'^test/$', s11_render.TestView.as_view()),
    url(r'^test\.(?P<format>[a-z0-9]+)', s11_render.TestView.as_view()),
]
```

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import serializers
from rest_framework.renderers import TemplateHTMLRenderer

from .. import models


class TestSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.UserInfo
        fields = "__all__"


class TestView(APIView):
    renderer_classes = [TemplateHTMLRenderer, ]

    def get(self, request, *args, **kwargs):
        user_list = models.UserInfo.objects.all().first()
        ser = TestSerializer(instance=user_list, many=False)
        return Response(ser.data, template_name='user_detail.html')
```

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    {{ user }}
    {{ pwd }}
    {{ ut }}
</body>
</html>
```

e. 浏览器格式API+JSON

访问URL：

- http://127.0.0.1:8000/test/?format=api
- http://127.0.0.1:8000/test.api
- http://127.0.0.1:8000/test/ 

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import serializers

from rest_framework.renderers import JSONRenderer
from rest_framework.renderers import BrowsableAPIRenderer

from .. import models


class TestSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.UserInfo
        fields = "__all__"


class CustomBrowsableAPIRenderer(BrowsableAPIRenderer):
    def get_default_renderer(self, view):
        return JSONRenderer()


class TestView(APIView):
    renderer_classes = [CustomBrowsableAPIRenderer, ]

    def get(self, request, *args, **kwargs):
        user_list = models.UserInfo.objects.all().first()
        ser = TestSerializer(instance=user_list, many=False)
        return Response(ser.data, template_name='user_detail.html')
```

注意：如果同时多个存在时，自动根据URL后缀来选择渲染器。
