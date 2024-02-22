# Django 和 DRF(Django Rest Framework) 下的全局异常处理

> 发布日期：2021-10-14 23:48:36

All good things must come to an end.

<!--more-->

Django 在 web 开发中往往会和 DRF 进行结合使用，DRF 有这自己的一套异常处理机制，我们可以对 DRF 中抛出的异常进行全局拦截，比如返回统一的 json 格式以便于前台进行处理。
但是我们同时在 Django 中使用后台 admin 管理数据的同时，不希望修改原有的异常返回。因此我们需要同时配置 Django 和 DRF 的全局异常来适应我们的需求。


## 异常种类

我们在写代码的时候一般会遇到以下两种异常种类：
- 【已知异常】可以预知的异常，大部分是我们在编程的时候手动抛出的异常。
- 【未知异常】我们完全没有意识到的异常，比如一些计算错误引发的异常。

## 异常返回格式

对于 Django admin 中的异常，我们仍然保持其原来的返回格式，对于其他异常，我们希望给前端返回下面的 json 格式以便于前端进行统一捕获。

```json
{
    "msg": "Sorry, ,we made a mistake!", 
    "error_code": 1000, 
    "request": "url"
}
```

## 异常来源

我们可以粗略的将程序中的异常来源分为以下几种：

- 继承 DRF 的 APIView 及其子类抛出的 APIException 类型的异常
- Django admin 抛出的异常
- 其他程序运行异常

## 定义全局异常处理

我们可以先定义一个 Django 的中间件来捕获 “Django admin 抛出的异常” 和 “其他程序运行异常”，
我们可以通过请求路径进行判断是否是 Django admin 的异常，如果是则直接返回 None，也就是按照原始格式抛出，
如果不是，则返回我们自己定的 json 格式的异常返回。

```python
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
别忘记在 Django setting 中添加中间件。

然后我们定义 DRF 的全局异常捕获，在调用父类的 exception_handler 方法后返回的 response 如果不为 None，说明是 APIException 类型的异常，
这时我们返回自己定义的异常格式，如果为 None，则直接返回 None，这时会被 Django 定义的 ExceptionGlobeMiddleware 捕捉到，同时仍然给客户端
返回自己定义的 json 格式。

```python
def globe_exception_handler(exc, context):
    """
        Below is the global exception handler of drf
        Http404 / PermissionDenied / APIException
    """

    # Call REST framework's default exception handler
    response = exception_handler(exc, context)
    request = context['request']

    # print exception to console
    print(traceback.format_exc())

    # Now add the HTTP status code to the response.
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

    # 如果 response 为 None 则直接触发上面的 ExceptionGlobeMiddleware
    return response
```

到这里，Django 结合 DRF 的统一全局异常处理就配置好了。

## REFERENCE
[1] Python Django，中间件，中间件函数，全局异常处理: https://blog.csdn.net/houyanhua1/article/details/85028983  
[2] Django-drf-全局异常处理: https://zhuanlan.zhihu.com/p/288814772  
[3] Django捕获所有异常的处理: https://blog.csdn.net/huangqiang1363/article/details/106472203/  
[4] Django rest framework 自定义Exception: https://www.cnblogs.com/ifykwf/p/7054790.html  
[5] Django rest framework自定义返回数据格式: https://www.jianshu.com/p/c0be24752584  
[6] Django中异常处理 Exceptions: https://www.cnblogs.com/itBlogToYpl/articles/11844547.html  
[7] django rest framework异常捕获: https://blog.csdn.net/weixin_46105038/article/details/117228013  
[8] Django REST framework异常捕获的全局配置: https://blog.csdn.net/Mr_w_ang/article/details/85017583  
[9] Exception handling in REST framework views: https://www.django-rest-framework.org/api-guide/exceptions/