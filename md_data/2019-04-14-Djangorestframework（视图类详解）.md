---
layout:     post
title:      Djangorestframework（视图类详解）
subtitle:   
date:       2019-04-14
author:     P
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - python
---
官网：[https://www.django-rest-framework.org/api-guide/viewsets/](https://www.django-rest-framework.org/api-guide/viewsets/)

在django rest framework 视图中一共有N个类

#### 第一类：APIview

```
<code class="python">class IndexView(APIView):

    def get(self,request,*args,**kwargs):
        pk = kwargs.get('pk')
        if pk:
            queryset = models.UserInfo.objects.get(pk=pk)
            ser = UserInfoSerializer(instance=queryset,many=False)
        else:
            queryset = models.UserInfo.objects.all()
            ser = UserInfoSerializer(instance=queryset,many=True)
        return Response(ser.data)
#
</code>
```

这种直接继承了APIView，是最原始的。请求方式就是那五种，get,post,put,patch,delete

#### 第二类 GenericAPIView

```
<code class="ruby">class IndexView(GenericAPIView):
    queryset = models.UserInfo.objects.all()
    serializer_class = UserInfoSerializer
    lookup_field = 'pk'

    def get(self,request,*args,**kwargs):
        pk = kwargs.get('pk')
        if pk:
            users = self.filter_queryset(queryset=models.UserInfo.objects.get(pk=pk))
            ser = self.get_serializer(instance=users)
        else:
            users = self.get_queryset()
            ser = self.get_serializer(instance=users,many=True)
        return Response(ser.data)
</code>
```

在GenericAPIView中要重写一些字段和方法，不常用。

#### 第三类 GenericViewSet

```
<code class="python">class IndexView(GenericViewSet):
    serializer_class = UserInfoSerializer
    queryset = models.UserInfo.objects.all()
    def create(self,request,*args,**kwargs):
        pass

    def list(self,request,*args,**kwargs):  # 获取列表数据
        users = models.UserInfo.objects.all()
        ser = UserInfoSerializer(instance=users,many=True)
        return Response(ser.data)

    def retrieve(self,request,*args,**kwargs):  # 获取单条数据
        pk = kwargs.get('pk')
        users = models.UserInfo.objects.get(pk=pk)
        ser = UserInfoSerializer(instance=users,many=False)
        return Response(ser.data)

    def destroy(self,request,*args,**kwargs):
        pass

    def update(self,request,*args,**kwargs):
        pass

    def partial_update(self,request,*args,**kwargs):
        pass

</code>
```

这个类继承了ViewSetMixin, generics.GenericAPIView，其中在ViewSetMixin中会重写as_view()方法，因此可以将URL中的请求方式与视图函数绑定到一起，在urls.py中以键值对的方式存在：
urls.py

```
<code class="python">from django.conf.urls import url
from django.contrib import admin
from app01 import  views

urlpatterns = [
    # url(r'^admin/', admin.site.urls),
    url(r'^hehe/', views.hehe),
    url(r'^index/$', views.IndexView.as_view({'get':'list','post':'create'})),
    url(r'^index/(?P<pk>\d+)/$', views.IndexView.as_view({'get':'retrieve','put':'update','patch':'partial_update','delete':'destroy'})),
]
</code>
```

ViewSetMixin源码部分：

```
<code class="python">class ViewSetMixin(object):
    """
    This is the magic.

    Overrides `.as_view()` so that it takes an `actions` keyword that performs
    the binding of HTTP methods to actions on the Resource.

    For example, to create a concrete view binding the 'GET' and 'POST' methods
    to the 'list' and 'create' actions...

    view = MyViewSet.as_view({'get': 'list', 'post': 'create'})
    """

    @classonlymethod
    def as_view(cls, actions=None, **initkwargs):
        """
        Because of the way class based views create a closure around the
        instantiated view, we need to totally reimplement `.as_view`,
        and slightly modify the view function that is created and returned.
        """
        # The suffix initkwarg is reserved for displaying the viewset type.
        # eg. 'List' or 'Instance'.
        cls.suffix = None

        # Setting a basename allows a view to reverse its action urls. This
        # value is provided by the router through the initkwargs.
        cls.basename = None

        # actions must not be empty
        if not actions:
            raise TypeError("The `actions` argument must be provided when "
                            "calling `.as_view()` on a ViewSet. For example "
                            "`.as_view({'get': 'list'})`")

        # sanitize keyword arguments
        for key in initkwargs:
            if key in cls.http_method_names:
                raise TypeError("You tried to pass in the %s method name as a "
                                "keyword argument to %s(). Don't do that."
                                % (key, cls.__name__))
            if not hasattr(cls, key):
                raise TypeError("%s() received an invalid keyword %r" % (
                    cls.__name__, key))

        def view(request, *args, **kwargs):
            self = cls(**initkwargs)
            # We also store the mapping of request methods to actions,
            # so that we can later set the action attribute.
            # eg. `self.action = 'list'` on an incoming GET request.
            self.action_map = actions

            # Bind methods to actions
            # This is the bit that's different to a standard view
            for method, action in actions.items():     
                handler = getattr(self, action)  # 通过反射获取请求方式
                setattr(self, method, handler) # 绑定到视图函数中的方法

            if hasattr(self, 'get') and not hasattr(self, 'head'):
                self.head = self.get

            self.request = request
            self.args = args
            self.kwargs = kwargs

            # And continue as usual
            return self.dispatch(request, *args, **kwargs)

        # take name and docstring from class
        update_wrapper(view, cls, updated=())

        # and possible attributes set by decorators
        # like csrf_exempt from dispatch
        update_wrapper(view, cls.dispatch, assigned=())

        # We need to set these on the view function, so that breadcrumb
        # generation can pick out these bits of information from a
        # resolved URL.
        view.cls = cls
        view.initkwargs = initkwargs
        view.suffix = initkwargs.get('suffix', None)
        view.actions = actions
        return csrf_exempt(view)

    def initialize_request(self, request, *args, **kwargs):
        """
        Set the `.action` attribute on the view,
        depending on the request method.
        """
        request = super(ViewSetMixin, self).initialize_request(request, *args, **kwargs)
        method = request.method.lower()
        if method == 'options':
            # This is a special case as we always provide handling for the
            # options method in the base `View` class.
            # Unlike the other explicitly defined actions, 'metadata' is implicit.
            self.action = 'metadata'
        else:
            self.action = self.action_map.get(method)
        return request

    def reverse_action(self, url_name, *args, **kwargs):
        """
        Reverse the action for the given `url_name`.
        """
        url_name = '%s-%s' % (self.basename, url_name)
        kwargs.setdefault('request', self.request)

        return reverse(url_name, *args, **kwargs)
</code>
```

#### 第四类 ModelViewSet 继承了这个类，ModelViewSet继承了四个混入类和一个泛类，

将会获得增删改查的所有方法。

```
<code class="python">class IndexView(ModelViewSet):
    queryset = models.UserInfo.objects.all()
    serializer_class = UserInfoSerializer
    
</code>
```

```
<code class="python">
class ModelViewSet(mixins.CreateModelMixin,
                   mixins.RetrieveModelMixin,
                   mixins.UpdateModelMixin,
                   mixins.DestroyModelMixin,
                   mixins.ListModelMixin,
                   GenericViewSet):
    """
    A viewset that provides default `create()`, `retrieve()`, `update()`,
    `partial_update()`, `destroy()` and `list()` actions.
    """
    pass
</code>
```

使用类视图的好处：
1、我们将各个HTTP请求方法之间，做了更好的分离。
2、可以很容易地，组成可重复使用的行为。
