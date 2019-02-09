## HDFS-各系统角色

### NameNode

#### NameNode的职责

1、负责客户端请求（读写数据  请求 ）的响应 
2、维护目录树结构（ 元数据的管理： 查询，修改 ）
3、配置和应用副本存放策略
4、管理集群数据块负载均衡问题

#### NameNode元数据的管理

预写式日志（Write-ahead logging，缩写 WAL）是关系数据库系统中用于提供原子性和持久性（ACID 属性中的两个）的一系列技术。在使用 WAL 的系统中，所有的修改在提交之前都要先写入 log 文件中，log 文件中通常包括 redo 和 undo 信息。

NameNode对数据的管理采用了两种存储形式：**内存和磁盘**

首先是**内存**中存储了一份完整的元数据，包括目录树结构，以及文件和数据块和副本存储地的映射关系；
1、内存元数据 metadata（全部存在内存中），其次是在磁盘中也存储了一份完整的元数据。
2、磁盘元数据镜像文件 **fsimage**
3、数据历史操作日志文件 edits（可通过日志运算出元数据，全部存在磁盘中）
4、数据预写操作日志文件 edits_inprogress（存储在磁盘中）

#### NameNode 元数据存储机制

A、内存中有一份完整的元数据(内存 metadata)

B、磁盘有一个“准完整”的元数据镜像（fsimage）文件(在 namenode 的工作目录中)

C、用于衔接内存 metadata 和持久化元数据镜像 fsimage 之间的操作日志（edits 文件）

（PS：当客户端对 hdfs 中的文件进行新增或者修改操作，操作记录首先被记入 edits 日志 文件中，当客户端操作成功后，相应的元数据会更新到内存 metadata 中）

### DataNode

#### Datanode 掉线判断时限参数

datanode 进程死亡或者网络故障造成 datanode 无法与 namenode 通信，namenode 不会立即 把该节点判定为死亡，要经过一段时间，这段时间暂称作超时时长。HDFS 默认的超时时长 为 10 分钟+30 秒。
如果定义超时时间为 timeout，则超时时长的计算公式为： 
timeout = 2 * heartbeat.recheck.interval + 10 * dfs.heartbeat.interval

而默认的 **heartbeat.recheck.interval** 大小为 5 分钟，**dfs.heartbeat.interval** 默认为 3 秒。 需要注意的是 hdfs-site.xml 配置文件中的 heartbeat.recheck.interval 的单位为毫秒， dfs.heartbeat.interval 的单位为秒。 所以，举个例子，如果 heartbeat.recheck.interval 设置为 5000（毫秒），dfs.heartbeat.interval 设置为 3（秒，默认），则总的超时时间为 40 秒。

### SecondaryNameNode

SecondaryNamenode 的作用就是分担 namenode 的合并元数据的压力。所以在配置 SecondaryNamenode 的工作节点时，一定切记，不要和 namenode 处于同一节点。但事实上， 只有在普通的伪分布式集群和分布式集群中才有SecondaryNamenode 这个角色，在 HA 或 者联邦集群中都不再出现该角色。在 HA 和联邦集群中，都是有 standby namenode 承担。

#### 元数据的 CheckPoint

每隔一段时间，会由 secondary namenode 将 namenode 上积累的所有 edits 和一个最新的 fsimage 下载到本地，并加载到内存进行 merge，这个过程称为 checkpoint。

CheckPoint 触发配置

```shell
dfs.namenode.checkpoint.check.period=60 ##检查触发条件是否满足的频率，60 秒
dfs.namenode.checkpoint.dir=file://${hadoop.tmp.dir}/dfs/namesecondary
##以上两个参数做 checkpoint 操作时，secondary namenode 的本地工作目录
dfs.namenode.checkpoint.edits.dir=${dfs.namenode.checkpoint.dir}
dfs.namenode.checkpoint.max-retries=3 ##最大重试次数
dfs.namenode.checkpoint.period=3600 ##两次 checkpoint 之间的时间间隔 3600 秒
dfs.namenode.checkpoint.txns=1000000 ##两次 checkpoint 之间最大的操作记录
```

#### CheckPoint 附带作用

Namenode 和 SecondaryNamenode 的工作目录存储结构完全相同，所以，当 Namenode 故障 退出需要重新恢复时，可以从SecondaryNamenode的工作目录中将fsimage拷贝到Namenode 的工作目录，以恢复 namenode 的元数据

















