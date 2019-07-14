#### sort

可以对list，set，zset进行排序

```shell
sort list [desc|asc]# 适用于集合里的元素都是数字
sort list alpha [desc|asc] # 适用于集合中有非数字
soze zset by score [desc|asc]
```

#### by参数和get参数

```shell
hmset post:1 name aa age 12
hmset post:2 name bb age 10

lpush tags 1 2

sort tags by post:*->age desc
sort tags by post:*->age asc get post:*->name
```

#### 查看键的编码

```shell
set k abc
object encoding k
```

#### 快照RDB

1、根据规则自动快照

```shell
save 900 1
save 300 10
save 60 10000
```

2、手动执行save或bgsave
save会阻塞所有来自客户端的请求。
推荐使用bgsave，在后台异步的执行，快照的同时还能继续响应客户端请求

3、执行flushall命令
会清除所有的数据。只要存在自动快照条件，就一定会触发一次快照。

4、执行复制
主从模式，redis会在复制初始化是进行自动快照。

#### 快照原理

1、redis使用fork函数复制一份当前进程（父进程）的副本（子进程）；

2、父进程继续接受并处理客户端发来的命令，而子进程开始将内存中的数据写入硬盘中的临时文件；

3、当子进程写完所有数据后会用临时文件替换就的RDB文件



#### AOF

AOF（append only file）默认没有开启，需要将appendonly设置为yes才能启用AOF。开启AOF之久化，会根据一定的策略将所有修改数据的命令记录到aof文件中

aof文件重写

```shell
auto-aof-rewrite-percentage 100 # 当前AOF文件大小超过上一次重写时AOF文件大小的百分比到达多少是才会重写
auto-aof-rewrite-min-size 64mb # 允许重写的最小AOF文件大小
```



### redis集群

#### 主从复制

主数据库（mase），可以进行读写操作，当写操作导致数据变化时会自动同步给从数据库；

从数据库（slave），只读，接受从主数据库同步来的数据。

一个主数据库可以有多个从数据库，从数据库只能有一个主数据库。

主从配置

1、从数据库配置文件加入：slaveof 主数据库IP 主数据库port，主数据库不用进行任何配置。

2、启动时命令行中配置

```shell
# 启动主数据库
redis-server
# 启动两个从数据库
redis-server --port 6380 --slaveof 127.0.0.1 6379
redis-server --port 6381 --slaveof 127.0.0.1 6379
```

3、用slaveof命令将已经启动的实例设置成从数据库

```shell
slaveof 127.0.0.1 6379
```

**原理**

1、复制初始化。从数据库启动之后向主数据库发送sync命令，主数据库收到sync命令之后，后台会进行快照操作，同时将快照期间接受到的命令缓存起来；当快照完成后，会将快照文件和缓存的命令发送给从数据库。

从数据库收到后，会载入快照文件并执行收到的缓存命令

2、复制初始化结束后，主数据库每当收到写命令时，会将命令同步给从数据库。

redis采用的是乐观复制，容忍在一定时间内主从数据库数据不一致，但是会中通同步

### 哨兵

哨兵的作用就是监控redis系统的运行状况

1）、监控主数据库和从数据库是否正常运行

2）、主数据库出现故障时自动将从数据库转换为主数据库

#### 哨兵配置

1、配置好主从数据库

2、启动sentinel进程

```shell
# 首先创建一个配置文件sentinel.conf，内容为：
sentinel monitor mymaster 127.0.0.1 6379
# 启动
redis-sentinel sentinel.conf
```

配置哨兵监控时，只需配置监控主数据库即可，哨兵会自动发现所有的从数据库。

+sdown 表示哨兵主观认为主数据库停止服务了

+0down 表示哨兵客观认为主数据库停止服务了

此时哨兵将开始故障恢复，挑选一个从数据升格为主数据库