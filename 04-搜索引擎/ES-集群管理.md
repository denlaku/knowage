## 集群规划

### 集群规划

#### 集群规模

推算的依据：ES JVM heap 最大 32G  。
30G heap  大概能处理的数据量 10 T。如果内存很大如128G，可在一台机器上运行多个ES节点实例。

集群规划满足当前数据规模+适量增长规模即可，后续可按需扩展。

两类应用场景：
用于构建业务搜索功能模块，且多是垂直领域的搜索。数据量级几千万到数十亿级别。一般2-4台机器的规模。
用于大规模数据的实时OLAP（联机处理分析），经典的如ELK Stack，数据规模可能达到千亿或更多。几十到上百节点的规模。

#### 角色分配

**节点角色**
Master 
	node.master: true    节点可以作为主节点
DataNode
	node.data: true    默认是数据节点。      
Coordinate node   协调节点
	如果仅担任协调节点，将上两个配置设为false。

一个节点可以充当一个或多个角色，默认三个角色都有

**分配规则**

1、小规模集群，不需严格区分。
2、中大规模集群（十个以上节点），应考虑单独的角色充当。特别并发查询量大，查询的合并量大，可以增加独立的协调节点。3、角色分开的好处是分工分开，不互影响。如不会因协调角色负载过高而影响数据节点的能力。

#### 脑裂问题

尽量避免脑裂，配置：
**discovery.zen.minimum_master_nodes: (有master资格节点数/2) + 1**
这个参数控制的是，选举主节点时需要看到最少多少个具有master资格的活节点，才能进行选举。
官方的推荐值是(N/2)+1，其中N是具有master资格的节点的数量。

**常用做法**（中大规模集群）：

1、Master 和 dataNode 角色分开，配置奇数个master
2、单播发现机制，配置master资格节点
	discovery.zen.ping.multicast.enabled: false 
	discovery.zen.ping.unicast.hosts: ["master1", "master2", "master3"] 
3、配置选举发现数，及延长ping master的等待时长
	discovery.zen.ping_timeout: 30（默认值是3秒）
	discovery.zen.minimum_master_nodes: 2

#### 索引分片

##### 分片过多的影响

1、每个分片本质上就是一个Lucene索引, 因此会消耗相应的文件句柄, 内存和CPU资源。
2、每个搜索请求会调度到索引的每个分片中. 如果分片分散在不同的节点倒是问题不太. 
	但当分片开始竞争相同的硬件资源时, 性能便会逐步下降。
3、ES使用词频统计来计算相关性. 当然这些统计也会分配到各个分片上. 
	如果在大量分片上只维护了很少的数据, 则将导致最终的文档相关性较差。

##### 索引分片原则

1、ElasticSearch推荐的最大JVM堆空间是30~32G, 所以把你的分片最大容量限制为30GB, 然后再对分片数量做合理估算. 例如, 你认为你的数据能达到200GB, 推荐你最多分配7到8个分片。
2、在开始阶段, 一个好的方案是根据你的节点数量按照1.5~3倍的原则来创建分片. 例如,如果你有3个节点, 则推荐你创建的分片数最多不超过9(3x3)个。当性能下降时，增加节点，ES会平衡分片的放置。
3、对于基于日期的索引需求, 并且对索引数据的搜索场景非常少. 也许这些索引量将达到成百上千, 但每个索引的数据量只有1GB甚至更小. 对于这种类似场景, 建议只需要为索引分配1个分片

##### 分片副本

副本数可以随之扩展

**基本原则**
1、为保证高可用，副本数设置为2即可。要求集群至少要有3个节点，来分开存放主分片、副本。
2、如发现并发量大时，查询性能会下降，可增加副本数，来提升并发查询能力。

### x-pack

为集群提供安全防护、监控、告警、报告等功能的收费组件；
部分免费：https://www.elastic.co/subscriptions
6.3开始已开源，并并入了elasticsearch核心中。

x-pack的功能组件：
**Security**：为ES stack 提供安全防护。
**Alerting**：提供告警能力，让你实时获知集群的异常状态。
**Monitoring**：提供ES stack的监控能力，一目了然掌握集群的健康状况，及故障诊断。
**Reporting**：能够为任何 Kibana 可视化或仪表板快速生成报告。您可以即需即取报告、预约报告、根据特定条件触发报告，并自动将报告分享给指定的人员。
**Graph**：提供分析数据间关系的功能。
**Machine Learning**：提供机器学习能力。

#### elasticsearch安装x-pack

1、执行命令安装x-pack

```shell
bin/elasticsearch-plugin install x-pack
```

2、重启elasticsearch

2、设置用户密码

```shell
bin/x-pack/setup-passwords interactive
```

共有三个内置用户，分别是elastic、 kibana、logstash，其中elastic是超级用户，拥有最高权限。

#### kibana安装x-pack

1、执行命令安装x-pack

```shell
bin/kibana-plugin install x-pack
```

2、修改kibana的配置文件
	在配置文件kibana.yml中配置内建用户kibana的密码

```shell
elasticsearch.username: "kibana"
elasticsearch.password: "kibanapassword"
```

3、重启kibana

#### logstash上安装x-pack

在logstash上安装 x-pack是可选的。如果你想安装，流程如下。
如果不安装，为了让logstash能访问elasticsearch索引数据，只需在配置文件logstash.yml中配置访问es的用户密码。

1、安装 x-pack 

```shell
bin/logstash-plugin install x-pack
```

2、修改logstash的配置文件
在配置文件 logstash.yml中配置内建用户logstash_system的密码

```shell
xpack.monitoring.elasticsearch.username: logstash_system
xpack.monitoring.elasticsearch.password: logstashpassword
```

3、启动/重启logstash