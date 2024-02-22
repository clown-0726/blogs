# 解决 Elastic Search 的 Fast Vector Highlighting (FVH) 策略无法高亮 nested 数据类型

> 发布日期：2022-04-08 10:40:10

## 前言

本文的目的不在介绍 ES 的三种高亮文本的策略，因此在阅读本文之前假设你已经

- 熟悉 ES 的使用
- 熟悉 ES 高亮的三种策略：Unified，Plain，FVH
- 需要对 nested field 的大文本进行高亮显示

## 提出问题

一般情况下，ES 的文本搜索结果高亮默认的策略是 Unified，这种高亮策略对于大部分文本是够用的，但是如果文本的长度比较长，比如达到几十兆甚至上百兆的文本数据，那么 ES 在 FETCH 阶段去解析文本并准备文本高亮的时候会非常耗时，并且很可能请求超时。从官网的文档中我们知道有一种 Fast Vector Highlighting (FVH) 策略，主要是针对大文本高亮准备的，也是一种典型的空间换时间的策略，但这需要我们文本属性的 `term_vector`  设置为 `with_positions_offsets`。

不过，当一切准备就绪，而你的文本字段是 nested 数据类型的时候，确发现搜素出的结果中没有了 highlight 这一项。因此提出问题：**FVH 策略如何高亮 nested field 的字段？**

## 解决办法

FHV 需要索引位置信息和偏移量信息。高亮操作是在顶层文档执行的，对于嵌套文档来说，位置和偏移量信息是保存在嵌套文档中的，因此高亮操作无法访问这些信息。

同时，在顶层高亮嵌套文档会导致错误的结果，比如下面例子：

```
# 创建一个文档的 mapping
PUT t
{
  "mappings": {
    "t": {
      "properties": {
        "foo": {
          "type": "nested",
          "properties": {
            "text": {
              "type": "text"
            },
            "num": {
              "type": "integer"
            }
          }
        }
      }
    }
  }
}

# 插入数据
PUT t/t/1
{
  "foo": [
    {
      "text": "brown",
      "num": 1
    },
    {
      "text": "cow",
      "num": 2
    }
  ]
}

# 搜索结果
GET t/_search
{
  "query": {
    "nested": {
      "path": "foo",
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "foo.text": "brown cow"
              }
            },
            {
              "match": {
                "foo.num": 1
              }
            }
          ]
        }
      }
    }
  },
  "highlight": {
    "fields": {
      "foo.text": {}
    }
  }
}
```

得到的高亮结果是 `brown` 和 `cow` ，然而 `cow` 是不应该是被高亮的。

但是通过 `inner_hits` 便可以得到正确的结果。

```
# 通过 inner_hits 进行搜索
GET t/_search
{
  "query": {
    "nested": {
      "path": "foo",
      "inner_hits": {      # 通过 inner_hits 进行搜索
        "_source": false,  # 不返回正文文本
        "highlight": {     # 高亮设置
          "fields": {
            "foo.text": {}
          }
        }
      },
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "foo.text": "brown cow"
              }
            },
            {
              "match": {
                "foo.num": 1
              }
            }
          ]
        }
      }
    }
  }
}
```

## 总结

对于嵌套文本来说，我们可以借助 inner_hits 和 FVH 来针对大文本进行快速的高亮显示。但是这也是一种通过空间换时间的做法，要针对具体的业务场景进行衡量选择。

## 参考文档

[1] [Search your data](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-your-data.html) » Highlighting https://www.elastic.co/guide/en/elasticsearch/reference/current/highlighting.html#highlighting  
[2] [Mapping parameters](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html) » term_vector https://www.elastic.co/guide/en/elasticsearch/reference/current/term-vector.html  
[3] 白话Elasticsearch62-进阶篇之Highlighting高亮显示 https://cloud.tencent.com/developer/article/1862466  
[4] ElasticSearch 高亮显示大文档搜索结果的策略和性能对比 https://cloud.tencent.com/developer/article/1491129  
[5] Elastic Search: Highlighting Text That Contains HTML Tags https://hamidmosalla.com/2021/06/06/elastic-search-highlighting-text-that-contains-html-tags/  
[6] FVH doesn't highlight nested fields https://github.com/elastic/elasticsearch/issues/19265  
[7] Elasticsearch highlight with nested objects https://stackoverflow.com/questions/15230580/elasticsearch-highlight-with-nested-objects
