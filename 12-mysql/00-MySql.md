# MYSQL

## mysql安装

### 单机安装

**安装前准备**

CentOS7环境
`10.10.10.21`
安装包：
` mysql-5.7.23-el7-x86_64.tar.g`

**上传安装包至目录`/usr/local/`并解压**

```shell
cd /usr/local/
tar -xvf mysql-5.7.23-el7-x86_64.tar.g
mv mysql-5.7.23-el7-x86_64 mysql
```

**创建mysql用户组及用户，并授权**

```shell
groupadd mysql
useradd -g mysql mysql
chown -R mysql /usr/local/mysql
chgrp -R mysql /usr/local/mysql
```

**创建配置文件`my.cnf`并添加如下配置**

```shell
[client]
port=3306
socket=/tmp/mysql.sock

[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
log-error=/var/log/mysqld.log
#pid-file=/var/run/mysqld/mysqld.pid
#不区分大小写
lower_case_table_names = 1
```

**初始化数据库**

```shell
#手动编辑一下日志文件，什么也不用写，直接保存退出
cd /var/log/
vim mysqld.log

chmod 777 mysqld.log
chown mysql:mysql mysqld.log

/usr/local/mysql/bin/mysqld --initialize --user=mysql \
--basedir=/usr/local/mysql --datadir=/usr/local/mysql/data \
--lc_messages_dir=/usr/local/mysql/share --lc_messages=en_US
```

**查看初始密码**

```shell
cat /var/log/mysqld.log
#密码在文件的最后一行
#[Note] A temporary password is generated for root@localhost: I3U#HIqPu7yv
#这安装后的初始密码是：I3U#HIqPu7yv

# 或者用如下命令查看密码
grep "password" /var/log/mysqld.log
```

**启动mysql服务，修改初始密码**

```shell
/usr/local/mysql/support-files/mysql.server start
/usr/local/mysql/bin/mysql -uroot -p
#输入密码 I3U#HIqPu7yv
set password=password('new_pwd');
grant all privileges on *.* to 'root'@'%' identified by 'new_pwd';
flush privileges;
```

**设置开机自启**

```shell
cd /usr/local/mysql/support-files
cp mysql.server /etc/init.d/mysqld
chkconfig --add mysqld
```

ln -s /usr/local/mysql/bin/mysql /usr/bin

至此单机安装已经完成。

------

### 主从复制安装

主库IP：`10.10.10.21`
从库IP：`10.10.10.22`

主库创建数据同步用户并授权

```sql
create user repli identified by 'new_password';
grant replication slave on *.* to 'repli'@'%' identified by 'new_password'; 
```

编辑主库my.cnf文件，添加如下内容

```shell
[mysqld]
log-bin=mysql-bin # 开启bin日志
server-id=21 #mysql服务ID(必须唯一)
```

编辑从库my.cnf文件，添加如下内容

```shell
[mysqld]
server-id=22 #mysql服务ID(必须唯一)
```

获取主库binlog文件名称及pos位置信息，执行 `show master status;`

| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
| ---------------- | -------- | ------------ | ---------------- | ----------------- |
| mysql-bin.000002 | 696      |              |                  |                   |

在从节点配置访问主节点的参数信息

```shell
change master to 
master_host='10.10.10.21',
master_user='repli',
master_password='denlaku', 
master_log_file='mysql-bin.000002',
master_log_pos=696;
```

启动从库复制，执行 `start slave;`
停止从库复制，执行  `stop slave;`

### 主从复制原理

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

### MySql文件

#### **参数文件my.cnf**

在linux中mysql的配置文件位于： /etc/my.cnf

#### 错误日志文件（error log）

错误日志log-err 默认是关闭的，记录严重的警告和错误信息
查看命令：`show variables like 'log_error%;`

#### 二进制日志文件（binary log）

记录对mysql数据库做真正更改的所有操作，不包含哪些没有做任何修改的语句，也不好喊select、show语句。

主要作用： 用于主从复制；进行恢复操作。
查看命令：`show variables like 'log_bin%;`

#### 慢查询日志（slow log）

把超过参数`long_query_time`时间的所有sql记录下来。long_query_time默认是10秒。
查看命令：`show variables like 'slow%';`

#### 全量日志（general log）

记录所有操作语句，包括select、show。该动能默认关闭。

#### 审计日志（audit log）

#### 中级日志（relay log）

#### pid文件

#### Socket文件

#### 表结构文件

fmt文件，存储表结构

#### **数据文件**

myd文件，存储表数据
myi文件，存储索引数据

### root用户密码丢失问题

首先，杀掉已经启动的mysql进程；
然后，加跳过权限表参数，重启数据库。如下：

```shell
./msqld_safe --defaults-file=/etc/my.conf --skip-grant-tables
```

之后，修改root用户密码并刷新权限；
最后，重启数据库。

## MySql体系结构

mysql体系结构分为两层，即**server层**和**存储引擎层**。
mysql server层又包含连接层和sql层。连接层又包含通信协议、线程处理、用户名密码认证。
**sql层**包含权限判断、查询缓存、解析器、预处理、查询优化器、缓存和执行计划。
**权限判断**可以审核用户有没有访问某个数据库、某张表或者表里行的权限；
**查询缓存**通过QueryCache进行操作，如果数据在QueryCache中，则直接返回结果给客户端；
**查询解析**器针对sql进行解析，判断语法是否正确；
**预处理器**对解析器无法解析的语义进行处理；
**优化器**对sql进行改写和相应的优化，并生成最优的执行计划；

## MySql存储引擎

### 主要存储引擎

| 存储引擎 | 特点                                                 | 引用场景      |
| -------- | ---------------------------------------------------- | ------------- |
| InnoDB   | 支持事务、行锁；支持并发控制，并发性高               | 应用于OTB系统 |
| MyISAM   | 不支持事务、行锁；支持表锁；并发低；资源利用率也低； |               |
| Memory   | 表中的数据存放在内存中，读取数据库，安全线不高       |               |

### InnoDB与MyISAM的对比

| 区别            | InnoDB   | MyISAM         |
| --------------- | -------- | -------------- |
| 事务            | 支持     | 不支持         |
| 锁粒度          | 行锁     | 表锁           |
| 并发性          | 高       | 低             |
| select count(*) | 全表扫描 | 冲计数器中读取 |





## MySql索引

### **什么是索引？**

索引本质上是一种排好序查找快的数据结构，有助于mysql快速高效返回符合条件的数据，提高数据查找效率。

索引的优势和劣势：
**优势**：提高了数据的检索效率，降低了数据库的IO成本；通过对索引列的排序，降低数据的排序成本，降低了CPU的消耗。

**劣势**：索引实际上也是一种表，保存了主键和索引字段，并指向实体表的记录，也是需要占用磁盘空间的。同时索引也会降低更新表的速度，对insert、update、delete都有影响。因为索引字段的数据发生变化，也需要对响应的索引进行根性。索引只是提高查询速度的一个因素，对于数据量大的表，还是需要建立最优秀的索引，并且还要优化查询语句。

### **索引分类**

**单只索引**：一个索引只包含一个列，一个表可以有多个单只索引。
**唯一索引**：索引列的值必须唯一，但允许有空值。
**复合索引**：一个索引包含多个列。

### 索引语法

**创建索引**

```sql
create [unique] index indexName on tableName(column1, column2);
alter tableName add index [indexName] on(column1, column2);
```

**删除索引**

```sql
drop index [indexName] on tableName;
```

**查看索引**

```sql
show index from tableName;
```

### 需要创建索引情况

主键自动建立唯一索引；
频繁作为查询条件的字段应该创建索引；
查询中与其他表关联的字段，外键关系建立索引；
查询中排序的字段，排序字段若通过索引去访问，将大大提高排序速度；
查询中统计或分组字段；

### 不需要创建索引情况

表记录太少；
经常增删改的表；
数据重复且分布平均的字段，因为包含很多重复内容，建索引没有太大实际效果；
频繁跟新的字段不宜创建索引，因为每次跟新不仅更新了记录，还会更新索引，加重了IO负担；
where中用不到的字段不创建索引；

### 索引分类

BTree索引

Hash索引

full-text索引

R-Tree索引

## Explain

**id**

id相同，执行顺序由上而下
id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行。
id相同、不同同时存在

**select_type**

SIMPLE：简单的select查询，不包含子查询或union
PRIMARY： 查询中最外层的select(如两表做union或者存在子查询的外层的表操作为PRIMARY,内层的操作为UNION)
SUBQUERY：select或where列表中的子查询
DERIVED：在from中包含的子查询
UNION：
UNION RESULT：union获得的结果集

**type**

all：全表扫描
index：全索引扫描
range：只检索给定的行，使用一个索引来选择行。一般是where中用了between, >, <, in等查询。返回扫描索引。
ref ：非唯一索引扫描，返回匹配某个单独值的所有行。
eq_ref： 唯一索引扫描，对于每一个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描。
const：表示通过索引一次就找到了，const用于比较primary key和unique索引。返回结果只有一行数据。
system：表只有一行记录(等于系统表)，是const类型的特例。

**possible_keys**

显示可能应用到这张表的索引，一个或多个。
查询上涉及到的字段上若存在索引，则该索引被列出，但不一定被查询实际使用

**key**

实际中使用的索引。若果为null，则没有使用索引。
若果查询中使用了覆盖索引，则索引只出现在key列表中。
**覆盖索引**：查询的字段和是建立索引的字段，也就是说select的字段从索引中就能获取，不用读取数据行；

**key_len**

表示索引中使用的字节数，可以通过该列结算查询中使用的索引长度。在不损失精确的情况下，长度越短越好。
key_len显示的值为索引字段的最大可能长度，并非实际使用长度。ken_len根据表定义计算而得，不是通过表内检索出的。

**ref**

显示了索引的哪一列被使用了，如果可能的话是一个常数。哪些列或常量被用于查找索引列上的值。

**rows**

根据表统计信息及索引选用情况，大致估算找出所需记录所要读取的行数。

**Extra**

Using filesort：对数据使用一个外部的索引排序，而不是按照表内的索引进行排序；无法利用索引完成排序的操作称“**文件排序**”。

Using temporary：使用了临时表保存中间结果，对查询结果排序时使用了临时表。常见于order by和group by。

Using index：使用了**覆盖索引**，避免访问表中的数据行。如果同时出现了Using where表明索引被用来执行索引键值的查找。

Using Where：

Using join buffer：

Impossible where：

Select tables optimized away：

distinct：

## MySql优化

### 查询优化

### 慢查询日志

### show profile

```reStructuredText

```

## 锁

### 锁的分类

从对数据操作的类型来分：

​	读锁（共享锁）：针对同一份数据，多个读操作可以同时进行，而不会互相影响。
​	写锁（排它锁）：当写操作没有完成之前，会阻塞其他写锁和读锁。

从对数据操作的粒度来分：

​	行锁：
​	表锁：

### 表锁

show open tables;

lock tabe tableName read/write;

unlock tables;

### 行锁



select @@tx_isolation;

show variables like '%REPEATABLE-READ%';













































