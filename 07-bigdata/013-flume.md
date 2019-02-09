flume 水槽

flume OG(original Generation) 第一代
flume NG(Next New Generation) 第二代

```properties
agent名称,source,channel,sink的名称 event

a1.sources = r1
a1.channels = c1
a1.sinks = k1


#定义具体的source的内容
#这个source具体是什么类型，读取什么样的数据
a1.sources.r1.type = spooldir
#这个类型是spooldir,只要这个目录中新添加文件
a1.sources.r1.spoolDir = /root/flume-1.8.0/bigdata/log
#这个source就会读取数据

#定义具体的channel信息
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity =100

#定义具体的sink信息
#数据写入何处
a1.sinks.k1.type = hdfs
#路径
a1.sinks.k1.hdfs.path = hdfs://H01:9000/flume/event
#文件的前缀
a1.sinks.k1.hdfs.filePrefix = events-
#文件的格式，默认是二进制的
a1.sinks.k1.hdfs.fileType = DataStream

#不按照条数生成文件
a1.sinks.k1.hdfs.rollCount = 0
#HDFS上的文件达到128M生成一个文件
a1.sinks.k1.hdfs.rollSize = 134217728
#HDFS的文件达到60秒生成一个文件
a1.sinks.k1.hdfs.rollInterval = 60

#定义拦截器
a1.sources.r1.interceptors =

#最后组装我们之前定义的channel和sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

解压

/etc/profile中配置FLUME_HOME

修改flume-env.sh中的java_home

编写配置文件

启动 flume-ng agent -n a1 -c conf -f conf/a1.properties -Dflume.root.logger=INFO,console

`bin/flume-ng agent -n $agent_name -c conf -f conf/flume-conf.properties`

