## 聚合分析

聚合分析是数据库中重要的功能特性，完成对一个查询的数据集中数据的聚合计算，如：找出某字段（或计算表达式的结果）的最大值、最小值，计算和、平均值等。ES作为搜索引擎兼数据库，同样提供了强大的聚合分析能力。

**指标聚合**
对一个数据集求最大、最小、和、平均值等指标的聚合，在ES中称为指标聚合(metric)

**分桶**
关系型数据库中除了有聚合函数外，还可以对查询出的数据进行分组group by，再在组上进行指标聚合。
在 ES 中group by 称为分桶，桶聚合(bucketing)

ES中还提供了矩阵聚合（matrix）、管道聚合（pipleline）

**聚合查询语法**

```shell
"aggregations" : {
    "<aggregation_name>" : {
        "<aggregation_type>" : {
            <aggregation_body>
        }
        [,"meta" : {  [<meta_data_body>] } ]?
        [,"aggregations" : { [<sub_aggregation>]+ } ]?
    }
    [,"<aggregation_name_2>" : { ... } ]*
}
```

聚合计算的值可以取字段的值，也可是脚本计算的结果。

### 指标聚合

#### Avg Aggregation

平均值，如果要计算的字段值缺失，则会忽略这个文档，仅计算其他文档的平局值。
当然，可以同missing给缺失的字段指定默认值，missing中的值不会参与脚本计算。

```shell
GET /students/_search?size=0
{
  "aggs": {
    "missing_avg": {
      "avg": {
        "field": "score",
        "missing": 190
      }
    }
  }
}
#
GET /students/_search
{
  "aggs": {
    "avg_score": {
      "avg": {
        "field": "score", 
        "missing": 0, 
        "script": {
          "source": "_value"
        }
      }
    }
  },
  "size": 0
}
#
GET /students/_search
{
  "aggs": {
    "avg_score": {
      "avg": {
        "missing": 0, 
        "script": {
          "source": "doc.score.value"
        }
      }
    }
  },
  "size": 0
}
#
GET /students/_search
{
  "aggs": {
    "avg_score": {
      "avg": {
        "field": "score"
      }
    }
  },
  "size": 0
}
```

#### Max Aggregation

```shell
GET /students/_search?size=0
{
  "aggs": {
    "sum_score": {
      "sum": {
        "field": "score",
        "missing": 100,
        "script": {
          "source": "doc['score'].value * _value"
        }
      }
    }
  }
} 
# 
GET /students/_search?size=0
{
  "aggs": {
    "sum_score": {
      "sum": {
        "field": "score",
        "missing": 100
      }
    }
  }
}
# 
GET /students/_search?size=0
{
  "aggs": {
    "sum_score": {
      "sum": {
        "field": "score"
      }
    }
  }
}
```

Min Aggregation

```shell
GET /students/_search?size=0
{
  "aggs": {
    "min_score": {
      "min": {
        "field": "score",
        "missing": 33,
        "script": {
          "source": "doc['score'].value * (1-params.rate)",
          "params": {
            "rate": 0.6
          }
        }
      }
    }
  }
} 

GET /students/_search?size=0
{
  "aggs": {
    "min_score": {
      "min": {
        "field": "score",
        "missing": 100
      }
    }
  }
}

GET /students/_search?size=0
{
  "aggs": {
    "min_score": {
      "min": {
        "field": "score"
      }
    }
  }
}
```

#### Sum Aggregation

```shell
GET /students/_search?size=0
{
  "aggs": {
    "sum_score": {
      "sum": {
        "field": "score",
        "missing": 100,
        "script": {
          "source": "doc['score'].value * (1-params.rate)",
          "params": {
            "rate": 0.6
          }
        }
      }
    }
  }
} 

GET /students/_search?size=0
{
  "aggs": {
    "sum_score": {
      "sum": {
        "field": "score",
        "missing": 100
      }
    }
  }
}

GET /students/_search?size=0
{
  "aggs": {
    "sum_score": {
      "sum": {
        "field": "score"
      }
    }
  }
}
```

#### Value Count Aggregation

```shell
GET /students/_search?size=0
{
  "aggs": {
    "score_count": {
      "value_count": {
        "field": "score",
        "script": {
          "source": "doc.score.value"
        }
      }
    }
  }
} 

GET /students/_search?size=0
{
  "aggs": {
    "score_count": {
      "value_count": {
        "field": "score",
        "missing": 100
      }
    }
  }
}

GET /students/_search?size=0
{
  "aggs": {
    "score_count": {
      "value_count": {
        "field": "score"
      }
    }
  }
}
```

#### Stats Aggregation

```shell
GET /students/_search?size=0
{
  "aggs": {
    "score_stats": {
      "stats": {
        "missing": 120, 
        "field": "score",
        "script": {
          "source": "_value * 1"
        }
      }
    }
  }
}

GET /students/_search?size=0
{
  "aggs": {
    "score_stats": {
      "stats": {
        "missing": 120, 
        "field": "score"
      }
    }
  }
}

GET /students/_search?size=0
{
  "aggs": {
    "score_stats": {
      "stats": {
        "field": "score"
      }
    }
  }
}
```

#### Extended Stats Aggregation

```shell
GET /students/_search?size=0
{
  "aggs": {
    "score_extend_stats": {
      "extended_stats": {
        "missing": 120, 
        "field": "score",
        "script": {
          "source": "_value * 1"
        }
      }
    }
  }
}

GET /students/_search?size=0
{
  "aggs": {
    "score_extend_stats": {
      "extended_stats": {
        "missing": 120, 
        "field": "score"
      }
    }
  }
}

GET /students/_search?size=0
{
  "aggs": {
    "score_extend_stats": {
      "extended_stats": {
        "field": "score"
      }
    }
  }
}
```

#### Cardinality Aggregation

```shell
GET /students/_search?size=0
{
 "aggs": {
   "cardinality_count": {
     "cardinality": {
       "field": "num",
       "script": {
         "source": "_value * 10"
       }
     }
   }
 } 
} 

GET /students/_search?size=0
{
 "aggs": {
   "cardinality_count": {
     "cardinality": {
       "field": "num",
       "precision_threshold": 20
     }
   }
 } 
}

GET /students/_search?size=0
{
 "aggs": {
   "cardinality_count": {
     "cardinality": {
       "field": "num"
     }
   }
 } 
}
```

#### Percentiles Aggregation

占比百分位对应的值统计。对指定字段（脚本）的值按从小到大累计每个值对应的文档数的占比（占所有命中文档数的百分比），返回指定占比比例对应的值。默认返回[ 1, 5, 25, 50, 75, 95, 99 ]分位上的值。如下中间的结果，可以理解为：占比为50%的文档的age值 <= 31，或反过来：age<=31的文档数占总命中文档数的50%

```shell
GET /students/_search?size=0
{
  "aggs": {
    "score_percentiles": {
      "percentiles": {
        "field": "num",
        "percents": [
          1,
          5,
          25,
          50,
          75,
          95,
          99
        ]
      }
    }
  }
}

GET /students/_search?size=0
{
  "aggs": {
    "score_percentiles": {
      "percentiles": {
        "field": "num"
      }
    }
  }
}
```

#### Percentile Ranks Aggregation

统计值小于等于指定值的文档占比

```shell
GET /students/_search?size=0
{
  "aggs": {
    "score_percentile_ranks": {
      "percentile_ranks": {
        "field": "score",
        "missing": 100, 
        "values": [
          100,
          300
        ]
      }
    }
  }
}
```

#### Top Hits Aggregation

```shell
GET /students/_search?size=0
{
  "aggs": {
    "top_tags": {
      "terms": {
        "field": "num"
      },
      "aggs": {
        "top_score_hits": {
          "top_hits": {
            "sort": ["num"],
            "size": 10
          }
        }
      }
    }
  }
}
```

Scripted Metric Aggregation

Weighted Avg Aggregation
Geo Bounds Aggregation
Geo Centroid Aggregation

#### max、min、avg、sum

计算索引bank中age字段的最大值、最小值、平均值、总和

```shell
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
  	"max_age": {
      "max": {
        "field": "age"
      }
    },
    "min_age": {
      "min": {
        "field": "age"
      }
    },
    "agv_age": {
      "avg": {
        "field": "age"
      }
    },
    "sum_age": {
      "sum": {
        "field": "age"
      }
    }
  },
  "size": 0
}
```

通过脚本计算的值聚合，可以通过doc.fieldName.value或者doc['fieldName'].value取字段的值，也可以通过_value取字段的值

```shell
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "agv_age": {
      "avg": {
        "field": "age",
        "script": {
          "source": "doc.age.value"
        }
      }
    },
    "agv_age_add_10": {
      "avg": {
        "field": "age",
        "script": {
          "source": "doc.age.value + 10"
        }
      }
    },
    "sum_age": {
      "sum": {
        "field": "age",
        "script": {
          "source": "_value"
        }
      }
    },
    "sum_age_minus_1": {
      "sum": {
        "field": "age",
        "script": {
          "source": "_value - 1"
        }
      }
    },
    "max_age": {
      "max": {
        "field": "age"
      }
    },
    "min_age": {
      "min": {
        "field": "age"
      }
    }
  },
  "size": 0
}
```

为缺失值字段，指定默认值。如未指定，缺失该字段值的文档将被忽略。

```shell
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "sum_age": {
      "sum": {
        "field": "age1",
        "missing": 1, 
        "script": {
          "source": "doc.age.value"
        }
      }
    }
  },
  "size": 0
}
```

#### 计数

##### 文档计数 count

```shell
POST /bank/_doc/_count
{
  "query": {
    "match_all": {}
  }
}
```

##### 有值字段计数 value_count

```shell
POST bank/_search
{
  "aggs": {
    "age_value_count": {
      "value_count": {
        "field": "age"
      }
    }
  },
  "size": 0
}
```

##### 值去重计数cardinality 

```shell
POST bank/_search
{
  "aggs": {
    "age_count": {
      "cardinality": {
        "field": "age"
      }
    }
  },
  "size": 0
}
```

##### stats 统计 count max min avg sum 5个值

```shell
POST bank/_search
{
  "aggs": {
    "age_stats": {
      "stats": {
        "field": "age"
      }
    }
  },
  "size": 0
}
```

##### Extended stats

高级统计，比stats多4个统计结果：
平方和、方差、标准差、平均值加/减两个标准差的区间

```shell
POST bank/_search
{
  "aggs": {
    "age_extended_stats": {
      "extended_stats": {
        "field": "age"
      }
    }
  },
  "size": 0
}
```

##### Percentiles

对指定字段（脚本）的值按从小到大累计每个值对应的文档数的占比（占所有命中文档数的百分比），返回指定占比比例对应的值。默认返回[ 1, 5, 25, 50, 75, 95, 99 ]分位上的值。如下中间的结果，可以理解为：占比为50%的文档的age值 <= 31，或反过来：age<=31的文档数占总命中文档数的50%

```shell
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_percent": {
      "percentiles": {
        "field": "age",
        "missing": 1, 
        "script": {
          "source": "doc.age.value"
        }
      }
    }
  },
  "size": 0
}
```

指定分位值

```shell
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_percent": {
      "percentiles": {
        "field": "age",
        "missing": 1, 
        "percents": [
          95,
          99
        ], 
        "script": {
          "source": "doc.age.value"
        }
      }
    }
  },
  "size": 0
}
```

##### Percentiles rank

统计值小于等于指定值的文档占比

```shell
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_percent_ranks": {
      "percentile_ranks": {
        "field": "age",
        "values": [20, 30], 
        "script": {
          "source": "doc.age.value"
        }
      }
    }
  },
  "size": 0
}
```

### 桶聚合

#### Terms Aggregation 

根据字段值项**分组聚合**

```shell
GET /bank/_search?size=0
{
  "aggs": {
    "age_terms": {
      "terms": {
        "field": "age",
        "size": 10,
        "min_doc_count": 10,
        "shard_size": 100,
        "show_term_doc_count_error": true,
        "order": {
          "age_stats.max": "asc"
        }
      },
      "aggs": {
        "age_stats": {
          "stats": {
            "field": "age"
          }
        }
      }
    }
  }
}

GET /bank/_search?size=0
{
  "aggs": {
    "age_terms": {
      "terms": {
        "field": "age",
        "size": 10,
        "min_doc_count": 10,
        "shard_size": 10,
        "show_term_doc_count_error": true,
        "order": {
          "_key": "asc"
        }
      }
    }
  }
}

GET /bank/_search?size=0
{
  "aggs": {
    "age_terms": {
      "terms": {
        "field": "age",
        "size": 10,
        "min_doc_count": 10,
        "shard_size": 10,
        "show_term_doc_count_error": true,
        "order": {
          "_count": "asc"
        }
      }
    }
  }
}

GET /bank/_search?size=0
{
  "aggs": {
    "age_terms": {
      "terms": {
        "field": "age"
      }
    }
  }
}
```

#### Range Aggregation

```shell
GET /bank/_search?size=0
{
  "aggs": {
    "age_range": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "to": 20
          },
          {
            "from": 20,
            "to": 25
          },
          {
            "from": 25
          }
        ]
      }
    }
  }
}

GET /bank/_search?size=0
{
  "aggs": {
    "age_range": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "to": 20,
            "key": "<20"
          },
          {
            "from": 20,
            "to": 25,
            "key": ">20&<25"
          },
          {
            "from": 25,
            "key": ">25"
          }
        ]
      }
    }
  }
}

GET /bank/_search?size=0
{
  "aggs": {
    "age_range": {
      "range": {
        "field": "age",
        "keyed": true, 
        "ranges": [
          {
            "to": 20
          },
          {
            "from": 20,
            "to": 25
          },
          {
            "from": 25
          }
        ]
      }
    }
  }
}

GET /bank/_search?size=0
{
  "aggs": {
    "age_range": {
      "range": {
        "field": "age",
        "keyed": true, 
        "ranges": [
          {
            "to": 20,
            "key": "<20"
          },
          {
            "from": 20,
            "to": 25,
            "key": ">20&<25"
          },
          {
            "from": 25,
            "key": ">25"
          }
        ]
      }
    }
  }
}
```

#### Missing Aggregation

```shell
GET /bank/_search?size=0
{
  "aggs": {
    "without_age": {
      "missing": {
        "field": "age"
      }
    }
  }
}
```

#### Histogram Aggregation

```shell
GET /bank/_search?size=0
{
  "aggs": {
    "histogram_balance": {
      "histogram": {
        "field": "balance",
        "interval": 10000
      }
    }
  }
}

GET /bank/_search?size=0
{
  "aggs": {
    "histogram_balance": {
      "histogram": {
        "field": "balance",
        "interval": 10000,
        "keyed": true
      }
    }
  }
}
```

#### Date Range Aggregation

```shell
POST /sales/_search?size=0
{
    "aggs": {
        "range": {
            "date_range": {
                "field": "date",
                "format": "MM-yyy",
                "ranges": [
                    { "to": "now-10M/M" }, 
                    { "from": "now-10M/M" } 
                ]
            }
        }
    }
}
#
POST /sales/_search?size=0
{
   "aggs": {
       "range": {
           "date_range": {
               "field": "date",
               "missing": "1976/11/30",
               "ranges": [
                  {
                    "key": "Older",
                    "to": "2016/02/01"
                  }, 
                  {
                    "key": "Newer",
                    "from": "2016/02/01",
                    "to" : "now/d"
                  }
              ]
          }
      }
   }
}
```

#### Date Histogram Aggregation

```shell
POST /sales/_search?size=0
{
    "aggs" : {
        "sales_over_time" : {
            "date_histogram" : {
                "field" : "date",
                "interval" : "month"
            }
        }
    }
}
#
POST /sales/_search?size=0
{
    "aggs" : {
        "sales_over_time" : {
            "date_histogram" : {
                "field" : "date",
                "interval" : "90m"
            }
        }
    }
}
#
POST /sales/_search?size=0
{
    "aggs" : {
        "sales_over_time" : {
            "date_histogram" : {
                "field" : "date",
                "interval" : "1M",
                "format" : "yyyy-MM-dd" 
            }
        }
    }
}
#
PUT my_index/_doc/1?refresh
{
  "date": "2015-10-01T00:30:00Z"
}

PUT my_index/_doc/2?refresh
{
  "date": "2015-10-01T01:30:00Z"
}

GET my_index/_search?size=0
{
  "aggs": {
    "by_day": {
      "date_histogram": {
        "field":     "date",
        "interval":  "day"
      }
    }
  }
}
#
GET my_index/_search?size=0
{
  "aggs": {
    "by_day": {
      "date_histogram": {
        "field":     "date",
        "interval":  "day",
        "time_zone": "-01:00"
      }
    }
  }
}
#
GET my_index/_search?size=0
{
  "aggs": {
    "by_day": {
      "date_histogram": {
        "field":     "date",
        "interval":  "day",
        "offset":    "+6h"
      }
    }
  }
}
# 
POST /sales/_search?size=0
{
    "aggs" : {
        "sales_over_time" : {
            "date_histogram" : {
                "field" : "date",
                "interval" : "1M",
                "format" : "yyyy-MM-dd",
                "keyed": true
            }
        }
    }
}
```

#### Filter Aggregation

```shell
POST /sales/_search?size=0
{
    "aggs" : {
        "t_shirts" : {
            "filter" : { "term": { "type": "t-shirt" } },
            "aggs" : {
                "avg_price" : { "avg" : { "field" : "price" } }
            }
        }
    }
}
```

#### Filters Aggregation

```shell
PUT /logs/_doc/_bulk?refresh
{ "index" : { "_id" : 1 } }
{ "body" : "warning: page could not be rendered" }
{ "index" : { "_id" : 2 } }
{ "body" : "authentication error" }
{ "index" : { "_id" : 3 } }
{ "body" : "warning: connection timed out" }
{ "index" : { "_id" : 4 } }
{ "body": "info: user Bob logged out" }

#
GET logs/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "filters" : {
          "errors" :   { "match" : { "body" : "error"   }},
          "warnings" : { "match" : { "body" : "warning" }}
        }
      }
    }
  }
}
#
GET logs/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "filters" : [
          { "match" : { "body" : "error"   }},
          { "match" : { "body" : "warning" }}
        ]
      }
    }
  }
}
#
GET logs/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "other_bucket_key": "other_messages",
        "filters" : {
          "errors" :   { "match" : { "body" : "error"   }},
          "warnings" : { "match" : { "body" : "warning" }}
        }
      }
    }
  }
}
```



Adjacency Matrix Aggregation
Auto-interval Date Histogram Aggregation
Intervals
Children Aggregation
Composite Aggregation

Diversified Sampler Aggregation
Geo Distance Aggregation
GeoHash grid Aggregation
Global Aggregation
IP Range Aggregation
Nested Aggregation
Reverse nested Aggregation
Sampler Aggregation
Significant Terms Aggregation
Significant Text Aggregation



根据脚本计算值分组

```shell
GET /bank/_search
{
    
    "aggs" : {
        "age_terms" : {
            "terms" : {
                "script" : {
                    "source": "doc.age.value",
                    "lang": "painless"
                }
            }
        }
    },
    "size": 0
}
```

缺失值处理

```shell
GET /bank/_search
{
    "aggs" : {
        "tags" : {
             "terms" : {
                 "field" : "tags",
                 "missing": "N/A" 
             }
         }
    },
    "size": 0
}
```

shard_size 指定每个分片上返回多少个分组
shard_size 的默认值为：
 索引只有一个分片：= size
多分片：=  size * 1.5 + 10
每个分组上显示偏差值

```shell
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_term": {
      "terms": {
        "field": "age",
        "shard_size": 10
      }
    }
  },
  "size": 0
}
```

order  指定分组的排序

```shell
# 根据_count排序
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_term": {
      "terms": {
        "field": "age",
        "shard_size": 10,
        "order": {
          "_count": "asc"
        }
      }
    }
  },
  "size": 0
}
# 根据_key排序
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_term": {
      "terms": {
        "field": "age",
        "shard_size": 10,
        "order": {
          "_key": "asc"
        }
      }
    }
  },
  "size": 0
}
```

取分组指标值

```shell
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_term": {
      "terms": {
        "field": "age",
        "shard_size": 10,
        "order": {
          "_key": "asc"
        }
      },
      "aggs": {
        "max_balance": {
          "max": {
            "field": "balance"
          }
        },
        "min_balance": {
          "min": {
            "field": "balance"
          }
        }
      }
    }
  },
  "size": 0
}
```

```shell
# 分词的字段是不能聚合的，如果保留了keyword，则可以根据keyword聚合
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_term": {
      "terms": {
        "field": "address.keyword",
        "shard_size": 10,
        "order": {
          "_key": "asc"
        }
      },
      "aggs": {
        "max_balance": {
          "max": {
            "field": "balance"
          }
        },
        "min_balance": {
          "min": {
            "field": "balance"
          }
        }
      }
    }
  },
  "size": 0
}
```

根据分组指标值排序

```shell
# max_balance是一个分组指标，先计算这个指标，然后再按这个指标排序
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_term": {
      "terms": {
        "field": "age",
        "shard_size": 10,
        "order": {
          "max_balance": "asc"
        }
      },
      "aggs": {
        "max_balance": {
          "max": {
            "field": "balance"
          }
        },
        "min_balance": {
          "min": {
            "field": "balance"
          }
        }
      }
    }
  },
  "size": 0
}
```

#### 筛选分组

用文档计数来筛选

```shell
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_term": {
      "terms": {
        "field": "age",
        "min_doc_count": 55
      }
    }
  },
  "size": 0
}
```

筛选指定的值列表

```shell
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_term": {
      "terms": {
        "field": "age",
        "include": [20, 40]
      }
    }
  },
  "size": 0
}
```

正则表达式筛选

```shell
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_term": {
      "terms": {
        "field": "city.keyword",
        "include": ".*e*"
      }
    }
  },
  "size": 0
}
```

指定值列表

```shell
POST /bank/_search
{
  "query": {
    "match_all": {
    }
  }, 
  "aggs": {
    "age_term": {
      "terms": {
        "field": "city.keyword",
        "include": ["Belvoir","Abiquiu"]
      }
    }
  },
  "size": 0
}
```

**filter Aggregation**  对满足过滤查询的文档进行聚合计算
在查询命中的文档中选取复合过滤条件的文档进行聚合

```shell
GET /bank/_search
{
  "aggs": {
    "age_terms": {
      "filter": {
        "match": {
          "state": "DE"
        }
      },
      "aggs": {
        "age_avg": {
          "avg": {
            "field": "age"
          }
        }
      }
    }
  },
  "size": 0
}
```

**Filters Aggregation**  多个过滤组聚合计算

```shell
GET /bank/_search
{
  "aggs": {
    "age_aggs": {
      "filters": {
        "filters": {
          "state_de": {
            "match": {"state": "DE"}
          },
          "state_pa": {
            "match": {"state": "PA"}
          }
        }
      }
    }
  },
  "size": 0
}
```

为其他值组指定key

```shell
GET /bank/_search
{
  "aggs": {
    "age_aggs": {
      "filters": {
        "other_bucket_key": "other_count", 
        "filters": {
          "state_de": {
            "match": {"state": "DE"}
          },
          "state_pa": {
            "match": {"state": "PA"}
          }
        }
      }
    }
  },
  "size": 0
}
```

**Range Aggregation**  范围分组聚合

```shell
GET /bank/_search
{
  "aggs": {
    "age_range": {
      "range": {
        "field": "age",
        "ranges": [
          {"to": 20},
          {"from": 20, "to": 25},
          {"from": 25}
        ]
      }
    }
  },
  "size": 0
}
```

为组指定key

```shell
GET /bank/_search
{
  "aggs": {
    "age_range": {
      "range": {
        "field": "age",
        
        "ranges": [
          {"to": 20, "key": "<20"},
          {"from": 20, "to": 25, "key": ">20&<25"},
          {"from": 25, "key": ">25"}
        ]
      }
    }
  },
  "size": 0
}
#
GET /bank/_search
{
  "aggs": {
    "age_range": {
      "range": {
        "field": "age",
        "keyed": true, 
        "ranges": [
          {"to": 20, "key": "<20"},
          {"from": 20, "to": 25, "key": ">20&<25"},
          {"from": 25, "key": ">25"}
        ]
      }
    }
  },
  "size": 0
}
```

Date Range Aggregation  时间范围分组聚合

```shell
GET my_index/_search
{
  "aggs": {
    "date_range": {
      "date_range": {
        "field": "date",
        "ranges": [
          {
            "from": "now-2000d/d",
            "to": "now/d"
          }
        ]
      }
    }
  }
}
```

Date Histogram Aggregation  时间直方图（柱状）聚合

就是按天、月、年等进行聚合统计。可按 year (1y), quarter (1q), month (1M), week (1w), day (1d), hour (1h), minute (1m), second (1s) 间隔聚合或指定的时间间隔聚合。

```shell
GET my_index/_search
{
  "aggs": {
    "date_range": {
      "date_histogram": {
        "field": "date",
        "interval": "hour"
      }
    }
  },
  "size": 0
}
#
GET my_index/_search
{
  "aggs": {
    "date_range": {
      "date_histogram": {
        "field": "date",
        "interval": "2h"
      }
    }
  },
  "size": 0
}
```

Missing Aggregation  缺失值的桶聚合

```shell
POST /bank/_search?size=0
{
    "aggs" : {
        "account_without_a_age" : {
            "missing" : { "field" : "age" }
        }
    }
}
```









































