#### 搜索API 端点地址

```shell
GET /bank/_search?q=age:22
GET /bank,user/_doc/_search?q=age:22
GET /_all/_search?q=age:22
GET /_search?q=age:22
```

搜索的端点地址可以是多索引多mapping type的。搜索的参数可作为URI请求参数给出，也可用 request body 给出。

##### URI Search

URI 搜索方式通过URI参数来指定查询相关参数。让我们可以快速做一个查询。
https://www.elastic.co/guide/en/elasticsearch/reference/current/search-uri-request.html

```shell
GET /bank/_search?q=age:22
```

##### 特殊的查询参数用法

```shell
# 有多少文档匹配某个查询 size=0
GET /bank/_search?q=city:b*&size=0
# 查询结果
{
  "took": 14,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 99,
    "max_score": 0,
    "hits": []
  }
}
# --------------------------------------------
# 有没有文档匹配某个查询， terminate_after 限定每个分片取多少文档
GET /bank/_search?q=city:b*&size=0&terminate_after=1
# 查询结果
{
  "took": 48,
  "timed_out": false,
  "terminated_early": true,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 5,
    "max_score": 0,
    "hits": []
  }
}
```

#### Request  body Search

Request body 搜索方式以JSON格式在请求体中定义查询 query。请求方式可以是 GET 、POST 。

```shell
GET /back/_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

可用的参数：

**timeout**：请求超时时长，限定在指定时长内响应（即使没查完）；
**from**： 分页的起始行，默认0；
**size**：分页大小；
**request_cache**：是否缓存请求结果，默认true。
**terminate_after**：限定每个分片取几个文档。如果设置，则响应将有一个布尔型字段terminated_early来指示查询执行是否实际已经terminate_early。缺省为no terminate_after；
**search_type**：查询的执行方式，可选值
	dfs_query_then_fetch
	query_then_fetch ，默认： query_then_fetch ；
**batched_reduce_size**：一次在协调节点上应该减少的分片结果的数量。如果请求中的潜在分片数量可能很大，则应将此值用作保护机制以减少每个搜索请求的内存开销。

##### query 元素定义查询

query 元素用Query DSL 来定义查询。

```shell
GET /bank/_search
{
  "query": {
    "term": {
      "firstname": "virginia"
    }
  }
}
# -----
GET /bank/_search
{
  "query": {
    "term": {
      "firstname": {
          "value": "virginia"
      }
    }
  }
}
```

##### 指定返回哪些内容

source filter  对_source字段进行选择

```shell
GET /bank/_search
{
    "_source": false,
    "query" : {
        "firstname": "virginia"
    }
}
# ------------------
GET /_search
{
    "_source": "a*",
    "query" : {
        "term" : { "firstname" : "virginia" }
    }
}
# ------------------
GET /_search
{
    "_source": [ "gen*", "addr*" ],
    "query" : {
        "term" : { "firstname" : "virginia" }
    }
}
# ------------------
GET /_search
{
    "_source": {
        "includes": [ "a*", "b*" ],
        "excludes": [ "*er" ]
    },
    "query" : {
        "term" : { "firstname" : "virginia" }
    }
}
```

**来指定返回哪些stored字段** stored_fields

```shell
GET /bank/_search
{
    "stored_fields" : ["balance", "city"],
    "query" : {
        "term" : { "firstname" : "virginia" }
    }
}
```

**返回存储了docValue的字段值** docValue Field

```shell
GET /_search
{
    "query" : {
        "match_all": {}
    },
    "docvalue_fields" : ["test1", "test2"]
}
```

**version 指定返回文档的版本字段**

```shell
GET /_search
{
    "version": true,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

**explain 返回文档的评分解释**

```shell
GET /_search
{
    "explain": true,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

**Script Field 用脚本来对命中的每个文档的字段进行运算后返回**

```shell
GET /bank/_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "test1": {
      "script": {
        "lang": "painless",
        "source": "doc['balance'].value * 2"
      }
    },
    "test2": {
      "script": {
        "lang": "painless",
        "source": "doc['age'].value * params.factor",
        "params": {
          "factor": 2
        }
      }
    }
  }
}

# params  _source 取 _source字段值
# 官方推荐使用doc，理由是用doc效率比取_source 高。
GET /bank/_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "ffx": {
      "script": {
        "lang": "painless",
        "source": "doc['age'].value * doc['balance'].value"
      }
    },
    "balance*2": {
      "script": {
        "lang": "painless",
        "source": "params['_source'].balance*2"
      }
    }
  }
}

```

##### 过滤

**min_score**  限制最低评分得分。

```shell
GET /_search
{
    "min_score": 0.5,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

**post_filter**  后置过滤：在查询命中文档、完成聚合后，再对命中的文档进行过滤。

```shell
# 创建一个索引
PUT /shirts
{
    "mappings": {
        "_doc": {
            "properties": {
                "brand": { "type": "keyword"},
                "color": { "type": "keyword"},
                "model": { "type": "keyword"}
            }
        }
    }
}
# 添加一些数据
PUT /shirts/_doc/1?refresh
{
    "brand": "gucci",
    "color": "red",
    "model": "slim"
}
PUT /shirts/_doc/2?refresh
{
    "brand": "gucci",
    "color": "green",
    "model": "seec"
}
# 查询，后置过滤
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": { "brand": "gucci" } 
      }
    }
  },
  "aggs": {
    "colors": {
      "terms": { "field": "color" } 
    }
  },
  "post_filter": { 
    "term": { "color": "red" }
  }
}
```

##### sort  排序

可以指定按一个或多个字段排序。也可通过_score指定按评分值排序，_doc 按索引顺序排序。默认是按相关性评分从高到低排序

```shell
GET /bank/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }    },
    {
      "balance": {
        "order": "asc"
      }    },
    "_score"
  ]
}
```

**多值字段排序**

```shell
POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
      {"price" : {"order" : "asc", "mode" : "avg"}}
   ]
}
```

对于值是数组或多值的字段，也可进行排序，通过mode参数指定按多值的：
min 	最小值
max 	最大值
sum 	和
avg  	平均
median 	中值

**Missing values  缺失该字段的文档**

```shell
GET /_search
{
    "sort" : [
        { "price" : {"missing" : "_last"} }
    ],
    "query" : {
        "term" : { "product" : "chocolate" }
    }
}
```

missing 的值可以是 **_last, _first** 

**地理空间距离排序**

https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-sort.html#geo-sorting

_geo_distance   距离排序关键字
pin.location是 geo_point 类型的字段
distance_type：距离计算方式 arc球面 、plane 平面。
unit: 距离单位 km 、m   默认m

```shell
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : [-70, 40],
                "order" : "asc",
                "unit" : "km",
                "mode" : "min",
                "distance_type" : "arc"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

**Script Based Sorting   基于脚本计算的排序**

```shell
GET /_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    },
    "sort" : {
        "_script" : {
            "type" : "number",
            "script" : {
                "lang": "painless",
                "source": "doc['field_name'].value * params.factor",
                "params" : {
                    "factor" : 1.1
                }
            },
            "order" : "asc"
        }
    }
}
```

#### 折叠

用 collapse指定根据某个字段对命中结果进行折叠

```shell
GET /bank/_search
{
    "query": {
        "match_all": {}
    },
    "collapse" : {
        "field" : "age" 
    },
    "sort": ["balance"] 
}
```

在inner_hits 中返回多个角度的组内topN

```shell
GET /twitter/_search
{
    "query": {
        "match": {
            "message": "elasticsearch"
        }
    },
    "collapse" : {
        "field" : "user", 
        "inner_hits": [
            {
                "name": "most_liked",  
                "size": 3,
                "sort": ["likes"]
            },
            {
                "name": "most_recent", 
                "size": 3,
                "sort": [{ "date": "asc" }]
            }
        ]
    },
    "sort": ["likes"]
}
```

##### 分页

from and size

```shell
GET /_search
{
    "from" : 0, 
    "size" : 10,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

注意：搜索请求耗用的堆内存和时间与 from + size 大小成正比。分页越深耗用越大，为了不因分页导致OOM或严重影响性能，ES中规定from + size 不能大于索引setting参数 index.max_result_window 的值，默认值为 10,000。

Search after  在指定文档后取文档， 可用于**深度分页**

```shell
# 首次查询第一页
GET twitter/_search
{
    "size": 10,
    "query": {
        "match" : {
            "title" : "elasticsearch"
        }
    },
    "sort": [
        {"date": "asc"},
        {"_id": "desc"}
    ]
}
# 后续页的查询
GET twitter/_search
{
    "size": 10,
    "query": {
        "match" : {
            "title" : "elasticsearch"
        }
    },
    "search_after": [1463538857, "654323"],
    "sort": [
        {"date": "asc"},
        {"_id": "desc"}
    ]
}

```

用search_after，要求查询必须指定排序，并且这个排序组合值每个文档唯一（最好排序中包含_id字段）。 search_after的值用的就是这个排序值。
用search_after时 from 只能为0、-1。

##### 高亮 

https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-highlighting.html

```shell
GET /bank/_search
{
    "query" : {
        "match": { "address": "court" }
    },
    "highlight" : {
        "pre_tags" : ["<tag1>"],
        "post_tags" : ["</tag1>"],
        "fields" : {
            "address" : {}
        }
    }
}
```

##### Profile  为了调试、优化

对于执行缓慢的查询，想知道它为什么慢，时间都耗在哪了，可以在查询上加入上 profile 来获得详细的执行步骤、耗时信息。
https://www.elastic.co/guide/en/elasticsearch/reference/current/search-profile.html

```shell
GET /twitter/_search
{
  "profile": true,
  "query" : {
    "match" : { "message" : "some number" }
  }
}
```

##### count  api  查询数量

```shell
PUT /twitter/_doc/1?refresh
{
    "user": "kimchy"
}
# 方式一
GET /twitter/_doc/_count?q=user:kimchy
# 方式二
GET /twitter/_count
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
# 方式三
GET /twitter/_doc/_count
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

##### Explain api

获得某个查询的评分解释,及某个文档是否被这个查询命中
https://www.elastic.co/guide/en/elasticsearch/reference/current/search-explain.html

```shell
GET /twitter/_doc/0/_explain
{
      "query" : {
        "match" : { "message" : "elasticsearch" }
      }
}
```

##### Search Shards API

```shell
# 了解可执行查询的索引分片节点情况
GET /twitter/_search_shards
# 指定routing值的查询将在哪些分片节点上执行
GET /twitter/_search_shards?routing=foo,baz
```

##### validate api 

用来检查我们的查询是否正确，以及查看底层生成查询是怎样的。

```shell
GET twitter/_validate/query?q=user:foo
# 校验查询
GET twitter/_doc/_validate/query
{
  "query": {
    "query_string": {
      "query": "post_date:foo",
      "lenient": false
    }
  }
}
# 获得查询的解释
GET twitter/_doc/_validate/query?explain=true
{
  "query": {
    "query_string": {
      "query": "post_date:foo",
      "lenient": false
    }
  }
}
# 用rewrite获得比explain 更详细的解释
GET twitter/_doc/_validate/query?rewrite=true
{
  "query": {
    "more_like_this": {
      "like": {
        "_id": "2"
      },
      "boost_terms": 1
    }
  }
}
# 获得所有分片上的查询解释
GET twitter/_doc/_validate/query?rewrite=true&all_shards=true
{
  "query": {
    "match": {
      "user": {
        "query": "kimchy",
        "fuzziness": "auto"
      }
    }
  }
}
```

##### Search Template

```shell
# 注册一个模板
POST _scripts/<templatename>
{
    "script": {
        "lang": "mustache",
        "source": {
            "query": {
                "match": {
                    "title": "{{query_string}}"
                }
            }
        }
    }
}
# 使用模板进行查询
GET _search/template
{
    "id": "<templateName>", 
    "params": {
        "query_string": "search for these words"
    }
}
```













