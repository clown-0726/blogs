# Python logging 模块详解

> 发布日期：2024-06-06 14:51:23

[![pkYsjL8.jpg](https://s21.ax1x.com/2024/06/06/pkYsjL8.jpg)](https://imgse.com/i/pkYsjL8)

Python 的 `logging` 模块提供了一个强大而灵活的日志系统。它是 Python 标准库的一部分，因此可以在任何 Python 程序中使用。`logging` 模块提供了许多有用的功能，包括日志消息的级别设置、日志消息的格式设置、将日志消息输出到不同的目标，以及处理复杂的日志系统配置。

## logging 组成部分

在 logging 模块中主要有四部分：Logger、Handler、Filter 和 Formatter。下面分别对这四部分进行介绍。

#### Logger

Logger 就是程序可以直接调用的一个日志接口，可以直接向 Logger 写入日志信息。但是 Logger 并不是直接实例化使用的，而是通过 `logging.getLogger(name)` 来获取对象，事实上 Logger 对象是单例模式，并且 logging 是多线程安全的，也就是在程序的任何地方使用的同名的 Logger 都指向了一个实例。

#### Handler

Handler 作用于 Logger，是真正的日志处理程序，一个 Logger 可以配置多个 Handler。同样的，一个 Handler 也可以作用在多个 Logger 上。每个 Handler 同样有一个日志级别，也就是说 Logger 可以根据不同的日志级别将日志传递给不同的 Handler，当然也可以相同的级别传递给多个 Handler 这就根据需求来灵活的设置了。

Handler 既有系统定义的，比如将日志输出到标准输出 stdout。也可以自定义日志处理器，比如将日志发送到邮件或者第三方的日志存储介质等。

#### Filter

Filter 提供了更细粒度的判断，来决定日志是否需要打印。原则上 Handler 获得一个日志就必定会根据日志级别进行统一处理，但是如果 Handler 拥有一个 Filter 的时候就可以对日志进行额外的处理和判断。例如 Filter 能够对来自特定源的日志进行拦截甚至进行修改（包括日志级别的修改，修改后再进行级别判断）。

Logger 和 Handler 都可以安装 Filter 甚至可以安装多个 Filter 串联起来。

#### Formatter

Formatter 指定了最终某条记录打印的格式。Formatter 会将传递来的信息拼接成一条具体的字符串，默认情况下 Formatter 只会将信息 `%(message)s` 直接打印出来。Formatter 中有一些自带的属性可以使用，如下表格：

[![pkYsOQP.png](https://s21.ax1x.com/2024/06/06/pkYsOQP.png)](https://imgse.com/i/pkYsOQP)

注意的是一个 Handler 只能拥有一个 Formatter。

## 日志级别

#### 作用

日志级别是一个用于控制和过滤日志消息的重要工具。日志级别可以帮助更有效地管理日志，只看到需要的信息，而忽略不必要的细节。

这个功能在开发和运行大型程序时非常有用。例如，在开发过程中，你可能需要 DEBUG 级别的日志来找出问题。但是在生产环境中，这么详细的日志可能会占用大量的磁盘空间，而且大部分信息可能都是不必要的。因此，在生产环境中，你可能只需要 WARNING 或者 ERROR 级别的日志。

#### 级别

在记录日志时, 日志消息都会关联一个级别（“级别”本质上是一个非负整数）。系统默认提供了6个级别，它们分别是：

| 级别     | 对应值 |
| -------- | ------ |
| CRITICAL | 50     |
| ERROR    | 40     |
| WARNING  | 30     |
| INFO     | 20     |
| DEBUG    | 10     |
| NOTSET   | 0      |

## 继承关系和处理流程

#### Logger 继承关系

Logger 对象是有父子关系的，当没有父 Logger 对象时它的父对象是 root，当拥有父对象时父子关系会被修正。举个例子，`logging.getLogger("abc.xyz")` 会创建两个 Logger 对象，一个是 abc 父对象，一个是 xyz 子对象，同时 abc 没有父对象，所以它的父对象是 root。但是实际上 abc 是一个占位对象（虚的日志对象），可以没有 Handler 来处理日志。但是 root 不是占位对象，如果某一个日志对象打日志时，它的父对象会同时收到日志，所以有些使用者发现创建了一个 Logger 对象时会打两遍日志，就是因为他创建的 Logger 打了一遍日志，同时 root 对象也打了一遍日志。

**1）level的继承**

子 Logger 写日志时，优先使用本身设置了的 level；如果没有设置，则逐层向上级父 Logger 查询，直到查询到为止。最极端的情况是，使用 root Logger 的默认日志级别 logging.WARNING。

参考源码：

```python
def getEffectiveLevel(self):
    """
    Get the effective level for this logger.

    Loop through this logger and its parents in the logger hierarchy,
    looking for a non-zero logging level. Return the first one found.
    """
    logger = self
    while logger:
        if logger.level:
            return logger.level
        logger = logger.parent
    return NOTSET
```

**2）Handler 的继承**

先将日志对象传递给子 Logger 的所有 Handler 处理，处理完毕后，如果该子 Logger 的 propagate 属性没有设置为 False，则将日志对象向上传递给第一个父 Logger，该父 Logger 的所有 Handler 处理完毕后，如果它的 propagate 也没有设置为 False，则继续向上层传递，以此类推。最终的状态，要么遇到一个 Logger，它的 propagate 属性设置为了 False；要么一直传递直到 root Logger 处理完毕。

注意，Handler 不是真正的（类）继承，只是“行为上的继承”，也就是子 Logger 并没有没有绑定父类的 Handler。

参考源码：

```python
def callHandlers(self, record):
    """
    Pass a record to all relevant handlers.

    Loop through all handlers for this logger and its parents in the
    logger hierarchy. If no handler was found, output a one-off error
    message to sys.stderr. Stop searching up the hierarchy whenever a
    logger with the "propagate" attribute set to zero is found - that
    will be the last logger whose handlers are called.
    """
    c = self
    found = 0
    while c:
        for hdlr in c.handlers:
            found = found + 1
            if record.levelno >= hdlr.level:
                hdlr.handle(record)
        if not c.propagate:
            c = None  # break out
        else:
            c = c.parent
    if (found == 0) and raiseExceptions and not self.manager.emittedNoHandlerWarning:
        sys.stderr.write("No handlers could be found for logger"
                         " \"%s\"\n" % self.name)
        self.manager.emittedNoHandlerWarning = 1
```

#### 日志记录的处理流程

[![pkYsXsf.png](https://s21.ax1x.com/2024/06/06/pkYsXsf.png)](https://imgse.com/i/pkYsXsf)

在一个给定继承关系的日志配置中，如果其中任何一个 Logger 进行了日志记录，会经历下面几个步骤：

- 首先这条记录会被 Logger 本身的 Handler 进行处理。
  - 检查日志级别是否满足最低要求
  - 是否有 Filter 进行更细粒度的处理
  - 用什么样的 Formatter 作为打印格式
- 默认 Logger 的 propagate 值为 True，也就是在自己进行日志处理的同时，也会把这条日志代理到其父 Logger 进行处理，一直会被代理到 root Logger。

参考下面代码：

```python
import logging

# root logger
logging.basicConfig(level=logging.INFO)

# parent logger
logger_parent = logging.getLogger("parent")
handler = logging.StreamHandler()
handler.setLevel(logging.CRITICAL)  # Set the handler's level to CRITICAL
logger_parent.addHandler(handler)
logger_parent.setLevel(level=logging.CRITICAL)

# child logger
logger_child = logging.getLogger("parent.child")
logger_child.addHandler(logging.StreamHandler())
logger_child.setLevel(level=logging.INFO)

if __name__ == "__main__":
    logger_child.info("Hi")
```

其输出结果为：

```tex
Hi
INFO:parent.child:Hi
```

在上面代码中有三个日志记录器，其继承关系分别是 logger_child 继承 logger_parent 继承 root。

每个日志记录器上设置的日志级别，比如代码 `logger_parent.setLevel(level=logging.CRITICAL)` 不会影响记录器对日志的代理关系，其影响的是这个记录器在使用时是否处理程序中的日志记录。比如 logger_parent 在使用的时候会忽略 CRITICAL 级别之下的日志。

在上述程序中：

- Hi 这条日志首先会被 logger_child 的 Handler 进行处理。
- 然后这条记录会被代理到 logger_parent，logger_parent 的 handler 会对这条日志做处理，由于 logger_parent 的 Handler 是 CRITICAL 级别的，因此忽略这条日志。
- 接下来这条日志会继续被代理到 root，root 的 Handler 接下来会继续处理这条日志。

因此日志是否会被向上代理，取决于 Logger 上的 propagate 属性。

## 使用实例

#### 基本日志打印

在下面的例子中，默认的日志级别是 WRANING，因此只有最后一个日志被打印出来。

```python
import logging

logging.debug('this is debug message')
logging.info('this is info message')
logging.warning('this is warning message')

# 打印结果：WARNING:root:this is warning message
```

#### root 日志配置

在下面的例子中，root 日志记录器可以通过 logging.basicConfig 进行配置，比如配置其默认的日志级别，Handler 或者输出格式等。

```python
import logging

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(name)s - %(message)s')

if __name__ == '__main__':
    logging.debug('this is debug message')
    logging.info('this is info message')
    logging.warning('this is warning message')

'''''
结果：
2024-06-06 17:10:43,873 - root - this is debug message
2024-06-06 17:10:43,873 - root - this is info message
2024-06-06 17:10:43,873 - root - this is warning message
'''
```

#### 将日志同时输出到文件和 stdout

在下面的配置中，定义了一个文件 Handler 和标准输出 Handler，并将其应用到 logger 日志处理器上，这样日志会同时被输出到文件和标准输出中。

如果你用 FileHandler 写日志，文件的大小会随着时间推移而不断增大。为了避免这种情况出现，可以在你的生成环境中使用 RotatingFileHandler 替代 FileHandler 做日志回滚。

```python
import logging

logger = logging.getLogger(__name__)
logger.setLevel(level=logging.INFO)

# 文件 Handler
file_handler = logging.FileHandler("log.txt")
file_handler.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
file_handler.setFormatter(formatter)

# 标准输出 Handler
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)

# 在 logger 中应用这两个 Handler
logger.addHandler(file_handler)
logger.addHandler(console_handler)

logger.info("Start print log")
logger.debug("Do something")
logger.warning("Something maybe fail.")
logger.info("Finish")
```

#### 使用适配器扩展

在下面的例子中通过 logging 提供的 LoggerAdapter 方法可以对日志的字段进行扩展，输出的时候通过适当的 Formatter 进行扩展字段的输出。

```python
import logging

format_str = '%(levelname)s %(filename)s %(document_id)s %(message)s'
logging.basicConfig(level=logging.INFO, format=format_str)

logger = logging.getLogger(__name__)

extra = {'document_id': 'doc id'}
logger = logging.LoggerAdapter(logger, extra)

logger.info({'name': 'Xiao Ming'})

# INFO example_use_adapter.py doc id {'name': 'Xiao Ming'}
```

#### 捕捉异常并使用 traceback 记录

当程序出现错误的时候，在使用 Logger 进行记录的时候通过设置参数 `exc_info=True` 可以在日志中记录详细的报错信息。

```python
import logging

logger = logging.getLogger(__name__)

try:
    open('/path/to/does/not/exist', 'rb')
except Exception as e:
    logger.error('Failed to open file', exc_info=True)

'''
Failed to open file
Traceback (most recent call last):
  File "/Users/crown/Projects/python101/playground/logging_watchtower/example_traceback.py", line 6, in <module>
    open('/path/to/does/not/exist', 'rb')
FileNotFoundError: [Errno 2] No such file or directory: '/path/to/does/not/exist'
'''
```

#### 通过配置文件

如果日志的配置比较复杂，可以考虑通过外置配置文件的方式进行 logging 配置，logging 模块支持 json 格式和 yaml 两种格式的配置文件，通过配置文件可以更清晰的进行负责日志的配置。

下面是通过 json 配置文件的方式对 logging 进行配置。

```json
{
  "version": 1,
  "disable_existing_loggers": false,
  "formatters": {
    "simple": {
      "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
    }
  },
  "handlers": {
    "console": {
      "class": "logging.StreamHandler",
      "level": "DEBUG",
      "formatter": "simple",
      "stream": "ext://sys.stdout"
    },
    "info_file_handler": {
      "class": "logging.handlers.RotatingFileHandler",
      "level": "INFO",
      "formatter": "simple",
      "filename": "info.log",
      "encoding": "utf8"
    }
  },
  "loggers": {
    "my_module": {
      "level": "ERROR",
      "handlers": [
        "info_file_handler"
      ],
      "propagate": "no"
    }
  },
  "root": {
    "level": "INFO",
    "handlers": [
      "console",
      "info_file_handler"
    ]
  }
}
```

在 python 中读取配置文件：

```python
{
  "version": 1,
  "disable_existing_loggers": false,
  "formatters": {
    "simple": {
      "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
    }
  },
  "handlers": {
    "console": {
      "class": "logging.StreamHandler",
      "level": "DEBUG",
      "formatter": "simple",
      "stream": "ext://sys.stdout"
    },
    "info_file_handler": {
      "class": "logging.handlers.RotatingFileHandler",
      "level": "INFO",
      "formatter": "simple",
      "filename": "info.log",
      "encoding": "utf8"
    }
  },
  "loggers": {
    "my_module": {
      "level": "ERROR",
      "handlers": [
        "info_file_handler"
      ],
      "propagate": "no"
    }
  },
  "root": {
    "level": "INFO",
    "handlers": [
      "console",
      "info_file_handler"
    ]
  }
}
```

## 参考文档

[1] python基础学习十 logging模块详细使用 https://www.cnblogs.com/louis-w/p/8567434.html
[2] github django logging https://github.com/django/django/blob/1586a09b7949bbb7b0d84cb74ce1cadc25cbb355/django/utils/log.py#L18
