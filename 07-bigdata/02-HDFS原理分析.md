HDFS源自于google的GFS论文

Hadoop Distributed File System

易于扩展的分布式文件系统
运行在大量普通廉价机器上，提供容错机制
为大量用户提供性能不错的文件存取服务

#### HDFS的优点：

+ 高容错性
  ​    数据自动保存多个副本
  ​    副本丢失后自动恢复
+ 适合批处理
  ​    移动计算而非数据
  ​    数据位置暴露给计算框架
+ 适合大数据处理
  ​    GB、TB、甚至PB级数据
  ​    百万规模以上的文件数量
  ​    10K+节点规模  
+ 流式文件访问
  ​    一次性写入，多次读取
  ​    保证数据一致性
+ 可构建在廉价机器上
  ​    通过多副本提高可靠性
  ​    提供了容错和恢复机制   

#### HDFS的缺点：

+ 低延迟数据访问
  ​    比如毫秒级
  ​    低延迟与高吞吐率
+ 小文件存取
  ​    占用NameNode大量内存
  ​    寻道时间超过读取时间
+ 并发写入、文件随机修改
  ​    一个文件只能有一个写者
  ​    仅支持append



对于文件，采用分块存储的方式，一个块也就是一个block，默认128MB，大小可调整。



**NameNode** 存储文件的元数据。

**SecondaryNameNode**

**DataNode**

**ResourceManager**

**NodeManager**



hsfs-site.xml

```xml
<property>  
	<name>dfs.block.size</name>  
	<value>5242880</value>  
</property>
```



HDFS写流程

HDFS读流程

HDFS副本放置策略

HDFS可靠性策略

+ 文件完整性
+ HeartBeat
+ 元数据信息

HDFS不适合存储小文件

SequenceFile

刀片服务器

每个机架有16~64台节点

复制因子

Edits/fsimage

check point

**Configuration**

**FileSystem**

**Path**

**PathFilter**

**FileStatus**

**FileUtil**

