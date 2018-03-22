# 前言
前一章，我们已经说明了django 是如何从 django 框架处理，走到 rest_framework 的处理的，这一章就要看一下，走到 rest_framework 框架后，调用的 dispatch 函数，到底做了什么东西，这里要揭晓了。


# APIView

```
def dispatch(self, request, *args, **kwargs):
        """
        `.dispatch()` is pretty much the same as Django's regular dispatch,
        but with extra hooks for startup, finalize, and exception handling.
        """
        self.args = args
        self.kwargs = kwargs
        request = self.initialize_request(request, *args, **kwargs)
        self.request = request
        self.headers = self.default_response_headers  # deprecate?

        try:
            self.initial(request, *args, **kwargs)
            # Get the appropriate handler method
            if request.method.lower() in self.http_method_names:
                handler = getattr(self, request.method.lower(),
                                  self.http_method_not_allowed)
            else:
                handler = self.http_method_not_allowed

            response = handler(request, *args, **kwargs)

        except Exception as exc:
            response = self.handle_exception(exc)

        self.response = self.finalize_response(request, response, *args, **kwargs)
        return self.response
```
dispatch 函数中，先是将 request 对象更换为 rest_framework中的 Request 对象，又做了一个initial初始化操作，然后从 request 的 method 方法，来查找对应的处理函数，最后再返回 reponse。这里我们有两个注意点，一个是我们都有哪些方法，这些方法是怎么赋值的？这个问题，我们将跟 Router 那一章最后的 method map 问题，一起探讨明白；二是我们调用这个方法，又走到了哪些流程，我们将在第六章、method 讲解清楚后，继续我们这个流程的分析。

# http 方法所对应的函数处理
如果你去第六章看了 method 的详解，那么你应该清楚了，默认情况下，http 方法所对应的函数处理是固定的，某个 url，比如get 方法，对应了 list 函数，post 方法，对应了create 函数。我们看一下 list 函数以及 create 函数都做了什么。
我们在 views.py 与 viewset.py 中，去找 list 函数，create 函数，但是这里貌似并未提供这些函数，没办法，我只能继续，找例子中，我用的是哪些类

```
class UserViewSet(ModelViewSet):
```
好，那我们去找 ModelViewSet
在 viewset解读处
