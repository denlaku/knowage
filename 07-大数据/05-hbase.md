### Hbase安装

这里使用hbase-1.3.1版本。

```shell
# 解压安装包
  tar xvf hbase-1.3.1-bin.tar.gz
# 编辑hbase-1.3.1/conf/hbase-env.sh
  cd hbase-1.3.1/conf
  vi hbase-env.sh
  # 修改java home
  export JAVA_HOME=/lib/java/jdk1.8.0_144/
  # 修改日志路径
  export HBASE_LOG_DIR=/home/hadoop/tmp/hbase131/logs
  # 默认启动zookeeper
  export HBASE_MANAGES_ZK=true
# 编辑hbase-1.3.1/conf/regionservers,把里边的localhost替换为：
  H21
  H22
  H23
# 编辑hbase-1.3.1/conf/hbase-site.xml
  <configuration>
    <property>
    	<name>hbase.rootdir</name>
    	<value>hdfs://H21:9000/hbase</value>
    </property>
    <property>
    	<name>hbase.zookeeper.property.dataDir</name>
    	<value>/home/hadoop/tmp/zookper3410/data</value>
    </property>
    <property>
      <name>hbase.cluster.distributed</name>
      <value>true</value>
    </property>
    <property>
      <name>hbase.master</name>
      <value>H21:60000</value>
    </property>
    <property>
      <name>hbase.zookeeper.quorum</name>
      <value>H21,H22,H23</value>
    </property>
  </configuration>
# 复制hbase配置文件到H22、H23
  scp -r /home/hadoop/hbase-1.3.1 root@H22:/home/hadoop
  scp -r /home/hadoop/hbase-1.3.1 root@H23:/home/hadoop
# 启动hbase 
  ./start-hbase.sh # 因为配置了export HBASE_MANAGES_ZK=true，所以会自动启动zookeeper
  # 打开shell命令行
  /hbase shell
  # url
  http://10.10.10.21:16010
  http://10.10.10.22:16030
  http://10.10.10.23:16030
```

### Hbase常用命令

```shell
# version 返回hbase版本信息
# status 返回hbase集群的状态信息
# exit 退出hbase shell
# shutdown 关闭hbase集群（与exit不同）
# ------------------------------------------------------------------------------------
# help查看某个命令的帮助信息
  help 'get' # 获取get命令的帮助信息
# 显示所有的表
	list
	list 'user'
# create 创建表
	create 'user','name','age'
# truncate 重新创建指定表,表里的数据会消失
	truncate 'test'
# describe 显示表相关的详细信息
	describe 'test'
# disable 使表无效
	disable 'test'
# enable 使表有效
	enable 'test'
# drop 删除表（删除表之前必须使用disable使表失效）
	drop 'test'
# exists 测试表是否存在
	exists 'test'
# alter 修改列族（column family）模式
	alter 'test','delete'=>'age' # 删除指定的列族
	alter 'test', READONLY   # 设置表为只读。
# ------------------------------------------------------------------------------------
# 直接把表赋值个某个变量，如
  t=create 'user','name','age' # 创建表时赋值
  t=get_table 'user' # 获取表并赋值
# scan 查看表的所有数据
	scan 'test'
# count 统计表中行的数量
	count 'test
# put 向指向的表单元添加值
	put '表名','行名','列名','值'
	put 'test','row1','name','tom'
# get 查看记录
	get '表名','行名' # 获取一行
	get 'test','row1'
	get '表名','行名','列名' # 获取某一行中的一列
	get 'test','row1','name'
# delete 删除指定列的值
	delete 'test','row1','name'
# deleteall删除指定行的所有元素值
	deleteall 'users','row1'
# incr 增加指定表行或列的值 (HBase计数器)
	incr 'users','row1','name:info'

```

