# 回顾
我们在 Router 解读那一章，最后提到了一个 method 的问题，那现在这一章，我们就来看一下，这个 method 是怎么解读的。

# 代码事例

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
依然回到 get_urls 函数中

```
lookup = self.get_lookup_regex(viewset)
routes = self.get_routes(viewset)
```
在遍历 registry列表的时候，通过 viewset获取了 lookup 与 routes。

### lookup

```
def get_lookup_regex(self, viewset, lookup_prefix=''):
        """
        Given a viewset, return the portion of URL regex that is used
        to match against a single instance.

        Note that lookup_prefix is not used directly inside REST rest_framework
        itself, but is required in order to nicely support nested router
        implementations, such as drf-nested-routers.

        https://github.com/alanjds/drf-nested-routers
        """
        base_regex = '(?P<{lookup_prefix}{lookup_url_kwarg}>{lookup_value})'
        # Use `pk` as default field, unset set.  Default regex should not
        # consume `.json` style suffixes and should break at '/' boundaries.
        lookup_field = getattr(viewset, 'lookup_field', 'pk')
        lookup_url_kwarg = getattr(viewset, 'lookup_url_kwarg', None) or lookup_field
        lookup_value = getattr(viewset, 'lookup_value_regex', '[^/.]+')
        return base_regex.format(
            lookup_prefix=lookup_prefix,
            lookup_url_kwarg=lookup_url_kwarg,
            lookup_value=lookup_value
        )
```
该函数从viewset中获取 lookup_field、lookup_url_kwarg、lookup_value_regex 属性，然后格式化成 base_regex。

### routes

```
known_actions = flatten([route.mapping.values() for route in self.routes if isinstance(route, Route)])
```
self.routes 是 SimpleRouter 类初始化的一个变量

```
    routes = [
        # List route.
        Route(
            url=r'^{prefix}{trailing_slash}$',
            mapping={
                'get': 'list',
                'post': 'create'
            },
            name='{basename}-list',
            initkwargs={'suffix': 'List'}
        ),
        # Dynamically generated list routes.
        # Generated using @list_route decorator
        # on methods of the viewset.
        DynamicListRoute(
            url=r'^{prefix}/{methodname}{trailing_slash}$',
            name='{basename}-{methodnamehyphen}',
            initkwargs={}
        ),
        # Detail route.
        Route(
            url=r'^{prefix}/{lookup}{trailing_slash}$',
            mapping={
                'get': 'retrieve',
                'put': 'update',
                'patch': 'partial_update',
                'delete': 'destroy'
            },
            name='{basename}-detail',
            initkwargs={'suffix': 'Instance'}
        ),
        # Dynamically generated detail routes.
        # Generated using @detail_route decorator on methods of the viewset.
        DynamicDetailRoute(
            url=r'^{prefix}/{lookup}/{methodname}{trailing_slash}$',
            name='{basename}-{methodnamehyphen}',
            initkwargs={}
        ),
    ]

```
这个 routes 包含了2个 Route, 一个DynamicListRoute，一个DynamicDetailRoute.从直观上看，应该是2个 Route 分别提供了get,create 以及 retrieve, update, partial_update, delete两种格式
，DynamicListRoute 以及 DynamicDetailRoute 包含了 listroute 以及 detail route 两种格式

再往下阅读函数，可以发现通过 @detail_route 以及 @list_route定义的路由，不能使用已经存在的 method 名称，并定义了获取动态路由的函数。
这样，路由中就包含了通用的 list, create, update, retrieve, delete 等，还包含了 list_route以及 detail_route 定义的路由。

### get_method_map
生成 routes 后，就可以查看 viewset 类中是否有对应 method 的函数了。生成的 mapping，传入 as_view 函数，供后续使用。

传入 as_view 函数后，能看到函数设置了http 请求方法所对应的处理函数，比如http get 方法，就调用 list 函数。

#

# 注:
http_method_names
```
http_method_names = ['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']
```
我們會在函數中看到有 cls.http_method_names 的表达式，从代码中，一层一层的读取代码，发现 http_method_names 是 django 中 view, generic 里面的函数，内容就是 http 允许的方法
