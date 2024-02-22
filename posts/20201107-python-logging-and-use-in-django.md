# Python logging 模块的使用

> 发布日期：2020-11-07 17:32:26

毫无疑问，一个好的日志输出在系统中是至关重要的，因为即便你很有经验，也没法保证一定不出错。

<!--more-->

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201107-python-logging-and-use-in-django/tech_company_structure.jpg)

<center><font face="黑体" size=2>世界顶尖 IT 公司的组织架构图 | 2020-06-07 | 拍摄于 iphone 7p</font></center>

<br/>

Python 中的 logging 模块提供了一组便利的函数，用来做简单的日志，它们是 debug()、 info()、 warning()、 error() 和 critical()。同时，django 用的也是 python 中标准的 logging 模块来进行日志记录的。

## logging 架构和调度过程

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201107-python-logging-and-use-in-django/how_python_logging_works.jpg)

上图是 logging 模块的主要模块和调度过程。我们可以将其主要分为4个部分，分别是 Logger，Handler，Filter 和 Formatter。

**Logger** 模块是日志的记录器，我们可以在程序的任意部分通过 `logging.getLogger` 得到一个 Logger，我们就可以用得到的 Logger 来进行日志的收集了。经由 Logger 收集到的日志会给到 Handler 模块进行日志的下一步处理。一个 Logger 模块可以调用多个 Handler 处理模块。

**Handler** 模块是日志的处理模块，负责将模块通过一定的输出格式（配置 Formatter模块）输出到配置好的存储介质（console，文件，Elastic Search，AWS Cloudwatch 等）之中。

**Formatter** 模块顾名思义是日志的格式化器。

**Filter** 模块介于 Logger 模块和 Handler 模块之间，用于日志的过滤，如果配置了此模块则由 Logger 收集到的日志必须符合 Filter 的配置规则才能进入 Handler 模块进行下一步的日志处理输出。

## logging 参数配置

#### level 级别

一个记录器是日志系统的一个实体，每一个记录器是一个已经命名好的可以将消息为进程写入的“桶”。
每一个记录器都会有一个日志等级，每个等级描述了记录器即将处理的信息的严重性，python定义了以下六个等级：

| 级别     | 值   | 描述                                                 |
| -------- | ---- | ---------------------------------------------------- |
| CRITICAL | 50   | 关键错误，描述已经发生的严重问题，程序已经没法执行   |
| ERROR    | 40   | 错误，描述已经发生的主要问题，程序某部分没法正常执行 |
| WARNING  | 30   | 警告消息，描述已经发生的小问题                       |
| INFO     | 20   | 通知消息，普通的系统信息列表内容                     |
| DEBUG    | 10   | 调试，出于调试目的的低层次系统信息                   |
| NOTSET   | 0    | 无级别                                               |

#### 处理器/记录器 关键字参数

| 关键字参数 | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| filename   | 将日志消息附加到指定文件名的文件                             |
| filemode   | 指定用于打开文件模式，文件打开方式，在指定了filename时使用这个参数，默认值为“a”还可指定为“w” |
| format     | 用于生成日志消息的格式字符串                                 |
| datefmt    | 用于输出日期和时间的格式字符串                               |
| level      | 设置记录器的级别                                             |
| propagate  | 可以基于每个记录器控制该传播。 如果您不希望特定记录器传播到其父项，则可以关闭此行为 |
| stream     | 提供打开的文件，用于把日志消息发送到文件。可以指定输出到`sys.stderr`，`sys.stdout`或者文件，默认为`sys.stderr` |

若同时列出了 filename 和 stream 两个参数，则 stream 参数会被忽略。

#### format 日志消息格式

| 格式           | 描述                               |
| -------------- | ---------------------------------- |
| %(name)s       | 记录器的名称                       |
| %(levelno)s    | 数字形式的日志记录级别             |
| %(levelname)s  | 日志记录级别的文本名称             |
| %(filename)s   | 执行日志记录调用的源文件的文件名称 |
| %(pathname)s   | 执行日志记录调用的源文件的路径名称 |
| %(funcName)s   | 执行日志记录调用的函数名称         |
| %(module)s     | 执行日志记录调用的模块名称         |
| %(lineno)s     | 执行日志记录调用的行号             |
| %(created)s    | 执行日志记录的时间                 |
| %(asctime)s    | 日期和时间                         |
| %(msecs)s      | 毫秒部分                           |
| %(thread)d     | 线程ID                             |
| %(threadName)s | 线程名称                           |
| %(process)d    | 进程ID                             |
| %(message)s    | 记录的消息                         |

#### 内置处理器

logging 模块提供了一些处理器，可以通过各种方式处理日志消息。

| 处理器                                         | 描述                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| logging.StreamHandler                          | 可以向类似与sys.stdout或者sys.stderr的任何文件对象(file object)输出信息 |
| logging.FileHandler                            | 将日志消息写入文件filename                                   |
| logging.handlers.DatagramHandler(host，port)   | 发送日志消息给位于制定host和port上的UDP服务器                |
| logging.handlers.HTTPHandler(host, url)        | 使用HTTP的GET或POST方法将日志消息上传到一台HTTP 服务器       |
| logging.handlers.RotatingFileHandler(filename) | 将日志消息写入文件filename                                   |
| logging.handlers.SocketHandler                 | 使用TCP协议，将日志信息发送到网络                            |
| logging.handlers.SysLogHandler                 | 日志输出到syslog                                             |
| logging.handlers.NTEventLogHandler             | 远程输出日志到Windows NT/2000/XP的事件日志                   |
| logging.handlers.SMTPHandler                   | 远程输出日志到邮件地址                                       |
| logging.handlers.MemoryHandler                 | 日志输出到内存中的制定buffer                                 |

注意：由于内置处理器还有很多，如果想更深入了解。可以查看官方手册。

## logging 配置实例

#### 使用流程图

```python
# 先获取记录器：
self.logger=logging.getLogger()
# 设置日志等级
self.logger.setLevel(level)
# 设置日志输出格式
fmt='%(asctime)s - %(pathname)s[line:%(lineno)d] - %(levelname)s: %(message)s'
format_str = logging.Formatter(fmt)
# 获取处理器
sh=logging.StreamHandler()
# 设置处理器的日志输出格式
sh.setFormatter(format_str)
# 添加到处理器中
self.logger.addHandler(sh)
```

#### 配置logging基本的设置，然后在控制台输出日志

```python
import logging
logging.basicConfig(level = logging.INFO,format = '%(asctime)s - %(name)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

logger.info("Start print log")
logger.debug("Do something")
logger.warning("Something maybe fail.")
logger.info("Finish")
```

#### 将日志写入到文件

```python
import logging
logger = logging.getLogger(__name__)
logger.setLevel(level = logging.INFO)
handler = logging.FileHandler("log.txt")
handler.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)

logger.info("Start print log")
logger.debug("Do something")
logger.warning("Something maybe fail.")
logger.info("Finish")
```

## django 中 logging 模块的使用

Django 中默认使用的也是 python 标准的 logging 模块，并且 django 扩展了很多处理器：

#### django提供的内置记录器

| 处理器             | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| django             | 在Django层次结构中的所有消息记录器。没有使用此名称发布消息，而是使用下面的记录器之一 |
| django.request     | 与请求处理相关的日志消息。5xx响应被提升为错误消息；4xx响应被提升为警告消息 |
| django.server      | 与由RunServer命令调用的服务器所接收的请求的处理相关的日志消息。HTTP 5XX响应被记录为错误消息，4XX响应被记录为警告消息，其他一切都被记录为INFO |
| django.template    | 与模板呈现相关的日志消息                                     |
| django.db.backends | 有关代码与数据库交互的消息。例如，请求执行的每个应用程序级SQL语句都在调试级别记录到此记录器 |

#### 配置实例

**配置信息需要在 setting.py 文件中进行添加**

```python
setting.py
DEBUG = True # 通过这种方式可以打开 DEBUG 模式

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'filters': {
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        }, # 针对 DEBUG = True 的情况
    },
    'formatters': {
        'standard': {
            'format': '%(levelname)s %(asctime)s %(pathname)s %(filename)s %(module)s %(funcName)s %(lineno)d: %(message)s'
        }, # 对日志信息进行格式化，每个字段对应了日志格式中的一个字段，更多字段参考官网文档
        # INFO 2016-09-03 16:25:20,067 /home/ubuntu/mysite/views.py views.py views get 29: some info...
    },
    'handlers': {
        'mail_admins': {
            'level': 'ERROR',
            'class': 'django.utils.log.AdminEmailHandler',
             'formatter':'standard'
        },
        'file_handler': {
             'level': 'DEBUG',
             'class': 'logging.handlers.TimedRotatingFileHandler',
             'filename': '/tmp/byod/byodadmin/byod.admin.log',
             'formatter':'standard'
        }, # 用于文件输出
        'console':{
            'level': 'INFO',
            'filters': ['require_debug_true'],
            'class': 'logging.StreamHandler',
            'formatter': 'standard'
        },
    },
    'loggers': {
        'django': {
            'handlers' :['file_handler', 'console'],
            'level':'DEBUG',
            'propagate': True # 是否继承父类的log信息
        }, # handlers 来自于上面的 handlers 定义的内容
        'django.request': {
            'handlers': ['mail_admins'],
            'level': 'ERROR',
            'propagate': False,
        },
    }
}
```

**配置好之后，在代码中可按照如下方法使用**

```python
import logging
logger = logging.getLogger("django") # 为loggers中定义的名称
logger.info("some info...")
```

## 总结

 毋庸置疑，在正式的项目或产品的开发中，系统的日志输出是至关重要的。它不仅仅能记录系统的运行过程，还能帮助我们快速定位到系统的故障和潜在的错误。另外，系统日志的记录也是进行后期用户行为分析的基础。

对于 python 生态圈的系统来说，logging 给我们提供了一个简单高效的日志处理方法，善用这个模块能提高项目获产品的研发迭代速度。

## References:

[1] Django配置日志输出、logging配置最详细大全、控制台日志全部输出到文件、日志/控制台console重定向到文件: https://blog.csdn.net/haeasringnar/article/details/82053714  
[2] Logging: https://docs.djangoproject.com/en/dev/topics/logging/#topic-logging-parts-formatters  
[3] python之logging模块详解: https://blog.csdn.net/mx472756841/article/details/89921888  
[4] python logging日志模块的使用: https://blog.csdn.net/pzqingchong/article/details/79777488  
[5] Django 日志模块 logging 的配置: https://blog.csdn.net/novostary/article/details/52424116?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param  
[6] python django日志的配置及使用: https://blog.csdn.net/myli_binbin/article/details/90678231

