# 导读
其实讲 queryset 完全是为了后面的 serializer，只有我们知道当前数据是什么样的，才能开始将数据进行序列化，最后变成我们想要的数据格式。

# 回忆GenericeViewSet
GenericeVewSet 类中，我们之前已经发现了 get_queryset 函数。

```
def get_queryset(self):
        assert self.queryset is not None, (
            "'%s' should either include a `queryset` attribute, "
            "or override the `get_queryset()` method."
            % self.__class__.__name__
        )

        queryset = self.queryset
        if isinstance(queryset, QuerySet):
            # Ensure queryset is re-evaluated on each request.
            queryset = queryset.all()
        return queryset
```
函数中，去找到 queryset 并返回该 queryset。queryset 就是就获取了 model 层的数据。

虽然获取了 queryset，但是我们好像并没看出来 queryset是个啥类型，那么我们不妨看看注释

```
Get the list of items for this view.
        This must be an iterable, and may be a queryset.
        Defaults to using `self.queryset`.
```

这段注释，很好的解释了 queryset 返回的数据类型，是一个 list,或者是一个 QuerySet，必须是可遍历的。
看不出来更多了，我们先假定返回的数据是 list，然后去看 serializer
