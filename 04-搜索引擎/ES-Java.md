## ES Client 简介

### ES支持的客户端连接方式

REST API ，端口 9200
Transport 连接    端口 9300

### ES提供了多种编程语言客户端

- [Java REST Client [6.5\]](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html) — [other versions](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/index.html)
- [Java API [6.5\]](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/index.html) — [other versions](https://www.elastic.co/guide/en/elasticsearch/client/java-api/index.html)
- [JavaScript API](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/index.html)
- [Groovy API [2.4\]](https://www.elastic.co/guide/en/elasticsearch/client/groovy-api/current/index.html) — [other versions](https://www.elastic.co/guide/en/elasticsearch/client/groovy-api/index.html)
- [.NET API [6.x\]](https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/index.html) — [other versions](https://www.elastic.co/guide/en/elasticsearch/client/net-api/index.html)
- [PHP API [6.0\]](https://www.elastic.co/guide/en/elasticsearch/client/php-api/current/index.html) — [other versions](https://www.elastic.co/guide/en/elasticsearch/client/php-api/index.html)
- [Perl API](https://www.elastic.co/guide/en/elasticsearch/client/perl-api/current/index.html)
- [Python API](https://www.elastic.co/guide/en/elasticsearch/client/python-api/current/index.html)
- [Ruby API](https://www.elastic.co/guide/en/elasticsearch/client/ruby-api/current/index.html)
- [Community Contributed Clients](https://www.elastic.co/guide/en/elasticsearch/client/community/current/index.html)

### Java REST API

### ES提供了两个JAVA REST client 版本

Java Low Level REST Client: 低级别的REST客户端，通过http与集群交互，用户需自己编组请求JSON串，及解析响应JSON串。兼容所有ES版本。
  Java High Level REST Client: 高级别的REST客户端，基于低级别的REST客户端，增加了编组请求、解析响应等相关api。

#### Java High Level REST Client 说明

从6.0.0开始加入的，目的是以java面向对象的方式来进行请求、响应处理。
  每个API 支持 同步/异步 两种方式，同步方法直接返回一个结果对象。异步的方法以 async 为后缀，通过listener 参数来通知结果。
高级java REST 客户端依赖Elasticsearch core project

**兼容性说明**

依赖 java1.8  和 Elasticsearch core project
请使用与服务端ES版本一致的客户端版本

