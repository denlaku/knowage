```shell
hadoop jar **.jar wordcount /input /output
```

  

MapReduce特点

易于编程
良好的扩展性
高容错性
适合PB级以上海量数据的离线处理

MapReduce的特色—不擅长的方面

实时计算
​    像MySQL一样，在毫秒级或者秒级内返回结果
流式计算
​    MapReduce的输入数据集是静态的，不能动态变化
​    MapReduce自身的设计特点决定了数据源必须是静态的
DAG计算
​    多个应用程序存在依赖关系，后一个应用程序的输入为前一个的输出

map -> combiner -> partition -> shuffle->reduce



InputSplit

mapred.InputFormat

TextInputFormat

Partitioner

MutipleOutputs

MapReduce Join

Hdfs HA

