# 位置
ViewSet 定义在REST framework 目录下的 viewset.py 文件里面，打开文件后，我发现 ViewSet 类，同时，还有GenericViewSet，ModelViewSet等。我忘记我在 url 注册的时候，注册的 ViewSet子类，是继承自 ViewSet，还是 GenericViewSet 了，我回去查看了一下，确认是 ViewSet 类，那么我们就需要解读 ViewSet类了。

# ViewSet
在上一章 Router 里面，我们确认了 route 的 get_urls 函数是调用了ViewSet 子类的 as_view 函数，并返回了 view 函数。那么我直接找 ViewSet 的 as_view 函数

```
class ViewSet(ViewSetMixin, views.APIView):
    """
    The base ViewSet class does not provide any actions by default.
    """
    pass
```
发现 ViewSet 类，啥都没有，是继承在 ViewSetMixin 与 views 文件里面的 APIView 的，我们直接查看这两个父类有没有 as_view 函数
经过查看，我发现ViewSetMixin和APIView都有 as_view 函数，这个是一个多继承的问题，python 新式类的多继承，是采用从左到右的广度优先策略，也就是先查找 ViewSetMixin，再查找 APIView，最后我们调用的是ViewSetMixin 的 as_view 函数。

# ViewSetMixin 类

```
@classonlymethod
    def as_view(cls, actions=None, **initkwargs):
        """
        Because of the way class based views create a closure around the
        instantiated view, we need to totally reimplement `.as_view`,
        and slightly modify the view function that is created and returned.
        """
        # The suffix initkwarg is reserved for identifying the viewset type
        # eg. 'List' or 'Instance'.
        cls.suffix = None

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
                handler = getattr(self, action)
                setattr(self, method, handler)

            # Patch this in as it's otherwise only present from 1.5 onwards
            if hasattr(self, 'get') and not hasattr(self, 'head'):
                self.head = self.get

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
        view.suffix = initkwargs.get('suffix', None)
        return csrf_exempt(view)
```
我们在阅读 as_view函数的时候，里面定义了一个 view 函数，并最终返回这个 view 函数。这个 view 函数其实就是 django 中匹配路由后，回调的函数, view 函数里面，调用了 dispatch()函数，这个函数，就正式进入 rest_framework 的处理流程了。  
但是 ViewSetMixin 类，并未提供dispatch 函数，但是调用 dispatch 这个对象，是 ViewSet 与 APIView 子类的对象，ViewSet 类下没有 dispatch 函数，那么理所当然，去 APIView 类中找 dispatch.
APIView 类在文件 views.py 中，我们去下一章，去看看在APIView 里面发生了什么。





# ModelViewSet

```
class ModelViewSet(mixins.CreateModelMixin,
                   mixins.RetrieveModelMixin,
                   mixins.UpdateModelMixin,
                   mixins.DestroyModelMixin,
                   mixins.ListModelMixin,
                   GenericViewSet):
```
我们先大概浏览一下

```
class GenericViewSet(ViewSetMixin, generics.GenericAPIView):
```
GenericViewSet，发现该函数是继承自 ViewSetMixin 类与 generics.GenericAPIView，ViewSetMixin，我们前面已经讲过这个类，那么我们继续看一下 generics.GenericAPIView都干了什么

```
class GenericAPIView(views.APIView):
```
除了继承自 views.APIView以外，还有一些函数，我们先不管这些函数是做什么的，大概先有一个印象即可

```
def get_queryset(self):

def get_object(self):

def get_serializer(self, *args, **kwargs):
```

GenericAPIView 看完了，我们再看ModelViewSet的其他父类。
mixin 下的

```
mixins.CreateModelMixin,
mixins.RetrieveModelMixin,
mixins.UpdateModelMixin,
mixins.DestroyModelMixin,
mixins.ListModelMixin
```

去下一章，查看 mixin 吧
