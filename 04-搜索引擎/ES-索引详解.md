# 认识分词器

## Analyzer 分析器

在ES中一个Analyzer 由下面三种组件组合而成：

**character filter** ：字符过滤器，对文本进行字符过滤处理，如处理文本中的html标签字符。处理完后再交给tokenizer进行分词。一个analyzer中可包含0个或多个字符过滤器，多个按配置顺序依次进行处理。

**tokenizer**：分词器，对文本进行分词。一个analyzer必需且只可包含一个tokenizer。

**token filter**：词项过滤器，对tokenizer分出的词进行过滤处理。如转小写、停用词处理、同义词处理。一个analyzer可包含0个或多个词项过滤器，按配置顺序进行过滤。

### 测试分词器

```shell
POST _analyze
{
  "analyzer": "whitespace",
  "text":     "The quick brown fox."
}
# 制定词项过滤器
POST _analyze
{
  "tokenizer": "standard",
  "filter":  [ "lowercase", "asciifolding" ],
  "text":      "Is this déja vu?"
}
# IK : ik_smart,ik_max_word
POST /_analyze
{
  "analyzer": "ik_smart",
  "text": "Undertow是一个采用Java开发的灵活的高性能Web服务器"
}
```

### 内建的character filter

HTML Strip Character Filter
	**html_strip** ：过滤html标签，解码HTML entities like &amp;. 

```shell
POST _analyze
{
  "tokenizer":      "keyword", 
  "char_filter":  [ "html_strip" ],
  "text": "<p>I&apos;m so <b>happy</b>!</p>"
}
```

在索引中配置：

```shell
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "keyword",
          "char_filter": ["my_char_filter"]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "html_strip",
          "escaped_tags": ["b"]
        }
      }
    }
  }
}
# 测试
POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "<p>I&apos;m so <b>happy</b>!</p>"
}
```

Mapping Character Filter
	**mapping** ：用指定的字符串替换文本中的某字符串。
	https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-mapping-charfilter.html
Pattern Replace Character Filter 
	https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pattern-replace-charfilter.html
	**pattern_replace** ：进行正则表达式替换。

### 内建的Tokenizer

Standard Tokenizer
Letter Tokenizer
Lowercase Tokenizer
Whitespace Tokenizer
UAX URL Email Tokenizer
Classic Tokenizer
Thai Tokenizer
NGram Tokenizer
Edge NGram Tokenizer
Keyword Tokenizer
Pattern Tokenizer
Simple Pattern Tokenizer
Simple Pattern Split Tokenizer
Path Hierarchy Tokenizer
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html
集成的中文分词器Ikanalyzer中提供的tokenizer：ik_smart 、 ik_max_word

测试tokenizer

```shell
POST _analyze
{
  "tokenizer": "standard", 
  "text": "张三说的确实在理"
}

POST _analyze
{
  "tokenizer": "ik_smart", 
  "text": "张三说的确实在理"
}
```

#### 内建的Token Filter

ES中内建了很多Token filter ，详细了解：
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html 

Lowercase Token Filter  ：lowercase 转小写
Stop Token Filter ：stop 停用词过滤器
Synonym Token Filter： synonym 同义词过滤器

**Synonym Token Filter  同义词过滤器**

```shell
PUT /test_index
{
    "settings": {
        "index" : {
            "analysis" : {
                "analyzer" : {
                    "my_ik_synonym" : {
                        "tokenizer" : "ik_smart",
                        "filter" : ["synonym123"]
                    }
                },
                "filter" : {
                    "synonym123" : {
                        "type" : "synonym",
                        "synonyms_path" : "analysis/synonym.txt"
                    }
                }
            }
        }
    }
}
# synonyms_path：指定同义词文件（相对config的位置）
```

##### 同义词定义格式

ES同义词格式支持 solr、 WordNet 两种格式。
在analysis/synonym.txt中用solr格式定义如下同义词

```shell
张三,李四
电饭煲,电饭锅 => 电饭煲
电脑 => 计算机,computer
```

测试

```shell
POST test_index/_analyze
{
  "analyzer": "my_ik_synonym",
  "text": "张三说的确实在理"
}

POST test_index/_analyze
{
  "analyzer": "my_ik_synonym",
  "text": "我想买个电饭锅和一个电脑"
}
```

#### 内建的Analyzer

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html
Standard Analyzer
Simple Analyzer
Whitespace Analyzer
Stop Analyzer
Keyword Analyzer
Pattern Analyzer
Language Analyzers
Fingerprint Analyzer

##### 自定义Analyzer

zero or more character filters
a tokenizer
zero or more token filters. 
配置参数：
**tokenizer**  Required
**char_filter**  字符过滤器数组
**filter**  词项过滤器器数组
**position_increment_gap** 
多值间的词项position增长间隔，默认是100。这个是用来解决多值字段的短语查询不准确问题的。

```shell
PUT my_index8
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_ik_analyzer": {
          "type": "custom",
          "tokenizer": "ik_smart",
          "char_filter": [
            "html_strip"
          ],
          "filter": [
             "synonym"
          ]
        }
      },
      "filter": {
        "synonym": {
          "type": "synonym",
          "synonyms_path": "analysis/synonym.txt"
        }
      }    
    }  
  }
}
```

##### 为字段指定分词器

```shell
PUT my_index8/_mapping/_doc
{
  "properties": {
    "title": {
        "type": "text",
        "analyzer": "my_ik_analyzer"
    }
  }
}
```

```shell
PUT my_index8/_mapping/_doc
{
  "properties": {
    "title": {
        "type": "text",
        "analyzer": "my_ik_analyzer",
        "search_analyzer": "other_analyzer" 
    }
  }
}
```

```shell
PUT my_index8/_doc/1
{
  "title": "张三说的确实在理"
}

GET /my_index8/_search
{
  "query": {
    "term": {
      "title": "张三"
    }
  }
}
```

**为索引定义个default分词器**

```shell
PUT /my_index10
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "tokenizer": "ik_smart",
          "filter": [
            "synonym"
          ]
        }
      },
      "filter": {
        "synonym": {
          "type": "synonym",
          "synonyms_path": "analysis/synonym.txt"
        }
      }
    }
  },
 "mappings": {
    "_doc": {
      "properties": {
        "title": {
          "type": "text"
        }
      }
    }
  }
}
```

Analyzer的使用顺序

可以为每个查询、每个字段、每个索引指定分词器。
在索引阶段ES将按如下**顺序来选用分词**：
首先选用字段mapping定义中指定的analyzer
字段定义中没有指定analyzer，则选用 index settings中定义的名字为default 的analyzer。
如index setting中没有定义default分词器，则使用 standard analyzer.



### 文档管理

#### 新建文档

```shell
# 指定文档id，新增/修改
PUT twitter/_doc/1
{
    "id": 1,
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
# 新增，自动生成文档id
POST twitter/_doc/
{
    "id": 1,
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

```

#### 获取单个文档

```shell
HEAD twitter/_doc/11 # 查看是否存储
GET twitter/_doc/1
GET twitter/_doc/1?_source=false
GET twitter/_doc/1/_source
# 获取存储字段
GET twitter11/_doc/1?stored_fields=tags,counter
```

#### 获取多个文档 _mget

```shell
# 方式一：
GET /_mget
{
    "docs" : [
        {
            "_index" : "twitter",
            "_type" : "_doc",
            "_id" : "1"
        },
        {
            "_index" : "twitter",
            "_type" : "_doc",
            "_id" : "2"
            "stored_fields" : ["field3", "field4"]
        }
    ]
}
# 方式二：
GET /twitter/_mget
{
    "docs" : [
        {
            "_type" : "_doc",
            "_id" : "1"
        },
        {
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
# 方式三：
GET /twitter/_doc/_mget
{
    "docs" : [
        {
            "_id" : "1"
        },
        {
            "_id" : "2"
        }
    ]
}
# 方式四：ids 表示文档ID
GET /twitter/_doc/_mget
{
    "ids" : ["1", "2"]
}
```

#### 删除文档

```shell
# 指定文档id进行删除
DELETE twitter/_doc/1

# 用版本来控制删除
DELETE twitter/_doc/1?version=1

# 查询删除
POST twitter/_delete_by_query
{
  "query": { 
    "match": {
      "message": "some message"
    }
  }
}

# 当有文档有版本冲突时，不放弃删除操作（记录冲突的文档，继续删除其他复合查询的文档）
POST twitter/_doc/_delete_by_query?conflicts=proceed
{
  "query": {
    "match_all": {}
  }
}

# 通过task api 来查看 查询删除任务
GET _tasks?detailed=true&actions=*/delete/byquery
# 查询具体任务的状态
GET /_tasks/taskId:1
# 取消任务
POST _tasks/task_id:1/_cancel
```

#### 更新文档

```shell
# 指定文档id进行修改
PUT twitter/_doc/1
{
    "id": 1,
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
# 乐观锁并发更新控制,通过version
PUT twitter/_doc/1?version=1
{
    "id": 1,
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

##### 通过脚本来更新文档 

Scripted update
脚本说明：painless是es内置的一种脚本语言，ctx执行上下文对象（通过它还可访问_index, _type, _id, _version, _routing and _now (the current timestamp) ），params是参数集合
脚本更新要求索引的_source 字段是启用的。
**更新执行流程**：
1、获取到原文档
2、通过_source字段的原始数据，执行脚本修改。
3、删除原索引文档
4、索引修改后的文档 
它只是降低了一些网络往返，并减少了get和索引之间版本冲突的可能性

```shell
# 1、准备一个文档
PUT uptest/_doc/1
{
    "counter" : 1,
    "tags" : ["red"]
}
# 2、对文档1的counter + 4
POST uptest/_doc/1/_update
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    }
}
# 3、往数组中加入元素
POST uptest/_doc/1/_update
{
    "script" : {
        "source": "ctx._source.tags.add(params.tag)",
        "lang": "painless",
        "params" : {
            "tag" : "blue"
        }
    }
}
# 4、添加一个字段
POST uptest/_doc/1/_update
{
    "script" : "ctx._source.new_field = 'value_of_new_field'"
}
# 5、移除一个字段
POST uptest/_doc/1/_update
{
    "script" : "ctx._source.remove('new_field')"
}
# 6、判断删除或不做什么
POST uptest/_doc/1/_update
{
    "script" : {
        "source": 
        "if (ctx._source.tags.contains(params.tag)) { ctx.op = 'delete' } else { ctx.op = 'none' }",
        "lang": "painless",
        "params" : {
            "tag" : "green"
        }
    }
}
# 7、合并传人的文档字段进行更新
POST uptest/_doc/1/_update
{
    "doc" : {
        "name" : "new_name"
    }
}
# 8、再次执行7，更新内容相同，不需做什么
# 9、设置不做noop检测
POST uptest/_doc/1/_update
{
    "doc" : {
        "name" : "new_name"
    },
    "detect_noop": false
}
# 10、upsert 操作：如果要更新的文档存在，则执行脚本进行更新，如不存在，则把 upsert中的内容作为一个新文档写入
POST uptest/_doc/1/_update
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    },
    "upsert" : {
        "counter" : 1
    }
}
```

##### 查询更新

```shell
# 通过条件查询来更新文档
POST twitter/_update_by_query
{
  "script": {
    "source": "ctx._source.likes++",
    "lang": "painless"
  },
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```

##### 批量操作

批量操作API /_bulk 让我们可以在一次调用中执行多个索引、删除操作。这可以大大提高索引数据的速度。批量操作内容体需按如下以新行分割的json结构格式给出：

```shell
POST _bulk
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "_doc", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "_doc", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```

请求端点可以是:  /_bulk,  /{index}/_bulk,  {index}/{type}/_bulk

##### 批量索引多个文档

```shelll
curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_doc/_bulk?pretty&refresh" \
--data-binary "@accounts.json"
```

#### 重索引

Reindex API /_reindex 让我们可以将一个索引中的数据重索引到另一个索引中（拷贝），要求源索引的_source 是开启的。目标索引的setting 、mapping 信息与源索引无关。

```shell
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

重索引要考虑的一个问题：目标索引中存在源索引中的数据，这些数据的version如何处理。
1、如果没有指定version_type 或指定为 internal，则会是采用目标索引中的版本，重索引过程中，执行的就是新增、更新操作。

```shell
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "internal"
  }
}
```

2、如果想使用源索引中的版本来进行版本控制更新，则设置 version_type 为external。
重索引操作将写入不存在的，更新旧版本的数据

```shell
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "external"
  }
}
```

如果只想从源索引中复制目标索引中不存在的文档数据，可以指定 op_type 为 create 。
此时存在的文档将触发 版本冲突（会导致放弃操作），可设置“conflicts”: “proceed“，跳过继续。

```shell
POST _reindex
{
  "conflicts": "proceed",
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "op_type": "create"
  }
}
```

可以只索引源索引的一部分数据，通过 type 或 查询来指定你需要的数据

```shell
POST _reindex
{
  "source": {
    "index": "twitter",
    "type": "_doc",
    "query": {
      "term": {
        "user": "kimchy"
      }
    }
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

从多个源获取数据

```shell
POST _reindex
{
  "source": {
    "index": ["twitter", "blog"],
    "type": ["_doc", "post"]
  },
  "dest": {
    "index": "all_together"
  }
}
```

##### ？refresh

对于索引、更新、删除操作如果想操作完后立马重刷新可见，可带上refresh参数。

```shell
PUT /test/_doc/1?refresh
{"test": "test"}
PUT /test/_doc/2?refresh=true
{"test": "test"}
```

refresh 可选值说明
未给值或=true，则立马会重刷新读索引。
=false ，相当于没带refresh 参数，遵循内部的定时刷新。
=wait_for ，登记等待刷新，当登记的请求数达到index.max_refresh_listeners 参数设定的值时(defaults to 1000)，将触发重刷新。

