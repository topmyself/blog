# 导读
在第五章 views 解读时，我们已经详细解读了 http 方法对应的处理函数，但是却没发现这些处理函数在哪。
从样例中，我们找到样例的 View 继承自 ModelViewSet，那么顺着 ModelViewSet，我们发现了 mixin 下的几个类，可能就是我们想要的结果。

这一章的例子有点多，我挑选2~3个样例解决，get 方法对应的 list函数(get 方法还可以对应 retrieve 函数，需要 url 控制)，post 方法对应的 create 函数，以及 delete 方法对应的 destroy 函数

# ListModelMixin

```
class ListModelMixin(object):
    """
    List a queryset.
    """
    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())

        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)

```
list 函数的主要流程就是调用 get_queryset, filter_queryset, 再分页 paginate_queryset，再通过 serializer 进行序列化，最后返回结果。

对于 queryset 的操作，我们发现 rest_framework 的样例中，就有queryset，那么我们到这里，其实已经完全将 rest_framework 的请求流程全部走通了，剩下的，我们再细化一些细节。


# CreateModelMixin

```
class CreateModelMixin(object):
    """
    Create a model instance.
    """
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)
        headers = self.get_success_headers(serializer.data)
        return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)

    def perform_create(self, serializer):
        serializer.save()

    def get_success_headers(self, data):
        try:
            return {'Location': data[api_settings.URL_FIELD_NAME]}
        except (TypeError, KeyError):
            return {}
```

同理，在调用 create 函数时，也是调用了 perform_create 函数，perform_create 函数，也是我们也样例，经常覆盖的函数，以正确处理业务逻辑。


# 结尾
这一章，我们连贯了 http 方法对应的函数处理，并了解到了这些处理函数到底做了些什么内容。目前的几章，我们已经将 rest_framework 的主要处理流程都走通了。接下来，我们看一下 queryset与serializer 如何对数据进行序列化。
其他旁枝末节的知识点，也将陆陆续续补充，但是已经不影响我们在rest_framework下的编程了。
