# 去掉特殊空格(\t \v \f \xa0 \u0020 \u3000 \u00A0 &nbsp;)

> 发布日期：2021-10-07 23:39:34

我们在做爬虫的时候，经常回遇到一些特殊的空格形式，如果不对这些空格进行妥善的处理，很可能会污染我们的数据。

<!--more-->

## 不同的空格种类
一般我们所认识的正常空格为 `0x20` 这种也就是我们直接在键盘上敲击的空格。但是还有很多其他的空格形式。
- `\t`：水平制表符
- `\v`：垂直制表符
- `\f`：换页符
- `\xa0`：不间断空白符
- `\u0020`：半角空格（英文符号），代码中常用的
- `\u3000 `：全角空格（中文符号），中文文章中使用
- `\u00A0`：不间断空格，主要用在office中，让一个单词在结尾处不会换行显示
- `&nbsp;`：HTML 中的空格表示形式


## 去掉空格的两种方法
使用正则表达式去掉空格
```
import re
re.sub(r'\s', '', msg)
```

借助 unicodedata 这个库，这个库里有一个 normalize 函数，可以将其他特殊的空格转换为标准的空格。
```
import unicodedata as ucd

ucd.normalize('NFKC', msg).replace(' ', '') 
```


## REFERENCE
[1] python剔除空格\u3000: https://zhuanlan.zhihu.com/p/348461462  
[2] 三种空格unicode(\u00A0,\u0020,\u3000)表示的区别: https://www.jianshu.com/p/4317e3749a13  
[3] 网页爬虫中\xa0、\u3000等字符的解释及去除: https://blog.csdn.net/pengjunlee/article/details/104674623/