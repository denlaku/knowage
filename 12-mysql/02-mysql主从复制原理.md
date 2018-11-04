**MySQL主从原理**

1. 每个从仅可以设置一个主。
2. 主在执行 SQL 之后，记录二进制 LOG 文件(bin-log)。
3. 从连接主，并从主获取 binlog，存于本地 relay-log，并从上次记住的位置起执行 SQL，一旦遇到错误则停止同步。

从库生成两个线程，一个I/O线程，一个SQL线程；I/O线程去请求主库 的binlog，并将得到的binlog日志写到relay log（中继日志） 文件中；主库会生成一个 log dump 线程，用来给从库 i/o线程传binlog；

**Replication原理推论**

1. 主从间的数据库不是实时同步，就算网络连接正常，也存在瞬间，主从数据不一致。
2. 如果主从的网络断开，从会在网络正常后，批量同步。
3. 如果对从进行修改数据，很可能从在执行主的bin-log出错而停止同步，一般不会修改从的数据。
4. 一个衍生的配置是双主，互为主从配置，只要双方的修改不冲突，可以工作良好。
5. 如果需要多主的话，可以用环形配置，这样任意一个节点的修改都可以同步到所有节点。
6. 可以应用在读写分离的场景中，用以降低单台 MySQL 服务器的 I/O。
7. 可以实现 MySQL 服务的 HA 集群。
8. 可以是一主多从，也可以是相互主从(主主)。

**问题及解决方法**

mysql主从复制存在的问题：

- 主库宕机后，数据可能丢失
- 从库只有一个sql Thread，主库写压力大，复制很可能延时

 解决方法：

- 半同步复制---解决数据丢失的问题
- 并行复制----解决从库复制延迟的问题

**半同步复制**

mysql semi-sync（半同步复制）

半同步复制：

- 5.5集成到mysql，以插件的形式存在，需要单独安装
- 确保事务提交后binlog至少传输到一个从库
- 不保证从库应用完这个事务的binlog
- 性能有一定的降低，响应时间会更长
- 网络异常或从库宕机，**卡主主库，直到超时或从库恢复**

**并行复制**

mysql并行复制

- 社区版5.6中新增
- 并行是指从库多线程apply binlog
- 库级别并行应用binlog，同一个库数据更改还是串行的(5.7版并行复制基于事务组)

设置 `set global slave_parallel_workers=10;`
设置sql线程数为10

**其他**

部分数据复制

主库添加参数：

```
binlog_do_db=db1
binlog_ignore_db=db1
binlog_ignore_db=db2
```

或从库添加参数

```
replicate_do_db=db1
replicate_ignore_db=db1
replicate_do_table=db1.t1
replicate_wild_do_table=db%.%
replicate_wild_ignore_table=db1.%
```

 联级复制（常用）

> A->B->C

B中添加参数：

> log_slave_updates
> B将把A的binlog记录到自己的binlog日志中

 

复制的监控：`show  slave status`

