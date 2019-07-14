## hive的高级操作

### 一、复杂数据类型

#### 1、array

#### 2、map

#### 3、struct

#### 4、uniontype

### 二、视图

#### 1、Hive 的视图和关系型数据库的视图区别

和关系型数据库一样，Hive 也提供了视图的功能，不过请注意，Hive 的视图和关系型数据库的数据还是有很大的区别：

（1）只有逻辑视图，没有物化视图；

（2）视图只能查询，不能 Load/Insert/Update/Delete 数据；

（3）视图在创建时候，只是保存了一份元数据，当查询视图的时候，才开始执行视图对应的 那些子查询

#### 2、Hive视图的创建语句

```sql
create view view_cdt as select * from cdt;
```

#### 3、Hive视图的查看语句

```
show views;
desc view_cdt; -- 查看某个具体视图的信息
```

#### 4、Hive视图的删除语句

```sql
drop view view_cdt;
```

### 三、函数

#### 1、内置函数

##### 1）查看内置函数

```
show functions;
```

##### 2）显示函数的详细信息

```sql
desc function substr;
```

##### 3）显示函数的扩展信息

```sql
desc function extended substr;
```

#### 2、自定义函数UDF

当 Hive 提供的内置函数无法满足业务处理需要时，此时就可以考虑使用用户自定义函数。

**UDF**（user-defined function）作用于单个数据行，产生一个数据行作为输出。（数学函数，字 符串函数）

**UDAF**（用户定义聚集函数 User- Defined Aggregation Funcation）：接收多个输入数据行，并产 生一个输出数据行。（count，max）

**UDTF**（表格生成函数 User-Defined Table Functions）：接收一行输入，输出多行（explode）

### 四、特殊分隔符处理

补充：hive 读取数据的机制：

1、 首先用 InputFormat<默认是：org.apache.hadoop.mapred.TextInputFormat >的一个具体实 现类读入文件数据，返回一条一条的记录（可以是行，或者是你逻辑中的“行”）

2、 然后利用 SerDe<默认：org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe>的一个具体 实现类，对上面返回的一条一条的记录进行字段切割

Hive 对文件中字段的分隔符默认情况下只支持单字节分隔符，如果数据文件中的分隔符是多 字符的，如下所示：

01||huangbo

02||xuzheng

03||wangbaoqiang

#### 1、使用RegexSerDe正则表达式解析

SerDe (Serializer and Deserializer)

创建表

```sql
create table t_bi_reg(id string,name string)
row format serde 'org.apache.hadoop.hive.serde2.RegexSerDe'
with serdeproperties('input.regex'='(.*)\\|\\|(.*)','output.format.string'='%1$s %2$s')
stored as textfile;
```

导入数据并查询

```
load data local inpath '/home/hadoop/data.txt' into table t_bi_reg;
```

#### 2、通过自定义InputFormat处理特殊分隔符