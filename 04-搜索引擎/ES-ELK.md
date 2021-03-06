## ELK简介

Elasticsearch    Logstash   Kibana 原来称为 ELK Stack ，现在称为Elastic Stack，加入了 beats 来优化Logstash。

**ELK的主要用途**

大型分布式系统的日志集中分析。

一般我们需要进行日志分析场景：直接在日志文件中 grep、awk 就可以获得自己想要的信息。但在规模较大的场景中，此方法效率低下，面临问题包括日志量太大如何归档、文本搜索太慢怎么办、如何多维度查询。需要集中化的日志管理，所有服务器上的日志收集汇总。常见解决思路是建立集中式日志收集系统，将所有节点上的日志统一收集，管理，访问。

一般大型系统是一个分布式部署的架构，不同的服务模块部署在不同的服务器上，问题出现时，大部分情况需要根据问题暴露的关键信息，定位到具体的服务器和服务模块，构建一套集中式日志系统，可以提高定位问题的效率。

**集中式日志系统特点**

1、收集－能够采集多种来源的日志数据
2、传输－能够稳定的把日志数据传输到中央系统
3、转换— 能够对收集的日志数据进行转换处理
4、存储－如何存储日志数据
5、分析－可以支持 UI 分析
6、告警－能够提供错误报告，监控机制

ELK提供了一整套解决方案，并且都是开源软件，之间互相配合使用，完美衔接，高效的满足了很多场合的应用。
目前主流的一种日志系统。

### Beats

轻量型数据采集器。负责从目标源上采集数据。
官网介绍：https://www.elastic.co/cn/products/beats

#### FileBeat

Prospector 勘测者：
	负责管理Harvester并找到所有读取源。
	6.3开始叫 input 了。

Harvestor 收割机
	负责读取单个文件内容，发送到输出。

