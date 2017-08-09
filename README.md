# Elasticsearch

- [基础入门](#基础入门)
- [搜索](#搜索)
- [查询](./query.md)
- [聚合](./aggs.md)

## 基础入门

### 健康状态

ES 健康状态`status` 分为三个状态：`green` 、 `yellow` 或者 `red`  

- green : 所有的主分片和副本分片都正常运行
- yellow : 所有的主分片都正常运行，但不是所有的副本分片都正常运行
- red : 有主分片没能正常运行

```
GET /_cluster/health
```

```json
{
   "cluster_name":          "elasticsearch",
   "status":                "green", 
   "timed_out":             false,
   "number_of_nodes":       1,
   "number_of_data_nodes":  1,
   "active_primary_shards": 0,
   "active_shards":         0,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```


### 索引

- [elastic mapping - field datatypes](https://www.elastic.co/guide/en/elasticsearch/reference/5.x/mapping.html#_field_datatypes)  

`text` `keyword` `date` `long` `double` `boolean` `ip`

```
PUT /YOUR_INDEX_NAME

{
   "mappings": {
      "YOUR_TYPE_NAME": {
         "properties": {
            "YOUR_FIELD_NAME": {
               "type": "YPUR_FIELD_TYPE"
            }
         }
      }
   }
}
```

**ES自动关系映射错误**   
- [需要在ES先创建索引及映射关系，然后导入数据](https://discuss.elastic.co/t/number-format-exception-for-string-type/59175/6)  
- [新建一个索引，然后将旧的索引指向新的索引](https://www.elastic.co/guide/en/elasticsearch/reference/5.x/mapping.html#_updating_existing_mappings)

### 文档

```
Relational DB -> Databases（数据库）-> Tables（表） -> Rows（记录）      -> Columns（字段）
Elasticsearch -> Indices（索引）    -> Types（类型）-> Documents（文档） -> Fields（字段）
```

在ES中没一条数据称之为一条 **文档**  

- 文档元数据

> 一个文档不仅仅包含它的数据 ，也包含 **元数据** —— *有关文档的信息*，其中包含三个必须的元素： 

- _index ： 索引，所属数据库
- _type ： 类型，所属表
- _id ： 唯一ID，表中唯一标示

#### GET 查询文档

```
GET /index_name/type_name/id
```

返回数据在`_source`字段中，如:

```
GET /website/blog/123?pretty
```

```json
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out...",
      "date":  "2014/01/01"
  }
}
```

**可选参数**

- pretty : 格式化JSON
- _source : 指定返回字段

如：指定返回`title` `text`字段，`date`将被过滤  

```
GET /website/blog/123?_source=title,text
```

```json
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :   true,
  "_source" : {
      "title": "My first blog entry" ,
      "text":  "Just trying this out..."
  }
}
```

*found* 字段标示是否找到匹配数据，bool值

**可选端点**

*如果你只想得到 _source 字段，不需要任何元数据，你能使用 `_source` 端点*，如：  

```
GET /website/blog/123/_source
```

```json
{
   "title": "My first blog entry",
   "text":  "Just trying this out...",
   "date":  "2014/01/01"
}
```

### 更新文档

**文档不能被修改，只能被替换** 简单理解的话就是对象的覆盖合并，新的参数覆盖老的参数

```
PUT /index_name/type_name/id

{
  ...some data...
}
```

如：

```
PUT /website/blog/123

{
  "title": "My first blog entry",
  "text":  "I am starting to get the hang of this...",
  "date":  "2014/01/02"
}
```

```json
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 2,
  "created":   false 
}
```

- _version : 每次数据变更的版本号，ES自动维护
- created : 相同唯一标示的数据是否存在，bool值，false标示已存在，则是更新，否则创建

#### 部分更新

由于文档只能替换，因此使用POST "重新创建(替换)"，替换的参数需通过`doc`属性传递

```
POST /index_name/type/id/_update
```

例如，我们增加字段 tags 和 views 到博客  

```
POST /website/blog/1/_update
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
```

### 创建文档

创建文档有三种方式：

- 自动生成ID

```
POST /index_name/type_name
{...}
```

如：

```
POST /website/blog
{...}
```

- 指定ID

  - 指定操作选项 `op_type`

  ```
  POST /index_name/type_name/id?op_type=create
  {...}
  ```

  - 指定端点 `_create`

  ```
  POST /index_name/type_name/id/_create
  {...}
  ```

创建成功返回 `201` ，失败返回 `409` 的状态码   


### 删除文档

```
DELETE /index_name/type_name/id
```

如： 

```
DELETE /website/blog/123
```

```
{
  "found" :    true,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 3
}
```

> found 是否匹配到这条数据，bool，true表示找到并删除；false表示没有找到这条数据，当然也就删除失败了，值得一提的是version不管删除是否成功都会增加  

## 搜索

搜索请求方式有两种：  
- 第一种是通过GET请求拼接请求参数的形式发送要查询的条件  
- 第二种 `请求体查询` 是通过请求体（body）的形式传递参数

前者由于参数需要拼接，并且请求URL需要编码，而不易阅读和操作性相对较低；而后者可以传递JSON格式的请求体，从而被推荐使用

关于[映射](https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping-intro.html)，自行查看

- 空查询

空查询将返回所有索引库（indices)中的所有文档，请求体中是一个空的JSON对象：

```
GET /_search
{}
```

- [多索引，多类型](https://www.elastic.co/guide/cn/elasticsearch/guide/current/multi-index-multi-type.html)

多个索引，类型使用`逗号` 隔开，并且使用`_search` 作为端点组合查询请求，而这中格式其实是 **规则匹配**，详情查看原文  

```
/index_name.../type_name.../_search
```


- 分页

分页使用 `size` 和 `from` 组合查询，类似mysql中的 `limit` 和 `offset`，其中size + from 不能大于10000

  - size : 显示应该返回的结果数量，默认是 10
  - from : 显示应该跳过的初始结果数量，默认是 0

- 查询表达式

查询表达式，是`查询语句（条件）`传递给 **query** 参数：

```
GET /_search
{
    "query": YOUR_QUERY_HERE
}
```

**查询语句的结构**

```
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}
```

如果是针对某个字段，那么它的结构如下：

```
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}
```

举个例子，你可以使用 match 查询语句 来查询 tweet 字段中包含 elasticsearch 的 tweet：

```
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}
```

**合并查询语句**

所谓合并查询语句就是组合嵌套多个查询条件，因此可以写出复杂的查询语句

## 参考文档

- [Elasticsearch: 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)
