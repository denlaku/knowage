# Solr安装

## Solr独立服务器模式启动

### 1、启动

Windows执行 `solr.cmd start` 
Linux执行`solr start`
默认端口是8983，当然可以通过`-p`指定端口
默认的solr home目录是`solr-7.3.0/server/solr`,可以通过`-s`指定solr home
`solr.cmd start -s D:/myspace/solr/solr-7.3.0/server/mysolr -p 8984`

solr home目录下一定要有solr.xml文件

### 2、查看Solr状态

 Windows执行`solr.cmd status`
Linux执行`solr start`

```json
{
  "solr_home":"D:\\myspace\\solr\\solr-7.3.0\\server\\solr",
  "version":"7.3.0 98a6b3d642928b1ac9076c6c5a369472581f7633 - woody - 2018-03-28 14:37:45",
  "startTime":"2019-01-19T04:09:40.895Z",
  "uptime":"0 days, 0 hours, 1 minutes, 50 seconds",
  "memory":"33.8 MB (%6.9) of 490.7 MB"
}
```

### 3、停止

`solr stop -p 8983`

### 4、web界面

core admin 内核管理菜单

### 5、Solr命令

start, stop, restart, healthcheck, create, create_core, create_collection, delete, version, zk, auth, assert, config

create命令相当于create_core, create_collection两个命令的集合，既可以创建solr core，也可以创建solr collection。

查看命令的帮助信息

start, stop, restart, status, healthcheck, create, create_core, create_collection, delete, version, zk, auth, assert, config

```shell
./solr start -help
./solr stop -help
```

### Solr Core

内核：是运行在Solr服务器中的具体唯一命名的、可管理和可配置的索引。一台solr服务器可以托管一个或多个内核。

内核就是索引，为什么需要多个内核？
不同的文档拥有不同的模式（字段构成、索引、存储方式），如商品数据和新闻数据就有不同的字段构成以及不同的字段索引、存储方式。就需要分别用两个内核来索引、存储它们。

内核的典型用途：区分不同模式的文档

#### 令行命令创建内核

命令：`solr create_core [-c name] [-d confdir] [-p port] [-V]`
-d 选项可选值有两个： 
        _default      默认值，最少配置；
        sample_techproducts_cnofigs     示例的配置

创建默认配置的内核
`solr.cmd  create_core –c  mycore`

根据sample_techproducts_configs示例配置创建内核
`solr.cmd  create_core  -c  techproducts –d sample_techproducts_configs`

#### 删除内核

`solr delete -c name -p 8986 -deleteConfig `

#### 卸载、重载、加载内核

重载：配置文件修改之后，就需要重载

#### post命令

数据提交给Solr服务器中的核心

Linux/Mac
      solr-7.3.0: bin/post -c techproducts example/exampledocs/* 
Windows
      solr-7.3.0> java -jar -Dc=techproducts -Dauto example\exampledocs\post.jar example\exampledocs\* 

## SolrCloud分布式集群

索引数据量大，存储、性能怎么保证
保证高可靠、高可用
应对高并发、实时响应

索引存储：分割成多个片存储到集群的不同节点上，每个分片有备份，存储在集群的不同节点上。
		   solrCloud中以 collection（集合）来称呼索引，内核存储的是集合分片（shard）的备份（replication）

### 启动

#### zookeeper

独立的zookeeper，则需先启动zookeeper
内嵌的zookeeper，则先启动包含zookeeper的solrNode

#### solrNode

内嵌的zookeeper的第一个solrNode节点服务启动:
solr start -c -p 8983 -s D:/myspace/solr/solr-7.3.0/solrCloud/node1
其他solr节点的启动:
solr start -c -p 8984 -s D:/myspace/solr/solr-7.3.0/solrCloud/node2 –z localhost:9983

#### 创建集合 collection

集合—分片数2---备份因子2

solr create -c collection01 -d _default –shards 2 -replicationFactor 2 -p 8983
solr create -c collection02 -d sample_techproducts_configs -shards 2 -replicationFactor 2 -p 8983

#### 查看集群状态

./solr status

```
{
  "solr_home":"/home/bigdata/solr-7.3.0/server/solr",
  "version":"7.3.0 98a6b3d642928b1ac9076c6c5a369472581f7633 - woody - 2018-03-28 14:37:45",
  "startTime":"2019-04-05T02:42:38.033Z",
  "uptime":"0 days, 0 hours, 28 minutes, 19 seconds",
  "memory":"73.8 MB (%15) of 490.7 MB",
  "cloud":{
    "ZooKeeper":"H11:2181,H12:2181,H13:2181",
    "liveNodes":"3",
    "collections":"1"
   }
}
```

#### Linux集群部署

1、安装并启动zookeeper集群
2、解压solr安装包并启动，命令
	`./solr start -c -p 8983 -z H11:2181,H12:2181,H13:2181`
3、创建集合
	`solr create -c techproducts -d sample_techproducts_configs -shards 2 -replicationFactor 2 -p 8983`
4、导入样例数据
	`bin/post -c techproducts example/exampledocs/*`
调整linux系统的部分限制

```
vim /etc/security/limits.conf
* soft nofile 65000
* hard nofile 65000
* soft nproc 65000
* hard nproc 65000
```

#### post命令

数据提交给Solr服务器中的核心

Linux/Mac
      solr-7.3.0: bin/post -c techproducts example/exampledocs/* 
Windows
      solr-7.3.0> java -jar -Dc=techproducts -Dauto example\exampledocs\post.jar example\exampledocs\* 