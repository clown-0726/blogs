# Django 与 DRF 结合的全局异常处理方案

> 发布日期：2022-02-01 20:24:31

## 前言
Django 和 DRF(django rest framawork) 的结合在 python 后台中经常出现的组合。对于异常的全局处理，我们系统能有一个统一的解决方案，在开发环境能看到比较全的异常堆栈，而在生产环境能更好的给用户一个友好的提示，本文旨在提出一个统一个全局异常处理方案，仅供参考使用。

## 实现的目标
如果没有 DRF，我们只需要在 Django 中加一个中间件就可以解决全局异常的处理问题，但是 DRF 会帮我们处理一些异常并自动返回到客户端，因此我们要协调两者的异常处理策略。

同时我们希望能使用 Django 的 admin 进行一些后台的数据查看和修改，因此最好要保留 admin 的内部异常处理行为。

本文的目标如下：
- 保留 Django 自带的 admin 的异常处理行为
- 拦截 DRF 的异常并进行全局异常行为处理
- 拦截除 DRF 的异常之外的其他 Django 异常并进行全局异常行为处理

## DRF 全局异常拦截的解决思路
首先 DRF 的异常都是继承自 `APIException` 这个类的，并且 DRF 跑出的异常会被 `exception_handler` 这个异常处理函数拦截（这个函数的位置在 `/python3.7/site-packages/rest_framework/views.py`中）。

我们进一步查看这个函数的源码：

```python
def exception_handler(exc, context):
    """
    Returns the response that should be used for any given exception.

    By default we handle the REST framework `APIException`, and also
    Django's built-in `Http404` and `PermissionDenied` exceptions.

    Any unhandled exceptions may return `None`, which will cause a 500 error
    to be raised.
    """
    if isinstance(exc, Http404):
        exc = exceptions.NotFound()
    elif isinstance(exc, PermissionDenied):
        exc = exceptions.PermissionDenied()

    if isinstance(exc, exceptions.APIException):
        headers = {}
        if getattr(exc, 'auth_header', None):
            headers['WWW-Authenticate'] = exc.auth_header
        if getattr(exc, 'wait', None):
            headers['Retry-After'] = '%d' % exc.wait

        if isinstance(exc.detail, (list, dict)):
            data = exc.detail
        else:
            data = {'detail': exc.detail}

        set_rollback()
        return Response(data, status=exc.status_code, headers=headers)

    return None

```

通过这个函数的文档签名我们知道，DRF 会处理所有继承自 `APIException` 的异常类，并且还会额外的处理 Django 内置的 `Http404` 和 `PermissionDenied` 异常，并将这些异常的处理结果返回到前台。

如果不再这些处理范围之内，函数会返回 `None`，这时候会给 Django 抛出一个 500 的服务器错误异常。

DRF 支持单独配置异常处理函数，因此第一步现在 setting 中指定自定义的异常处理函数的位置：

```python
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'server.exception.exception_globe.globe_exception_handler'
}
```

然后我们定义自己的异常处理程序：
- 第一步，调用 DRF 自己的异常处理函数
- 第二步，对 DRF 拦截的异常进行处理
- 第三步，将其他异常抛给 Django 处理


```python
def globe_exception_handler(exc, context):
    """
        Below is the global exception handler of drf
        Http404 / PermissionDenied / APIException
    """

    # Call REST framework's default exception handler
    response = exception_handler(exc, context)
    request = context['request']

    # Exceptions form DRF and Django built-in `Http404` and `PermissionDenied`
    if response is not None:
        if isinstance(response.data, list):
            msg = '; '.join(response.data)
        elif isinstance(response.data, str):
            msg = response.data
        else:
            msg = 'Sorry, we make a mistake (*￣︶￣)!'

        ex_data = {
            "msg": msg,
            "error_code": 1000,
            "request": request.path
        }
        return JsonResponse(data=ex_data, status=response.status_code)

    # Exceptions from others
    # 如果 response 为 None 则直接触发上面的 ExceptionGlobeMiddleware
    return response
```

## Django 异常处理方案
从上一步的结果我们知道，DRF 处理不了的异常我们抛给了 Django，而 Django 支持通过定义中间件进行全局异常处理，因此接下来我们只需要定一个 Django 全局异常处理的中间件，并将中间件配置到 setting 文件中的 MIDDLEWARE 数组即可。

```python
try:
    from django.utils.deprecation import MiddlewareMixin  # Django 1.10.x
except ImportError:
    MiddlewareMixin = object  # Django 1.4.x - Django 1.9.x


class ExceptionGlobeMiddleware(MiddlewareMixin):
    """
        Below is the global exception handler of django
    """

    def process_exception(self, request, exception):
        # 直接抛出 django admin 的异常
        if str(request.path).startswith('/admin/'):
            return None

        # 捕获其他异常，直接返回 500
        ex_data = {
            "msg": "Sorry, we make a mistake (*￣︶￣)!",
            "error_code": 1000,
            "request": request.path
        }
        return JsonResponse(data=ex_data, status=500)
```

值得注意的是，我们可以在中间件的处理函数中拿到 `request` 对象，因此，我们可以通过这个对象拿到用户请求的 url，这样，我们通过判断 url 就可以得到那些请求是来自 Django 自带的 admin 的。参考代码如下：

```
# 直接抛出 django admin 的异常
if str(request.path).startswith('/admin/'):
    return None

```

## 总结
通过上述的配置，我们可以对完成 Django 结合 DRF 的全局异常处理，并且保留了 Django 自带 admin 的异常处理策略。

在实际的环境中，我们可以通过环境变量进行有选择的日志打印和记录。
