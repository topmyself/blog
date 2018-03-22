# 导读
我们知道http 服务是在接收到 http 报文时，有 server 端程序解析 http 报文，通过 http 报文的 uri 来决定该由哪个视图处理。
那么在django 的基础上，对于要弄清楚REST framework 的流程来讲，弄清楚 framework 的入口函数，至关重要

# url route注册
在讲REST framework 的 route之前，我们先复习一下 django 是如何注册 url 的。

django工程在 settings.py 中定义了 ROOT_URLCONF 变量，该变量能指导框架通过哪个 URL配置来对访问的uri进行匹配，从而找到能够匹配 uri 的视图进行处理。

```
from django.conf.urls import url
from . import views

urlpatterns = [
        url(r'^articles/2003/$', views.special_case_2003),
        url(r'^articles/([0-9]{4})/$', views.year_archive),
        url(r'^articles/([0-9]{4})/([0-9]{2})/$', views.month_archive),
        url(r'^articles/([0-9]{4})/([0-9]{2})/([0-9]+)/$', views.article_detail),

]
```
从 django 的 url 处理样例中，就能看出，关键点是 urlpatterns。

## Routers
我们先看一个例子：

```
from rest_framework import routers

router = routers.SimpleRouter()
router.register(r'users', UserViewSet)
router.register(r'accounts', AccountViewSet)
urlpatterns = router.urls
```
上面代码是 RESST framework 中 url 的样例写法，我们从中能够看到，REST framework 增加了 routers 的模块，并且定义了不同的 Router，本例中，使用的是 SimpleRouter.
而在 SimpleRouter 的对象注册 url 后，会将 router.urls 赋值给 urlpatterns。  
而 urlpatterns 是 django 中处理的，这是 django 与 REST framework 的第一个交汇点。

我们已经知道，django框架在处理请求时，通过 uri 识别后，能找到views 下的一个处理函数。
而REST framework 注册时，是注册的 ViewSet 类，那么必然，中间有一个由 ViewSet 类转换成 view 函数的过程，这个过程，我们来参考一下 router 解析

```
```
```
```
