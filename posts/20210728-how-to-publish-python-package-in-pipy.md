# 如何在 pypi 上发布自己的 python 包

> 发布日期：2021-07-28 20:06:11

为什么人们喜欢视频学习？一篇[文章](https://samoburja.com/the-youtube-revolution-in-knowledge-transfer/)中解释到，人类学习效率最高的方式，不是"读书 + 思考"，而是"观察 + 模仿"。 前者需要较长时间的注意力投入，后者只需要短时间注意力，更符合人类的天性。

<!--more-->

![呼伦贝尔-186彩带河](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210728-how-to-publish-python-package-in-pipy/WechatIMG199.jpeg)

## 准备工作
- 首先在 [pipy](https://pypi.org/) 上注册自己的帐号，这一步不再赘述
- 本地电脑上安装有 python 的环境，这里推荐 3.x 以上的 python 环境
- 打包工作主要依赖 python 的一个叫 setuptools 的包来完成，因此安装这个包 `pip install setuptools`

## 组织代码结构
下面是我所发布的包的一个初始结构，这个项目是为公司写的，名字为 `orbitkit`。

```
orbitkit(也可以用 abc 来命名)
  ├── docs
  ├── examples
  ├─┬ orbitkit
  │ ├── file-extractor
  │ ├── util
  │ └── __init__.py
  ├── .gitignore
  ├── LICENSE
  ├── README.md
  └── setup.py
```

在这里我们的项目名字是 `orbitkit`，在 `orbitkit` 项目的根目录下有一个重名的文件夹。这里要注意，项目名字取什么其实不重要，如果你喜欢也可以用 `abc` 来命名，重要的是和 `setup.py` 同级的根目录下的 `orbitkit` 文件夹一定要是我们要发布的名字。

`docs` 和 `examples` 这两个文件夹顾名思义，一个是项目的文档，另一个是使用实例，能让用户更好的了解我们的项目。

`LICENSE` 是我们项目的许可证书，这里不做详细讨论。

这里重点讨论 `orbitkit` 文件夹，也就是我们的核心代码文件夹。python 和 java 不一样，并不是一个文件就是一个类，在 python 中一个文件中可以写多个类。我们推荐把希望向用户暴漏的类和方法都先导入到 `__init__.py` 中，并且用关键词 `__all__` 进行限定。下面是我的一个 `__init__.py` 文件。

```python
from orbitkit import util
from orbitkit.file_extractor.dispatcher import FileDispatcher

name = 'orbitkit'

__version__ = '0.0.8'
VERSION = __version__

__all__ = [
    'util',
    'FileDispatcher',
]
```
这样用户在使用的时候可以清楚的知道哪些类和方法是可以使用的，也就是关键词 `__all__` 所限定的类和方法。

另外，在写自己代码库的时候，即便我们可以使用相对导入，但是模块导入一定要从项目的根目录进行导入，这样可以避免一些在导入包的时候因路径不对而产生的问题。比如
```
from orbitkit.file_extractor.dispatcher import FileDispatcher
```
我就是是从 orbitkit 开始导入的。

## 准备 `setup.py` 文件
`setup.py` 是进行文件打包发布的配置文件，包括要发布的包名字，版本，license，描述，特性（classifier) 等等。必须要将其放到项目的根目录下，下面是我再为公司的写的一个包的 `setup.py` 配置文件。

```python
from setuptools import setup, find_packages
import orbitkit

setup(
    name='orbitkit',
    version=orbitkit.__version__,
    description=(
        'This project is only for Orbit Tech internal use.'
    ),
    long_description=open('README.md', 'r', encoding='utf-8').read(),
    long_description_content_type="text/markdown",
    author='Lilu Cao',
    author_email='lilu.cao@qq.com',
    maintainer='Lilu Cao',
    maintainer_email='lilu.cao@qq.com',
    license='MIT License',
    packages=find_packages(),
    platforms=["all"],
    url='https://github.com/clown-0726/orbitkit',
    classifiers=[
        # 'Development Status :: 4 - Beta',
        'Operating System :: OS Independent',
        'Intended Audience :: Developers',
        'License :: OSI Approved :: BSD License',
        'Programming Language :: Python',
        'Programming Language :: Python :: Implementation',
        'Programming Language :: Python :: 3',
        'Programming Language :: Python :: 3.4',
        'Programming Language :: Python :: 3.5',
        'Programming Language :: Python :: 3.6',
        'Programming Language :: Python :: 3.7',
        'Topic :: Software Development :: Libraries'
    ],
    install_requires=[
        "boto3 >= 1.17.0",
        "requests >= 2.12.1",
    ]
)

```
在上一个段落中我们看到我们程序的版本号定义了 `__init__.py` 文件中，因此在 `setup.py` 文件中我们就直接导入就好了。

## 准备 `README.md` 文件
一个好的 `README.md` 文件能让用户更好的了解我们的项目，并且快速上手使用。

`README.md` 文件会出现在两个地方，一个地方是自己的 github 仓库中介绍中，另一个是 pipy 的项目介绍中。现在很多优秀的代码库的 `README.md` 文件都写的非常好，这里推荐仿照成熟库的 `README.md` 文件来写自己的。

## 打包代码
可以使用下面命令打包一个源代码的包:
```
python setup.py sdist build
```

## 上传到 pipy
可以使用 `setuptools`，或者 `twine` 上传,推荐使用 `twine` 上次，因为使用 `setuptools` 上传时，你的用户名和密码是明文或者未加密传输，安全起见还是使用twine。
```
sudo pip install twine

twine upload dist/*
```
会提示你输入帐号和密码，输入完后上传即可.

## TODO
这里只是讨论来如何创建一个自己的 pipy 包，并没有提及测试项目，之后会补充如何使用 tox 进行测试。

## Reference
[1] 在Pypi上发布自己的Python包: https://www.cnblogs.com/sting2me/p/6550897.html  
[2] django-rest-framework: https://github.com/encode/django-rest-framework