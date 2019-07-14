#### linux上安装

1、下载安装包
2、解压

```shell
tar -xvf elasticsearch-6.2.4.tar.gz
```

3、配置
4、启动

```shell
./elasticsearch
# 后台运行
./elasticsearch -d
```

启动时指定参数：

```shell
./elasticsearch -Ecluster.name=my_cluster_name -Enode.name=my_node_name
```

了解启动脚本可用选项

```shell
./elasticsearch -h
```

linux 虚拟机上运行可能的失败问题
**问题一**：内存不够用，默认es配置使用1G堆内存，如果的你学习用的虚拟机没有这么大的内存，请在config/jvm.options中调整。

**问题二**：max file descriptors [4096] for elasticsearch process likely too low, increase to at least [65536]
解决：修改切换到root用户修改配置limits.conf 添加下面两行
命令:vim /etc/security/limits.conf

```shell
* hard nofile 65536
* soft nofile 65536
```

**问题三**：max number of threads [1024] for user [lish] likely too low, increase to at least [2048]
解决：切换到root用户，进入limits.d目录下修改配置文件。
vim /etc/security/limits.d/20-nproc.conf 
修改如下内容：
\* soft nproc 1024
修改为
\* soft nproc 2048

**关于soft nproc 与 soft nofile**

> soft nproc: 可打开的文件描述符的最大数(软限制)
>
> hard nproc： 可打开的文件描述符的最大数(硬限制)
>
> soft nofile：单个用户可用的最大进程数量(软限制)
>
> hard nofile：单个用户可用的最大进程数量(硬限制)

**问题四**：max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
解决：切换到root用户修改配置sysctl.conf
vim /etc/sysctl.conf 
添加下面配置：
vm.max_map_count=655360  # 定义了一个进程能拥有的最多的内存区域，默认为65536
并执行命令：
sysctl -p
切换到es的用户。
然后，重新启动elasticsearch，即可启动成功。

#### ES 配置说明

**配置文件分离**

配置文件目录默认为：$ES_HOME/config，如果需要配置文件与软件分离（方便升级），可以通过 ES_PATH_CONF 环境变量来指定。如你可以在命令行指定声明：

```shell
ES_PATH_CONF=/path/to/ny/config ./elasticsearch
```

JVM参数

```shell
-Xms1g
-Xmx1g
```

**数据目录和日志目录**
生成环境下应与软件分离

```yaml
# Path to directory where to store the data (separate multiple locations by comma):
path.data: /path/to/data
# Path to log files:
path.logs: /path/to/logs
```

**集群**

```yaml
# ---------------------------------- Cluster -----------------------------------
# Use a descriptive name for your cluster:
cluster.name: my-application
```

**节点**

```yaml
# ------------------------------------ Node ------------------------------------
# Use a descriptive name for the node:
node.name: node-1
# Add custom attributes to the node:
node.attr.rack: r1

```

**network**

默认绑定的是["127.0.0.1", "[::1]"]回环地址，集群下要服务间通信，需绑定一个ipv4或ipv6地址

```yaml
# ---------------------------------- Network -----------------------------------
# Set the bind address to a specific IP (IPv4 or IPv6):
network.host: 10.10.10.11
# Set a custom port for HTTP:
http.port: 9200
# For more information, consult the network module documentation.
```

**Discovery(节点发现配置)**

ES中默认采用的节点发现方式是  zen（基于组播（多播）、单播）。在应用于生产前有两个重要参数需配置：
**discovery.zen.ping.unicast.hosts**: ["host1","host2:port","host3[portX-portY]"]
单播模式下，设置具有master资格的节点列表，新加入的节点向这个列表中的节点发送请求来加入集群。
**discovery.zen.minimum_master_nodes**: 1
这个参数控制的是，一个节点需要看到具有master资格的节点的最小数量，然后才能在集群中做操作。官方的推荐值是(N/2)+1，其中N是具有master资格的节点的数量。

```yaml
# --------------------------------- Discovery ----------------------------------
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
discovery.zen.ping.unicast.hosts: ["host1", "host2"]
# Prevent the "split brain" by configuring the majority of nodes 
# (total number of master-eligible nodes / 2 + 1):
discovery.zen.minimum_master_nodes: 
# For more information, consult the zen discovery module documentation.
```

transport.tcp.compress: false
    是否压缩tcp传输的数据，默认false
http.cors.enabled: true
    是否使用http协议对外提供服务，默认true
http.max_content_length: 100mb
    http传输内容的最大容量，默认100mb
node.master: true
    指定该节点是否可以作为master节点，默认是true。
    ES集群默认是以第一个节点为master，如果该节点出故障就会重新选举master。
node.data: true
    该节点是否存索引数据，默认true。
discover.zen.ping.timeout: 3s
    设置集群中自动发现其他节点时ping连接超时时长，默认为3秒。
    在网络环境较差的情况下，增加这个值，会增加节点等待响应的时间，从一定程度上会减少误判。
discovery.zen.ping.multicast.enabled: false
    是否启用多播来发现节点

### 安装Kibana

Kibana是ES的可视化管理工具。

**下载安装包**
https://www.elastic.co/downloads/kibana

**安装**
解压到安装目录即可

**配置**
在config/kibana.yml中配置 elasticsearch.url的值为 ES的访问地址

```yaml
server.host: "10.10.10.11"
elasticsearch.url: "http://10.10.10.11:9200"
```

**启动**
./bin/kibana
访问地址：http://10.10.10.11:5601

**数据批量导入**

```shell
curl -H "Content-Type: application/json" -XPOST "10.10.10.11:9200/bank/_doc/_bulk?pretty&refresh" --data-binary "@accounts.json"
```



### IKAnalyzer集成

#### 获取 ES-IKAnalyzer插件

地址： https://github.com/medcl/elasticsearch-analysis-ik/releases

#### 安装插件

将 ik 的压缩包解压到 ES安装目录的plugins/目录下（最好把解出的目录名改一下，防止安装别的插件时同名冲突），然后重启ES。

#### 扩展词库

配置文件config/IKAnalyzer.cfg.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">custom/mydict.dic;custom/single_word_low_freq.dic</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords">custom/ext_stopword.dic</entry>
 	<!--用户可以在这里配置远程扩展字典 -->
	<entry key="remote_ext_dict">location</entry>
 	<!--用户可以在这里配置远程扩展停止词字典-->
	<entry key="remote_ext_stopwords">http://xxx.com/xxx.dic</entry>
</properties>
```

