## Solr提供的数据提交方式简介

Solr中数据提交进行索引都是通过http请求，针对不同的数据源solr提供了几种方式来方便提交数据。

基于Apache  Tika 的 solr cell(Solr Content Extraction Library )，来提取上传文件内容进行索引。
应用中通过Index handler（即 index API）来提交数据。
通过Data Import Handler 来提交结构化数据源的数据

Post工具：提交服务器上的xml、json、csv数据文件及pdf、excel等富文本文件。

### Index handler

 Index handler 索引处理器，是一种Request handler 请求处理器。
solr对外提供http服务，每类服务在solr中都有对应的request handler来接收处理，solr中提供了默认的处理器实现，如有需要我们也可提供我们的扩展实现，并在conf/solrconfig.xml中进行配置。

在 conf/solrconfig.xml中，requestHandler的配置就像我们在web.xml中配置servlet-mapping（或spring mvc 中配置controller 的requestMap）一样：配置该集合/内核下某个请求地址的处理类。

Solrconfig中通过updateHandler元素配置了一个统一的更新请求处理器支持XML、CSV、JSON和javabin更新请求（映射地址为/update），它根据请求提交内容流的内容类型Content-Type将其委托给适当的ContentStreamLoader来解析内容，再进行索引更新。

配置一个requestHandler示例

```xml
<requestHandler name=“/update" class="solr.UpdateRequestHandler" />
```

Solr支持从关系数据库、基于http的数据源(如RSS和ATOM提要)、电子邮件存储库和结构化XML 中索引内容。

### 索引更新

#### Xml 格式数据索引更新

请求头中设置 Content-type: application/xml or Content-type: text/xml

##### **添加、替换文档**

<add> 操作，支持两个可选属性：
commitWithin：限定在多少毫秒内完成
overwrite：指定当唯一键已存在时是否覆盖，默认true。

```xml
<add>
  <doc>
    <field name="authors">Patrick Eagar</field>
    <field name="subject">Sports</field>
    <field name="dd">796.35</field>
    <field name="numpages">128</field>
    <field name="desc"></field>
    <field name="price">12.40</field>
    <field name="title">Summer of the all-rounder</field>
    <field name="isbn">0002166313</field>
    <field name="yearpub">1982</field>
    <field name="publisher">Collins</field>
  </doc>
  <doc>
  ...
  </doc>
</add>
```

##### **删除文档**

<delete> 操作，支持两种删除方式：
1、根据唯一键
2、根据查询

```xml
<delete>
  <id>0002166313</id>
  <id>0031745983</id>
  <query>subject:sport</query>
  <query>publisher:penguin</query>
</delete>
```

##### **组合操作**

```xml
<update>
  <add>
    <doc><!-- doc 1 content --></doc>
  </add>
  <add>
    <doc><!-- doc 2 content --></doc>
  </add>
  <delete>
    <id>0002166313</id>
  </delete>
</update>
```

```xml
<response>
  <lst name="responseHeader">
    <int name="status">0</int>
    <int name="QTime">127</int>
  </lst>
</response>
```

##### 提交、优化、回滚操作

在集群环境，一定不要用优化

```xml
<commit waitSearcher="false"/>
<commit waitSearcher="false" expungeDeletes="true"/>
<optimize waitSearcher="false"/>
<rollback/>
```

commit、optimize 属性说明：
waitSearcher：默认true，阻塞等待打开一个新的IndexSearcher并注册为主查询searcher,来让提交的改变可见。
expungeDeletes： (commit only) 默认false，合并删除文档量占比超过10%的段，合并过程中删除这些已删除的文档。
maxSegments： (optimize only) 默认1，优化时，将段合并为最多多少个段

#### JSON 格式数据索引更新

请求头中设置 Content-Type: application/json or Content-Type: text/json

##### 添加、替换一个文档

```json
{
  "id": "1",
  "title": "Doc 1"
}
```

#### 添加、替换多个文档

```json
[
  {
    "id": "1",
    "title": "Doc 1"
  },
  {
    "id": "2",
    "title": "Doc 2"
  }
]
```

#### 在json中指定多个操作

```json
{
  "add": {
    "doc": {
      "id": "DOC1",
      "my_field": 2.3,
      "my_multivalued_field": [ "aaa", "bbb" ]   
    }
  },
  "add": {
    "commitWithin": 5000, 
    "overwrite": false,  
    "doc": {
      "f1": "v1", 
      "f1": "v2"
    }
  },
  "commit": {},
  "optimize": { "waitSearcher":false },
  "delete": { "id":"ID" },  
  "delete": { "query":"QUERY" } 
}
```

#### 根据唯一键删除

```json
{ "delete":"myid" }
```

```json
{ "delete":["id1","id2"] }
```


