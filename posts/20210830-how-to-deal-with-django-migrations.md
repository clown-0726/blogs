# 如何整理积重难返的 django db migrations 文件

> 发布日期：2021-08-30 16:46:23

你的代码应该是写给下一个开发者的情书。

<!--more-->

![也许是设计最合理门把手](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210830-how-to-deal-with-django-migrations/the-best-door-knob.jpeg)

最简单的方式当然是删除数据库然后重新生成，但本文主要针对在原有数据的基础上进行更新。

主要分以下三步，多数据库环境适用，执行之前建议有二：
1. 建议一个模块一个模块执行；
2. 执行之前保证数据库 django_migrations 中的迁移记录和模块下的 migrations 子文件夹下的迁移文件保持一致；

## django 数据库迁移原理（如已知晓，则跳过）
我们知道在 django 中执行了 `python manage.py makemigrations MODEL_NAME` 之后会在对应的模块下的 migrations 文件夹下生成相应的更改文件，但是这一步并不会对数据库产生任何更新。

当我们执行了 `python manage.py migrate MODEL_NAME` 之后，对数据库的更新有二：
1. 首先会根据上一步生成的更新文件进行表的创建或者更新；
2. 然后在数据库 django_migrations 中增加一条记录来记录所做的操作；

## 第一步：重置数据库 django_migrations 指定模块的记录

```
python manage.py migrate --fake MODEL_NAME zero
```

这一步主要是根据指定模块下的，migrations 子文件夹下的生成的更改记录，将数据库 django_migrations 中指定模块的（MODEL_NAME）迁移记录进行删除。

但是！这只是删除记录，并不会对已经生成的数据库结构做任何更改。

如果有多个环境，那么在进行下一步之前分别对多环境执行相同操作。

## 第二步：重新生成新的 migrations 文件

手动删除指定模块下的， migrations 子文件夹下的除了 `__init__.py` 文件之外的所有的数据库迁移文件。然后重新执行下面命令生成新的 `0001_initial` 记录。

```
python manage.py makemigrations MODEL_NAME
```

## 第三步：

通过执行下面命令重新在数据库 django_migrations 中生成新的迁移记录，这时候记录就只有一个了。

```
python manage.py migrate --fake-initial MODEL_NAME
```

如果有多个环境，那么分别对多环境执行相同操作。

## 参考文档

[1] Django重置migrations文件的方法步骤 https://www.zhangshengrong.com/p/rG1V53MvX3/  
[2] Django重置migrations文件的方法步骤 https://blog.csdn.net/sinat_33384251/article/details/109962333