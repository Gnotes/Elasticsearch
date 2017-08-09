# 聚合

> 所谓聚合就是数学函数的使用，如：`avg` `max` `min` `sum` 等.

## 桶&指标

> 桶 简单来说就是`满足`特定`条件`的`文档`的`集合`,其本质意思可以解释为满足查询条件的所有数据；
如：原文中所说的:  
一个雇员属于 男性 桶或者 女性 桶 : 男，女就是条件

> 指标 其意思是我们要计算或查询的结果，如：男雇员的人数，平均年龄等...这些称之为指标

熟悉sql的同学可以这样来看 `select 指标 from table where 桶`，查询的条件就是`桶`（它将我们的数据分类，得到最终要查询的所有数据），而要查询的字段就是我们需要的`指标` 

## 常用聚合及参数

- 聚合函数 : `avg` `min` `max` `sum` `extended_stats`
- 直方图 : `histogram` 
- 根据时间统计 : `date_histogram` 
  - 时间参数 : `now` `year` `quarter` `month` `week` `day` `hour` ...

## 基本聚合

> 聚合通过`aggregations` 简写 `aggs`来标示

ie.官方汽车销售的例子

- `GET /cars/transactions/_search` 

*注* : 使用`elasticsearch-head`项目测试时，应使用**POST**来请求数据

```
{
   "size" : 0,
   "aggs": { ①
      "colors": { ②
         "terms": { ③
            "field": "color" ④
         },
         "aggs": { ⑤
            "avg_price": { ⑥
               "avg": { ⑦
                  "field": "price" ⑧
               }
            }
         }
      }
   }
}
```

- **size为0**，表示不查询匹配的数据，这个很重要，通常我们只需要查询的指标，而不需要查询的源数据,因此就不会在hits中返回数据  
- ① aggs 标示聚合
- ② 为聚合指定名称colors,相当于sql中的 `as` 定义别名
- ③ 划分桶
- ④ 指定桶划分条件，以color字段来划分
- ⑤ 添加聚合度量aggs层
- ⑥ 为聚合指定名称avg_price
- ⑦ 使用聚合函数avg
- ⑧ 指定要聚合的字段price

解释：上例意思为：查询`不同颜色(桶)`汽车的`平均(聚合函数avg)`销售`价格(聚合字段price)`

返回结果:

```json
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 16,
        "max_score": 0,
        "hits": []
    },
    "aggregations": { ①
        "colors": { ②
            "buckets": [ ③
                {
                    "key": "red", ④
                    "doc_count": 4, ⑤
                    "avg_price": { ⑥
                        "value": 32500 ⑦
                    }
                },
                {
                    "key": "blue",
                    "doc_count": 2,
                    "avg_price": {
                        "value": 20000
                    }
                },
                {
                    "key": "green",
                    "doc_count": 2,
                    "avg_price": {
                        "value": 21000
                    }
                }
            ]
        }
    }
}
```

- ① aggregations 聚合返回结果
- ② 聚合名称colors
- ③ buckets桶（上例中以颜色划分桶，而数据中共有三种颜色，因此出现了三个桶）
- ④ key桶名，即color的值
- ⑤ doc_count该桶拥有数据的条数
- ⑥ 聚合名称avg_price
- ⑦ value聚合计算出的结果

## 桶嵌套

> 聚合可以嵌套 一个 度量，桶可以嵌套在另一个桶中，如示例中查询 : 每个颜色的汽车制造商的分布

```json
{
   "size" : 0,
   "aggs": {
      "colors": {
         "terms": {
            "field": "color"
         },
         "aggs": {
            "avg_price": { 
               "avg": {
                  "field": "price"
               }
            },
            "make": { ①
                "terms": { ②
                    "field": "make" ③
                }
            }
         }
      }
   }
}
```

- ① 定义聚合名称
- ② 指定桶
- ③ 指定以make划分桶

返回结果:

```json
{
    "aggregations": {
        "colors": {
            "buckets": [
                {
                    "key": "red",
                    "doc_count": 4,
                    "make": { ①
                        "buckets": [
                            {
                                "key": "honda", ②
                                "doc_count": 3 ③
                            },
                            {
                                "key": "bmw",
                                "doc_count": 1
                            }
                        ]
                    },
                    "avg_price": {
                        "value": 32500
                    }
                }
            ]
        }
    }
}
```

- ① 聚合名称
- ② 桶的值
- ③ 该桶数据条数

其意思是 : 红色车有4辆，其销售均价为32500，其中，3辆是honda，1辆是bmw

## 聚合嵌套

在官网示例3中可以看到我们还可以为每个品牌的汽车计算最高价，最低价等...

```json
{
   "size" : 0,
   "aggs": {
      "colors": {
         "terms": {
            "field": "color"
         },
         "aggs": {
            "avg_price": { "avg": { "field": "price" }
            },
            "make" : {
                "terms" : {
                    "field" : "make"
                },
                "aggs" : { ①
                    "min_price" ② : { "min" ③: { "field": "price" ④} }, 
                    "max_price" : { "max": { "field": "price"} } 
                }
            }
         }
      }
   }
}
```

- ① 嵌套聚合
- ② min_price 指定聚合名称
- ③ min 聚合函数，求最小值
- ④ 通过字段price来计算聚合

返回结果 : 

```json
{
    "aggregations": {
        "colors": {
            "buckets": [
                {
                    "key": "red",
                    "doc_count": 4,
                    "make": {
                        "buckets": [
                            {
                                "key": "honda",
                                "doc_count": 3,
                                "min_price": { ①
                                    "value": 10000 ②
                                },
                                "max_price": {
                                    "value": 80000
                                }
                            }
                        ]
                    },
                    "avg_price": {
                        "value": 32500
                    }
                }
            ]
        }
    }
}
```

- ① 聚合名称
- ② 聚合结果

由此可知：4辆红色honda中最低价min_price为10000，最高价max_price为80000

## 直方图聚合histogram

> 直方图聚合可以理解为划分区间范围的聚合函数，如第4个示例中，计算每个售价区间内汽车的销量，以及该区间销售总额  

- histogram 桶要求两个参数：一个数值字段以及一个定义桶大小间隔

可以用 `histogram` 和一个嵌套的 `sum` 度量得到我们想要的答案

```json
{
   "size" : 0,
   "aggs":{
      "price":{ ①
         "histogram":{ ②
            "field": "price", ③
            "interval": 20000 ④
         },
         "aggs":{ 
            "revenue": { ⑤
               "sum": { ⑥
                 "field" : "price" ⑦
               }
             }
         }
      }
   }
}
```

- ① price聚合名称
- ② histogram 指定聚合函数
- ③ 根据field聚合直方图
- ④ interval 每个桶区间的间隔大小
- ⑤ revenue聚合名称
- ⑥ sum聚合函数
- ⑦ 根据price计算聚合

**注** 间隔设置为 20,000 意味着我们将会得到如 [0-19999, 20000-39999, ...] 这样的区间

返回结果 : 

```json
{
    "aggregations": {
        "price": { ①
            "buckets": [
                {
                    "key": 0, ②
                    "doc_count": 3, ③
                    "revenue": {
                        "value": 37000 ④
                    }
                },
                {
                    "key": 20000,
                    "doc_count": 4,
                    "revenue": {
                        "value": 95000
                    }
                },
                {
                    "key": 80000,
                    "doc_count": 1,
                    "revenue": {
                        "value": 80000
                    }
                }
            ]
        }
    }
}
```

- ① price聚合名称
- ② key 该桶的值0，注意的是0 代表[0-19999]区间
- ③ doc_count 数据条数（销售了3辆）
- ④ value 聚合结果

解释为 : 售价为0-19999区间的车，总共买了3辆，销售总额为37000

### extended_stats 

这个聚合函数聚合的值包含 `最大值` `最小值` `平均值` `标准差` 什么的，反正就是min max avg... 的加强版吧，使用方式一样

```json
{
  "size" : 0,
  "aggs": {
    "makes": {
      "terms": {
        "field": "make",
        "size": 10
      },
      "aggs": {
        "stats": {
          "extended_stats": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

## 根据时间统计date_histogram

> date_histogram 与 通常的 histogram 类似。 但不是在代表数值范围的数值字段上构建 buckets，而是在时间范围上构建 buckets。 因此每一个 bucket 都被定义成一个特定的日期大小 (比如， 1个月 或 2.5 天 )

```json
{
   "size" : 0,
   "aggs": {
      "sales": {
         "date_histogram": { ①
            "field": "sold", ②
            "interval": "month", ③
            "format": "yyyy-MM-dd" ④
         }
      }
   }
}
```

- ① date_histogram聚合函数
- ② interval 指定时间区间（year、month、week、day、hour...）
- ③ format显示时间格式化，由于时间字段el中是timestamp时间戳，因此在查询结果中可以使用该字段来显示格式化后的时间

返回结果 : 

```json
{
    "aggregations": {
        "sales": {
            "buckets": [
                {
                    "key_as_string": "2014-01-01", ①
                    "key": 1388534400000, ②
                    "doc_count": 1 ③
                },
                {
                    "key_as_string": "2014-02-01",
                    "key": 1391212800000,
                    "doc_count": 1
                },
                {
                    "key_as_string": "2014-05-01",
                    "key": 1398902400000,
                    "doc_count": 1
                }
            ]
        }
    }
}
```

- ① key_as_string 格式化后的时间
- ② key 格式化前的时间戳
- ③ doc_count 统计数据条数

## 空桶

仔细观察上例中你会发现没有3月的数据，这是因为el自动为我们过滤了`空`数据(doc_count 为0的数据)，要想返回空桶数据需要添加 `min_doc_count : 0`  
除了min_doc_count之外，它还常常结合`extended_bounds`一起使用

- min_doc_count : 指定返回文档最小条数的桶,强制返回所有 buckets，即使 buckets 可能为空
- extended_bounds : 指定日期桶的`额外`范围

> extended_bounds 参数需要一点解释。 min_doc_count 参数强制返回空 buckets，但是 Elasticsearch 默认只返回你的数据中最小值和最大值之间的 buckets。
因此如果你的数据只落在了 4 月和 7 月之间，那么你只能得到这些月份的 buckets（可能为空也可能不为空）。因此为了得到全年数据，我们需要告诉 Elasticsearch 我们想要全部 buckets， 即便那些 buckets 可能落在最小日期 之前 或 最大日期 之后 

```json
{
   "size" : 0,
   "aggs": {
      "sales": {
         "date_histogram": {
            "field": "sold",
            "interval": "month",
            "format": "yyyy-MM-dd",
            "min_doc_count" : 0, 
            "extended_bounds" : { 
                "min" : "2014-01-01",
                "max" : "2014-12-31"
            }
         }
      }
   }
}
```

**提示** : 在指定时间的时候常常与`now : 当前时间`（内置的时间函数吧？）组合使用，如 `now` `now-1d` `now+1w` `now-1M` ...

### 日期直方图与聚合函数嵌套

```json
{
   "size" : 0,
   "aggs": {
      "sales": {
         "date_histogram": {
            "field": "sold",
            "interval": "quarter", 
            "format": "yyyy-MM-dd",
            "min_doc_count" : 0,
            "extended_bounds" : {
                "min" : "2014-01-01",
                "max" : "2014-12-31"
            }
         },
         "aggs": {
            "per_make_sum": {
               "terms": {
                  "field": "make"
               },
               "aggs": {
                  "sum_price": {
                     "sum": { "field": "price" } 
                  }
               }
            },
            "total_sum": {
               "sum": { "field": "price" } 
            }
         }
      }
   }
}
```

解释 : 按季度统计汽车销售总额，并统计每个季度下个品牌汽车的销售总额

## 范围限定的聚合

> 聚合可以与搜索请求同时执行，但是我们需要理解一个新概念： 范围 。 默认情况下，聚合与查询是对同一范围进行操作的，也就是说，聚合是基于我们查询匹配的文档集合进行计算的，使用`query` 进行组合查询，如下请求示例与没有写query时等价

```json
{
    "size" : 0,
    "query" : {
        "match_all" : {}
    },
    "aggs" : {
        "colors" : {
            "terms" : {
              "field" : "color"
            }
        }
    }
}
```

## 全局桶

> 通常我们希望聚合是在查询范围内的，但有时我们也想要搜索它的子集，而聚合的对象却是 所有 数据。  
例如，比方说我们想知道福特汽车与 所有 汽车平均售价的比较。我们可以用普通的聚合（查询范围内的）得到第一个信息，然后用 全局 桶获得第二个信息。  
全局 桶包含 所有 的文档，它无视查询的范围。因为它还是一个桶，我们可以像平常一样将聚合嵌套在内：  

```json
{
    "size" : 0,
    "query" : {
        "match" : {
            "make" : "ford"
        }
    },
    "aggs" : {
        "single_avg_price": {
            "avg" : { "field" : "price" } ①
        },
        "all": {
            "global" : {}, ②
            "aggs" : {
                "avg_price": {
                    "avg" : { "field" : "price" } ③
                }

            }
        }
    }
}
```
- ① 聚合操作在查询范围内（例如：所有文档匹配 ford ）
- ② global 全局桶没有参数
- ③ 聚合操作针对所有文档，忽略汽车品牌

## 过滤

> 因为聚合是在查询结果范围内操作的，任何可以适用于查询的过滤器也可以应用在聚合上

```json
{
    "size" : 0,
    "query" : {
        "constant_score": {
            "filter": {
                "range": {
                    "price": {
                        "gte": 10000
                    }
                }
            }
        }
    },
    "aggs" : {
        "single_avg_price": {
            "avg" : { "field" : "price" }
        }
    }
}
```

> 使用 non-scoring 查询和使用 match 查询没有任何区别。查询（包括了一个过滤器）返回一组文档的子集，聚合正是操作这些文档。使用 filtering query 会忽略评分，并有可能会缓存结果数据等等

### 过滤桶

> 所谓过滤桶就是对过滤条件（桶）的过滤

```json
{
   "size" : 0,
   "query":{
      "match": {
         "make": "ford"
      }
   },
   "aggs":{
      "recent_sales": {
         "filter": { ①
            "range": { ②
               "sold": { ③
                  "from": "now-1M" ④
               }
            }
         },
         "aggs": {
            "average_price":{
               "avg": {
                  "field": "price" 
               }
            }
         }
      }
   }
}
```

- ① filter 使用过滤桶
- ② range 过滤类型(范围过滤)
- ③ 过滤字段(sold)
- ④ 过滤条件(`from : now-1M` 时间从上月开始)

### [后过滤器](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_post_filter.html)

> 只过滤搜索结果，不过滤聚合结果 : post_filter

```json
{
    "size" : 0,
    "query": {
        "match": {
            "make": "ford"
        }
    },
    "post_filter": {    
        "term" : {
            "color" : "green"
        }
    },
    "aggs" : {
        "all_colors": {
            "terms" : { "field" : "color" }
        }
    }
}
```

### 小结:

- 在 filter 过滤中的 non-scoring 查询，同时影响搜索结果和聚合结果
- filter 桶影响聚合
- post_filter 只影响搜索结果

## 排序(多桶排序)

> 多值桶（ terms 、 histogram 和 date_histogram ）动态生成很多桶,Elasticsearch 是如何决定这些桶展示给用户的顺序呢?  

- 默认:桶会根据 **doc_count** **降序排列**

### 内置排序

el中提供了3中内置的排序字段:  

#### _count
按文档数排序。对 terms 、 histogram 、 date_histogram 有效。
#### _term
按词项的字符串值的字母顺序排序。只在 terms 内使用。
#### _key
按每个桶的键值数值排序（理论上与 _term 类似）。 只在 histogram 和 date_histogram 内使用

```json
{
    "size" : 0,
    "aggs" : {
        "colors" : {
            "terms" : {
              "field" : "color",
              "order": {
                "_count" : "asc" 
              }
            }
        }
    }
}
```

### 按度量排序

> 根据统计的指标排序

```json
{
    "size" : 0,
    "aggs" : {
        "colors" : {
            "terms" : {
              "field" : "color",
              "order": { ①
                "avg_price" : "asc" ②
              }
            },
            "aggs": {
                "avg_price": { ③
                    "avg": {"field": "price"} 
                }
            }
        }
    }
}
```

- ① 指定排序
- ② 指定排序字段及顺序
- ③ 统计度量（指标）

#### [基于“深度”度量排序](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_sorting_based_on_deep_metrics.html)

## [去重统计](https://www.elastic.co/guide/cn/elasticsearch/guide/current/cardinality.html)

> Elasticsearch 提供的首个 **近似** 聚合是 cardinality （注：基数）度量。 它提供一个字段的基数，即该字段的 distinct 或者 unique 值的数目

```sql
SELECT COUNT(DISTINCT color)
FROM cars
```


```json
{
    "size" : 0,
    "aggs" : {
        "distinct_colors" : {
            "cardinality" : {
              "field" : "color"
            }
        }
    }
}
```

### [度量精度权衡](https://www.elastic.co/guide/cn/elasticsearch/guide/current/cardinality.html#_学会权衡)

**注** 这个度量是 **近似** 的，因此可以设置精确度

> 要配置精度，我们必须指定 precision_threshold 参数的值，precision_threshold 接受 0–40,000 之间的数字，更大的值还是会被当作 40,000 来处理

```json
{
    "size" : 0,
    "aggs" : {
        "distinct_colors" : {
            "cardinality" : {
              "field" : "color",
              "precision_threshold" : 100 
            }
        }
    }
}
```


## [百分位计算(去异常统计)](https://www.elastic.co/guide/cn/elasticsearch/guide/current/percentiles.html#percentiles)


**本文仅供个人学习，详细内容以[原文](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)为准**
