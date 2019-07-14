## hive安装

### 1、解压

### 2、配置hive的环境变量

```shell
vi .bash_profile
export HIVE_HOME=/home/bigdata/hive-2.3.4
```

### 3、 配置hive-env.sh文件

`hive-env.sh`文件可以通过复制`hive-env.sh.template`得到

配置内容如下：

```shell
HADOOP_HOME=/home/bigdata/hadoop-2.8.5
export HIVE_CONF_DIR=/home/bigdata/hive-2.3.4/conf
export HIVE_AUX_JARS_PATH=/home/bigdata/hive-2.3.4/lib
```

### 5、配置hive-log4j2.properties文件

```shell
# 通过复制创建hive-log4j2.properties
cp hive-log4j2.properties.template hive-log4j2.properties
# 通过vi修改日志的存放文件
property.hive.log.dir=/home/bigdata/hive-2.3.4/logs
```

### 6、启动hadoop集群

hive是基于hadoop运行的，在启动hive之前必须启动hadoop

### 7、初始化元数据

查看schematool命令的一些选项：`./schematool --help`

这里可以采用derby数据库，也可以使用MySql数据库

#### derby数据库

derby数据库为hive的默认的元数据库，元数据初始化命令：

`./schematool -dbType derby -initSchema`

#### MySql数据库

使用mysql的话，需要创建hive-site.xml文件，添加我们自定义的信息

```xml
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://10.10.10.2:3306/hive?createDatabaseIfNotExist=true</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123</value>
    </property>
    <property>
        <name>hive.exec.local.scratchdir</name>
        <value>/home/bigdata/var/hive234/tmp/${user.name}</value>
        <description>Local scratch space for Hive jobs</description>
    </property>
    <property>
        <name>hive.downloaded.resources.dir</name>
        <value>/home/bigdata/var/hive234/tmp/${hive.session.id}_resources</value>
        <description>Temporary local directory for added resources in the remote file system.</description>
    </property>
    <property>
        <name>hive.querylog.location</name>
        <value>/home/bigdata/var/hive234/tmp/${user.name}</value>
        <description>Location of Hive run time structured log file</description>
    </property>
    <property>
        <name>hive.server2.logging.operation.log.location</name>
        <value>/home/bigdata/var/hive234/tmp/${user.name}/operation_logs</value>
        <description>Top level directory </description>
    </property>
</configuration>

```

在mysql中初始化元数据

`./schematool -dbType mysql -initSchema`

关于mysql作为元数据库的几点说明
​	1）hive当中创建的表的信息，在元数据库的TBLS表里面
​	2）这个表的字段信息，在元数据库的COLUMNS_V2表里面
​	3）这个表在HDFS上面的位置信息，在元数据库的SDS表里面

## hive使用

### 1：创建表

```sql
-- 创建表
create table teacher (
    id int, name string
) 
row format delimited fields terminated by ','; -- 指定分隔符
-------------------------------------------------------------------------------------------------
-- 创建外部表
create external table person(
    id int,
    name string,
    hobby array<string>,
    addr map<string,string>
)
row format delimited 
fields terminated by ',' 
collection items terminated by '-' 
map keys terminated by ':' 
location '/hive/tmp/person'；

/*
内部表(MANAGED TABLE): 未被external修改的
外部表(EXTERNAL TABLE): 被external修饰的
内部表和外部表区别
  1）内部表数据由hive自身管理，外部表数据由hdfs来管理
    内部表数据存储的位置默认/user/hive/warehouse,
    外部表数据存储的位置由用户自己指定
  2）删除内部表会直接删除元数据和存储数据
    删除外部表仅仅只会删除元数据，HDFS上的文件不会删除。
*/
-------------------------------------------------------------------------------------------------
-- 分区表
create table person_pt(
	id int,
	name string,
	hobby array<string>,
	addr map<string,string>
)
partitioned by (pt string) -- 指定分区字段 
-- partitioned by (p_dt string, pk String) 多分区
row format delimited 
fields terminated by ',' 
collection items terminated by '-' 
map keys terminated by ':';
-- 注意：分区字段不能和表中的字段重复，若要创建分区表，必须在表定义的时候创建partition
```

### 2、加载文件中的数据至hive表中

```shell
# 从本地导入文件文件
load data local inpath '/home/bigdata/teacher.txt' into table teacher;
load data local inpath '/home/bigdata/person.txt' into table person;
load data local inpath '/home/bigdata/person.txt' into table person_pt partition(pt='201808');
# 从hdfs导入数据，不要local
load data local inpath '/home/bigdata/teacher.txt' into table teacher;
```

表分区

```shell
# 查看persons表的表分区
show partitions persons;
# 添加表分区
alter table persons add partition (p_dt=201809);
# 删除分区
alter table persons drop partition (p_dt=201809);
```



### 3、删除表

```sql
drop table tableName;
```

### 4、删除表数据

```sql
truncate table tableName;
```

```shell
show tables;
show databases;
show create table persons;
select * from persons where p_dt='201808';
```







