## 索引详解

## 索引管理

### 集群_cat

```
http://10.10.10.11:9200/_cat
```

```
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/tasks
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/thread_pool/{thread_pools}
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/nodeattrs
/_cat/repositories
/_cat/snapshots/{repository}
/_cat/templates
/_cat/health?v
```

### 集群的健康状况

```
http://10.10.10.11:9200/_cat/health?v
```

状态值说明

**Green** - everything is good (cluster is fully functional)
**Yellow** - all data is available but some replicas are not yet allocated (cluster is fully functional)
**Red** - some data is not available for whatever reason (cluster is partially functional)

### 创建索引

索引的名字必须是小写的，不可重名

```shell
# 创建一个名为 customer 的索引。pretty要求返回一个漂亮的json 结果
PUT /customer?pretty
# 索引的名字必须是小写的，不可重名
PUT twitter
{
    "settings" : {
        "index" : {
            "number_of_shards" : 5, 
            "number_of_replicas" : 2 
        }
    }
}
# 简写
PUT twitter
{
    "settings" : {
        "number_of_shards" : 5,
        "number_of_replicas" : 2
    }
}

# number_of_shards 分片数
# number_of_replicas 副本数
```

创建索引时的返回

```shell
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "twitter"
}
# 索引创建成功,所需数量的分片+副本启动成功
```

创建索引时加入映射定义

```shell
PUT test
{
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "_doc" : {
            "properties" : {
                "field1" : { "type" : "text" }
            }
        }
    }
}

```

查看索引的定义信息

```shell
GET /twitter
GET /twitter/_settings
GET /twitter/_mapping
```

判断索引是否存在

```shell
HEAD twitter
```

HTTP status code 表示结果 404 不存在 ， 200 存在

### 删除索引

```shell
DELETE /twitter
```

可以一次删除多个索引（以逗号间隔）
删除所有索引   _all 或 通配符 *

### 修改索引

索引的设置信息分为静态信息和动态信息两部分。静态信息不可更改，如索引的分片数。动态信息可以修改。

参考：https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#index-modules-settings

REST 访问端点： 
 /_settings  更新所有索引的。
 {index}/_settings     更新一个或多个索引的settings。

#### 修改备份数

```shell
PUT /twitter/_settings
{
    "index" : {
        "number_of_replicas" : 2
    }
}
```

#### 设置索引属性默认值

用null把某个属性设置成默认值

```shell
PUT /twitter/_settings
{
    "index" : {
        "refresh_interval" : null
    }
}
```

#### 设置索引读写

index.blocks.read_only：设为true,则索引以及索引的元数据只可读
index.blocks.read_only_allow_delete：设为true，只读时允许删除。
index.blocks.read：设为true，则不可读，默认值false。
index.blocks.write：设为true，则不可写，默认是false。
index.blocks.metadata：设为true，则索引元数据不可读写。

blocks 阻塞

### 索引模板 Index Templates

在创建索引时，为每个索引写定义信息可能是一件繁琐的事情。ES提供了索引模板功能，可以定义一个索引模板，模板中定义好settings、mapping、以及一个模式定义来匹配创建的索引。

模板只在索引创建时被参考，修改模板不会影响已创建的索引。

#### 创建索引模板

```shell
# 新增/修改名为tempae_1的模板，匹配名称为te* 或 bar*的索引创建。
PUT _template/template_1
{
  "index_patterns": ["te*", "bar*"],
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "_doc": {
      "_source": {
        "enabled": false
      },
      "properties": {
        "host_name": {
          "type": "keyword"
        },
        "created_at": {
          "type": "date",
          "format": "EEE MMM dd HH:mm:ss Z YYYY"
        }
      }
    }
  }
}
```

#### 查看索引模板

```shell
GET /_template/template_1
GET /_template/temp*
GET /_template/template_1,template_2
GET /_template
```

#### 删除模板

```shell
DELETE /_template/template_1
```

###  打开/关闭索引 Open / Close Index API

```shell
POST /my_index/_close
POST /my_index/_open
```

关闭的索引不能进行读写操作，几乎不占集群开销。
关闭的索引可以打开，打开走的是正常的恢复流程。

### 收缩索引 Shrink Index

索引的分片数是不可更改的，如要减少分片数可以通过收缩方式收缩为一个新的索引。新索引的分片数必须是原分片数的因子值，如原分片数是8，则新索引的分片数可以为4、2、1 。

**收缩的流程：**
先把所有主分片都转移到一台主机上；
在这台主机上创建一个新索引，分片数较小，其他设置和原索引一致；
把原索引的所有分片，复制（或硬链接）到新索引的目录下；
对新索引进行打开操作恢复分片数据；
(可选)重新把新索引的分片均衡到其他节点上。

**收缩前的准备工作：**
将原索引设置为只读；
将原索引各分片的一个副本重分配到同一个节点上，并且要是健康绿色状态。

```shell
PUT /my_source_index/_settings
{
  "settings": {
    "index.routing.allocation.require._name": "shrink_node_name", 
    "index.blocks.write": true 
  }
}
# shrink_node_name: 将索引的所有分片都复制到此节点上
```

**进行收缩：**

```shell
POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 1,
    "index.number_of_shards": 1, 
    "index.codec": "best_compression" 
  }
}
```

**监控收缩过程：**

```shell
GET _cat/recovery?v
GET _cluster/health
```

### 拆分索引 Split Index

当索引的分片容量过大时，可以通过拆分操作将索引拆分为一个倍数分片数的新索引。
能拆分为几倍由创建索引时指定的index.number_of_routing_shards 路由分片数决定。
这个路由分片数决定了根据一致性hash路由文档到分片的散列空间。

如index.number_of_routing_shards = 30 ，指定的分片数是5，则可按如下倍数方式进行拆分：

5 → 10 → 30 (split by 2, then by 3) 
5 → 15 → 30 (split by 3, then by 2) 
5 → 30 (split by 6) 

**注意**：只有在**创建时指定了index.number_of_routing_shards 的索引才可以进行拆分**，ES7开始将不再有这个限制。

拆分的工作原理:

1. 创建一个新的索引，其定义与源索引完全相同，但主分片数较多；
2. 将源索引中的段复制到目标索引（这个过程会比较耗时）；
3. 1
4. 它恢复了目标索引，好像它是一个刚重新打开的封闭索引。

**准备一个索引来做拆分：**

```shell
PUT my_source_index
{
    "settings": {
        "index.number_of_shards" : 1,
        "index.number_of_routing_shards" : 2 
    }
}
```

**先设置索引只读：**

```shell
PUT /my_source_index/_settings
{
  "settings": {
    "index.blocks.write": true 
  }
}
```

**执行拆分：**

```shell
POST my_source_index/_split/my_target_index
{
  "settings": {
    "index.number_of_shards": 2
  }
}
```

**监控拆分过程：**

```shell
GET _cat/recovery?v
GET _cluster/health
```

### Rollover Index 

别名滚动指向新创建的索引

对于有时效性的索引数据，如日志，过一定时间后，老的索引数据就没有用了。我们可以像数据库中根据时间创建表来存放不同时段的数据一样，在ES中也可用建多个索引的方式来分开存放不同时段的数据。比数据库中更方便的是ES中可以通过别名滚动指向最新的索引的方式，让你通过别名来操作时总是操作的最新的索引。

ES的rollover index API 让我们可以根据满足指定的条件（时间、文档数量、索引大小）创建新的索引，并把别名滚动指向新的索引。**这时的别名只能是一个索引的别名。**

Rollover Index 示例

```shell
# 创建一个名字为logs-0000001 、别名为logs_write 的索引
PUT /logs-000001 
{
  "aliases": {
    "logs_write": {}
  }
}
```

```shell
# 如果别名logs_write指向的索引是7天前（含）创建的或索引的文档数>=1000或索引的大小>= 5gb，
# 则会创建一个新索引logs-000002，并把别名logs_writer指向新创建的logs-000002索引
POST /logs_write/_rollover 
{
  "conditions": {
    "max_age":   "7d",
    "max_docs":  1000,
    "max_size":  "5gb"
  }
}
```

```shell
# Dry run 实际操作前先来个排练
# 排练不会创建索引，只是检测条件是否满足
POST /logs_write/_rollover?dry_run
{
  "conditions" : {
    "max_age": "7d",
    "max_docs": 1000,
    "max_size": "5gb"
  }
}
```

rollover是你请求它才会进行操作，并不是自动在后台进行的。你可以周期性地去请求它。

**在名称中使用Date math（时间表达式）**
如果你希望生成的索引名称中带有日期，如logstash-2016.02.03-1 ，则可以在创建索引时采用时间表达式来命名：

```shell
# PUT /<logs-{now/d}-1> with URI encoding:
# PUT /<logs-{now%2Fd}-1>
PUT /%3Clogs-%7Bnow%2Fd%7D-1%3E 
{
  "aliases": {
    "logs_write": {}
  }
}
# 
PUT logs_write/_doc/1
{
  "message": "a dummy log"
}
# 
POST logs_write/_refresh
#
POST /logs_write/_rollover 
{
  "conditions": {
    "max_docs":   "1"
  }
}
```

### 索引段 Indices Segments

查看索引段信息

https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-segments.html

```shell
GET /test/_segments
GET /index1,index2/_segments
GET /_segments
```

查看索引恢复信息

https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-recovery.html

```shell
GET index1,index2/_recovery?human
GET /_recovery?human
```

查看索引分片的存储信息
https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-shards-stores.html

```shell
# return information of only index test
GET /test/_shard_stores
# return information of only test1 and test2 indices
GET /test1,test2/_shard_stores
# return information of all indices
GET /_shard_stores
#
GET /_shard_stores?status=green
```

### 状态管理 Indices Stats

查看索引状态信息
https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-stats.html

```shell
GET /_stats
GET /index1,index2/_stats
```

### 清理缓存Clear Cache 

```shell
POST /twitter/_cache/clear
# 默认会清理所有缓存，可指定清理query, fielddata or request 缓存
POST /kimchy,elasticsearch/_cache/clear 
POST /_cache/clear
```

### Refresh

重新打开读取索引。周期性自动执行。

解析： 当索引一个文档，文档先是被存储在内存里面，默认1秒后，会进入文件系统缓存，这样该文档就可以被搜索到，但是该文档还没有存储到磁盘上，如果机器宕机了，数据就会丢失。因此fresh实现的是从内存到文件系统缓存的过程。

```shell
# 刷新单个索引
POST / twitter / _refresh
# 刷新多个索引
POST /kimchy,elasticsearch/_refresh
# 刷新所有索引
POST /_refresh
```

### Flush

将缓存在内存中的索引数据刷新到持久存储中

```shell
POST twitter/_flush
```

### 强制段合并Force merge 

```shell
POST /kimchy/_forcemerge?only_expunge_deletes=false&max_num_segments=100&flush=true

POST /kimchy,elasticsearch/_forcemerge
POST /_forcemerge
```

可选参数说明：
max_num_segments  合并为几个段，默认1
only_expunge_deletes  是否只合并含有删除文档的段，默认false
flush 合并后是否刷新，默认true

### 索引设置 Index Settings

#### 静态索引设置 Static index settings

只有在创建索引时或关闭索引时才能设置

1. index.number_of_shards 
   主分片数， 默认值是5，上限是1024。只能在创建索引的时候设定， 不管是索引关闭与否，都不能修改。
2. index.shard.check_on_startup 
   在索引打开前是否检查分片，取值如下：
   + false 不检查
   + checksum 检查物理损坏
   + true 检查物理和逻辑损坏
   + fix 7.0版本将会被废弃
3. index.codec 
   默认数据压缩策略是`LZ4`，也可以设置成`best_compression` ，压缩率更高，但是性能较慢
4. index.routing_partition_size
   自定义分片的路由值，默认为1。这个值必须小于`index.number_of_shards`，除非`index.number_of_shards`等于1

#### 动态索引设置 Dynamic index settings

随时可以修改

1. index.number_of_replicas
2. index.auto_expand_replicas
3. index.refresh_interval
4. index.max_result_window
5. index.max_inner_result_window
6. index.max_rescore_window
7. index.max_docvalue_fields_search
8. index.max_script_fields
9. index.max_ngram_diff
10. index.max_shingle_diff
11. index.blocks.read_only
12. index.blocks.read_only_allow_delete
13. index.blocks.read
14. index.blocks.write
15. index.blocks.metadata
16. index.max_refresh_listeners
17. index.highlight.max_analyzed_offset
18. index.max_terms_count
19. index.routing.allocation.enable
20. index.routing.rebalance.enable
21. index.gc_deletes
22. index.max_regex_length
23. index.default_pipeline

### 索引映射 Mapping 

#### Mapping 映射是什么

映射定义索引中有什么字段、字段的类型等结构信息。相当于数据库中表结构定义，或 solr中的schema。因为lucene索引文档时需要知道该如何来索引存储文档的字段。
ES中支持手动定义映射，动态映射两种方式。

**Create Index  with mapping**

```shell
# 映射定义后续可以修改
PUT test
{
	"mappings" : {
        "type1" : { # 名为type1的映射类别 mapping type
            "properties" : { # 字段定义
                "field1" : { "type" : "text" }
            }
        }
    }
}
```

**映射类别 Mapping type 废除说明**

ES最先的设计是用索引类比关系型数据库，用mapping type 来类比表，一个索引中可以包含多个映射类别。这个类比存在一个严重的问题，就是当多个mapping type中存在同名字段时（特别是同名字段还是不同类型的），在一个索引中不好处理，因为搜索引擎中只有 索引-文档的结构，不同映射类别的数据都是一个一个的文档（只是包含的字段不一样而已）

从6.0.0开始限定仅包含一个映射类别定义（ "index.mapping.single_type": true ），兼容5.x中的多映射类别。从7.0开始将移除映射类别。
为了与未来的规划匹配，请现在将这个唯一的映射类别名定义为“_doc”,因为索引的请求地址将规范为：PUT {index}/_doc/{id} and POST {index}/_doc

多个映射类别就转变为多个索引。如果你还想要在一个索引中放多类数据，也可像在数据库中定义骷髅表一样，通过定义一个数据类别字段，来区分不同的数据。

**Mapping 映射示例**

```shell
PUT twitter
{
  "mappings": {
    "_doc": {
      "properties": {
        "type": { "type": "keyword" }, 
        "name": { "type": "text" },
        "user_name": { "type": "keyword" },
        "email": { "type": "keyword" },
        "content": { "type": "text" },
        "tweeted_at": { "type": "date" }
      }
    }
  }
}
```

**多映射类别数据转储到独立的索引中**

```shell
PUT users
{
  "settings": {
    "index.mapping.single_type": true
  },
  "mappings": {
    "_doc": {
      "properties": {
        "name": {
          "type": "text"
        },
        "user_name": {
          "type": "keyword"
        },
        "email": {
          "type": "keyword"
        }
      } }  
  }
}
```

```shell
POST _reindex
{
  "source": {
    "index": "twitter",
    "type": "user"
  },
  "dest": {
    "index": "users"
  }
}
```

转储到骷髅索引

```shell
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  },
  "script": {
    "source": """
      ctx._source.type = ctx._type;
      ctx._id = ctx._type + '-' + ctx._id;
      ctx._type = '_doc';
    """
  }
}
```

#### 动态映射 Dynamic Mapping

ES中提供的重要特性，让我们可以快速使用ES，而不需要先创建索引、定义映射。 
如我们直接向ES提交文档进行索引，ES将自动为我们创建data索引、_doc 映射、类型为 long 的字段 count

索引文档时，当有新字段时， ES将根据我们字段的json的数据类型为我们自动加人字段定义到mapping中。

##### Date detection  时间侦测

date_detection 默认是开启的，默认的格式dynamic_date_formats为：
[ "strict_date_optional_time","yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z"]

"2019/01/01"、"2019/01/01 10:10:12" 都会被映射成date类型，
同时还会指定format为"yyyy/MM/dd HH:mm:ss||yyyy/MM/dd||epoch_millis"

"2019-01-01"、"2019-01-01T10:10:12"、"2019-01-01T10:10:12.235Z"也会被映射成date类型，但是不会指定format

```shell
PUT my_index/_doc/1
{
  "create_date": "2015/09/02"
}

GET my_index/_mapping
```

##### Numeric detection  数值侦测

数值侦测（默认是禁用的）

```shell
PUT my_index
{
  "mappings": {
    "_doc": {
      "numeric_detection": true 
    }
  }
}

PUT my_index/_doc/1
{
  "my_float":   "1.0", 
  "my_integer": "1" 
}
```

#### 字段类型 Field datatypes

https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html

##### Core Datatypes 核心类型

string
    **text** and **keyword** 
Numeric datatypes
    **long, integer, short, byte, double, float, half_float, scaled_float** 
Date datatype
    **date** 
Boolean datatype
    **boolean** 
Binary datatype
    **binary** 
Range datatypes     范围
    **integer_range, float_range, long_range, double_range, date_range**

**Complex datatypes  复合类型**

Array datatype
    **array**数组就是多值，不需要专门的类型
	事实上没有array类型，
Object datatype
    **object** ：表示值为一个JSON 对象 
Nested datatype
    **nested**：for arrays of JSON objects（表示值为JSON对象数组 ）

**Geo datatypes  地理数据类型**

Geo-point datatype
    **geo_point**： for lat/lon points  （经纬坐标点）
Geo-Shape datatype
    **geo_shape**： for complex shapes like polygons （形状表示）

**Specialised datatypes  特别的类型**

IP datatype
    ip： for IPv4 and IPv6 addresses 
Completion datatype
    completion： to provide auto-complete suggestions 
Token count datatype
    token_count： to count the number of tokens in a string 
mapper-murmur3
    murmur3： to compute hashes of values at index-time and store them in the index 
Percolator type
    Accepts queries from the query-dsl 
join datatype
    Defines parent/child relation for documents within the same index

##### 多重字段 Multi Field

当我们需要对一个字段进行多种不同方式的索引时，可以使用fields多重字段定义。如一个字符串字段即需要进行text分词索引，也需要进行keyword 关键字索引来支持排序、聚合；或需要用不同的分词器进行分词索引。

Multi Field 多重字段—示例

```shell
# 创建索引
# raw是一个多重版本名（自定义）
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "city": {
          "type": "text",
          "fields": {
            "raw": { 
              "type":  "keyword"
            }
          }
        }
      }
    }
  }
}
```

```shell
# 添加数据
PUT my_index/_doc/1
{
  "city": "New York"
}

PUT my_index/_doc/2
{
  "city": "York"
}
```

```shell
# 查询
GET my_index/_search
{
  "query": {
    "match": {
      "city": "york" 
    }
  },
  "sort": {
    "city.raw": "asc" 
  },
  "aggs": {
    "Cities": {
      "terms": {
        "field": "city.raw" 
      }
    }
  }
}
```

#### 字段定义属性 Mapping parameters

字段的type (Datatype)定义了如何索引存储字段值，还有一些属性可以让我们根据需要来覆盖默认的值或进行特别定义。

https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html

analyzer   指定分词器
normalizer   指定标准化器
boost        指定权重值
coerce      强制类型转换
copy_to    值复制给另一字段
doc_values  是否存储docValues
dynamic
enabled    字段是否可用
fielddata
eager_global_ordinals
format    指定时间值的格式
ignore_above 忽略大于或超长的部分

ignore_malformed 忽略异常的值，如需要数值，实际上的值是字符
index_options 索引选项
index 是否索引
fields 多重字段
norms 是否标准化
null_value 给null指定一个值
position_increment_gap 多值字段，短语查询，位置的增量
properties 如果是对象，对象里边还会有属性
search_analyzer 搜索时的分词器
similarity 相关性计算算法或模型
store 是否存储，默认为false
term_vector 词项向量

#### 元字段 Meta-Fields

元字段是ES中定义的文档字段，有以下几类：

**Identity meta-fields**
\_**index**  The index to which the document belongs.
**_uid**  A composite field consisting of the _type and the _id.
	6.0开始被废弃
\_**type**  The document’s mapping type.
\_**id**  The document’s ID.

**Document source meta-fields**
**_source** 是否存储文档的原JSON  对象，默认是true 存储.
The original JSON representing the body of the document.
**_size**  The size of the _source field in bytes, provided by the mapper-size plugin.

**Indexing meta-fields**
**_all**  A catch-all field that indexes the values of all other fields. Disabled by default.
	复制文档的其他字段值到这一个字段上进行索引，默认禁用。6.0以后版本废弃该字段，改用copy-to
**_field_names**  All fields in the document which contain non-null values.

**Routing meta-field**
**_routing**  A custom routing value which routes a document to a particular shard.
	添加文档是需要制定routing参数

**Other meta-field**
**_meta**  Application specific metadata.

### 索引别名 Index Aliases

#### 别名的用途

如果希望一次查询可查询多个索引。
如果希望通过索引的视图来操作索引，就像数据库中的视图一样。

索引的别名机制，就是让我们可以以视图的方式来操作集群中的索引，这个视图可是多个索引，也可是一个索引或索引的一部分。

**新建索引时定义别名**

```shell
PUT /logs_20162801
{
    "mappings" : {
        "type" : {
            "properties" : {
                "year" : {"type" : "integer"}
            }
        }
    },
    "aliases" : {
        "current_day" : {},
        "2016" : {
            "filter" : {
                "term" : {"year" : 2016 }
            }
        }
    }
}
```

**创建别名**     /_aliases

```shell
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "test1", "alias" : "alias1" } }
    ]
}
```

**删除别名**

```shell
POST /_aliases
{
    "actions" : [
        { "remove" : { "index" : "test1", "alias" : "alias1" } }
    ]
}

# 或者
DELETE /{index}/_alias/{name}
```

**批量操作**

```shell
POST /_aliases
{
    "actions" : [
        { "remove" : { "index" : "test1", "alias" : "alias1" } },
        { "add" : { "index" : "test2", "alias" : "alias1" } }
    ]
}
```

**为多个索引定义别名**

```shell
# 方式一：
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "test1", "alias" : "alias1" } },
        { "add" : { "index" : "test2", "alias" : "alias1" } }
    ]
}
# 方式二：
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "test*", "alias" : "all_test_indices" } }
    ]
}
# 方式三：通过统配符*模式来指定要别名的索引
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "test*", "alias" : "all_test_indices" } }
    ]
}
# 注意：在这种情况下，别名是一个点时间别名，它将对所有匹配的当前索引进行别名，
# 当添加/删除与此模式匹配的新索引时，它不会自动更新
```

**带过滤器的别名**
索引中需要有字段。

```shell
PUT /test1
{
  "mappings": {
    "type1": {
      "properties": {
        "user" : {
          "type": "keyword"
        }
      }
    }
  }
}
```

过滤器通过Query DSL来定义，将作用于通过该别名来进行的所有Search, Count, Delete By Query and More Like This 操作。

```shell
POST /_aliases
{
    "actions" : [
        {
            "add" : {
                 "index" : "test1",
                 "alias" : "alias2",
                 "filter" : { "term" : { "user" : "kimchy" } }
            }
        }
    ]
}
```

**带routing的别名**

可在别名定义中指定路由值，可和filter一起使用，用来限定操作的分片，避免不需要的其他分片操作。

```shell
POST /_aliases
{
    "actions" : [
        {
            "add" : {
                 "index" : "test",
                 "alias" : "alias1",
                 "routing" : "1"
            }
        }
    ]
}
```

为搜索、索引指定不同的路由值

```shell
{
    "actions" : [
        {
            "add" : {
                 "index" : "test",
                 "alias" : "alias2",
                 "search_routing" : "1,2",
                 "index_routing" : "2"
            }
        }
    ]
}
```

下面的查询本身指定了路由值，请问它最终使用的路由值是多少？

```shell
GET /alias2/_search?q=user:kimchy&routing=2,3
```

最终使用的是2，因为索引中定义的是1,2

**以PUT方式来定义一个索引**

```shell
PUT /{index}/_alias/{name}
```

带filter 和 routing

```shell
PUT /users
{
    "mappings" : {
        "user" : {
            "properties" : {
                "user_id" : {"type" : "integer"}
            }
        }
    }
}
```

```shell
PUT /users/_alias/user_12
{
    "routing" : "12",
    "filter" : {
        "term" : {
            "user_id" : 12
        }
    }
}
```

**查看别名定义信息**

```shell
GET /{index}/_alias/{alias}
GET /logs_20162801/_alias/*
GET /_alias/2016
GET /_alias/20*
```



























