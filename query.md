# 查询

## 常用的查询

- `match_all`

match_all 匹配`所有文档`，在没有指定查询方式时，它是`默认`的查询

- [match](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_most_important_queries.html#_match_查询)

match 查询可理解为匹配，在全文查询中，匹配前它将 先分析匹配字符，然后执行匹配查询；而在精确查询中（如：数字、日期、布尔或者一个 not_analyzed 字符串字段），它将精确匹配给定的值

- multi_match

multi_match和match类似，只是提供在`多个字段`上执行`相同匹配`的搜索

- range

看单词就可以才出来啦，区间查询嘛，指定查询范围，(类似`between and`) ;可以使用的操作符如下：

`gt` `gte` `lt` `lte`

- term

同样，精确查询，可匹配 数字、时间、布尔或 not_analyzed 的字符串

- terms

和term查询一样，只是可以提供可多个可选值，任意一个匹配即可，类似`ANY`

- [exists & missing](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_most_important_queries.html#_exists_查询和_missing_查询)

exists 和 missing 被用于查找那些指定字段中有值 (exists) 或无值 (missing) 的文档。这与SQL中的 IS_NULL (missing) 和 NOT IS_NULL (exists) 类似，如下判断title字段存在值:

```
{
    "exists":   {
        "field":    "title"
    }
}
```

- [bool](https://www.elastic.co/guide/cn/elasticsearch/guide/current/combining-queries-together.html)

布尔组合过滤器查询，其意思是根据查询条件返回true 、false值，并筛选数据的一类条件，它可以将多个查询组合在一起，详细示例[查看](https://www.elastic.co/guide/cn/elasticsearch/guide/current/combining-filters.html#bool-filter)原文，可选参数如下：

 - must : `必须` 满足条件才可以匹配，`AND`
 - must_not : `必须不` 满足条件才可以匹配, `NOT`
 - should : `应该` 满足条件，当然也可以不满足啦，如果满足这些语句中的任意语句，将增加 _score ，否则，无任何影响。它们主要用于修正每个文档的相关性得分 , `OR`
 - filter : `必须` 匹配，提供数据过滤的筛选条件，以不评分、过滤模式来进行
- minimum_should_match : should 评分的精确度参数，可以使用数字或百分比，详细[查看](https://www.elastic.co/guide/cn/elasticsearch/guide/current/bool-query.html#_控制精度)原文

- [constant_score](https://www.elastic.co/guide/cn/elasticsearch/guide/current/combining-queries-together.html#constant_score-query)

常量+评分查询，类似bool查询，但它只存在filter过滤条件，因此也就不需要bool查询这样复杂的结构，他们在性能上是相同的，如：

```
{
    "constant_score":   {
        "filter": {
            "term": { "category": "ebooks" } 
        }
    }
}
```
*精确查询是不评分的，因此上例中term的使用将查询变成了不评分的*

### [近似匹配](https://www.elastic.co/guide/cn/elasticsearch/guide/current/proximity-matching.html) & [部分匹配](https://www.elastic.co/guide/cn/elasticsearch/guide/current/partial-matching.html)

- match_phrase

短语匹配

- prefix

前缀查询

- 通配符与正则表达式查询

`?` 匹配任意字符， `*` 匹配 0 或多个字符



## [排序](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_Sorting.html)

排序使用关键参数`sort`实现，sort 可以接收字符串，对象，数组为值作为排序条件


## [聚合函数](https://www.elastic.co/guide/cn/elasticsearch/guide/current/aggregations.html)

使用聚合函数通过`aggs`参数指定，可选函数如下：

`avg` `max` `min` `sum` `extended_stats`


