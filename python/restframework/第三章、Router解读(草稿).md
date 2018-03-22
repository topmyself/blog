# Router
从 url 注册一章，我们了解到，REST framework 可能是通过 Router 注册的 ViewSet，最终转化成了 django 的 views 下的函数。那么事实是不是这样呢？我们来详细阅读一下Router 函数

# Router
## BaseRouter
BaseRouter是基类，我们看到BaseRouter的 register 与 urls，正好是上一章讲 url 注册时，由 SimpleRouter，转化为 urlpatterns 时，调用的两个函数。

```
class BaseRouter(object):
    def __init__(self):
        self.registry = []

    def register(self, prefix, viewset, base_name=None):
        if base_name is None:
            base_name = self.get_default_base_name(viewset)
        self.registry.append((prefix, viewset, base_name))

    @property
    def urls(self):
        if not hasattr(self, '_urls'):
            self._urls = self.get_urls()
        return self._urls
```
其中我们发现 register函数，最后都将 url 放在了 registry 的列表中，而 urls，最后调用的是 get_urls函数，BaseRouter 未实现get_urls函数，应该是留给子类实现的。
那么现在我们需要2点需要注意，第一是列表 registry 中的数据是怎么用的，二是 get_urls 函数在子类中的实现。
例子中，使用的是 SimpleRouter，那么我们就去找 SimpleRouter 看一下。

## SimpleRouter

```
def get_urls(self):
        """
        Use the registered viewsets to generate a list of URL patterns.
        """
        ret = []

        for prefix, viewset, basename in self.registry:
            lookup = self.get_lookup_regex(viewset)
            routes = self.get_routes(viewset)

            for route in routes:

                # Only actions which actually exist on the viewset will be bound
                mapping = self.get_method_map(viewset, route.mapping)
                if not mapping:
                    continue

                # Build the url pattern
                regex = route.url.format(
                    prefix=prefix,
                    lookup=lookup,
                    trailing_slash=self.trailing_slash
                )
                view = viewset.as_view(mapping, **route.initkwargs)
                name = route.name.format(basename=basename)
                ret.append(url(regex, view, name=name))

        return ret
```
我们找到了get_urls函数，发现这个函数从 self.registry, 而 registry 是我们之前已经阅读的，SimpleRouter 调用 register 后，生成的列表，这里面包含了路由匹配的 pattern 与 viewset。

```
for prefix, viewset, basename in self.registry
```
我们看看这个函数，在拿到 viewset 之后，做了什么东西。

```
view = viewset.as_view(mapping, **route.initkwargs)
name = route.name.format(basename=basename)
ret.append(url(regex, view, name=name))
```
上面的代码，viewset 调用了 as_view 函数，并返回 view，而这个 view 最后作为参数传入 url 中，而 django 中,url 的 view 是作为匹配路由后的回调函数的，这里也就说明，当匹配路由后，会调用 view 函数进行处理。我们继续深入阅读代码,as_view 生成的这个 view 函数，到底做了什么。请转到 viewset 章查看。

如果你仔细看，还会看到，在 get_urls 中，还有

```
mapping = self.get_method_map(viewset, route.mapping)
```
这个处理又是干什么的，我们稍后就会给出答案
