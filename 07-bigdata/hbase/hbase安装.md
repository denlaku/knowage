### Hbase安装

这里使用hbase-1.3.1版本。

```shell
# 解压安装包
  tar xvf hbase-1.3.1-bin.tar.gz
# 编辑hbase-1.3.1/conf/hbase-env.sh
  cd hbase-1.3.1/conf
  vi hbase-env.sh
  # 修改java home
  export JAVA_HOME=/lib/java/jdk1.8.0_151
  # 修改日志路径
  export HBASE_LOG_DIR=/home/bigdata/hbase-1.3.1/logs
  # 默认启动zookeeper
  export HBASE_MANAGES_ZK=true
# 编辑hbase-1.3.1/conf/regionservers,把里边的localhost替换为：
  H11
  H12
  H13
# 配置HBASE的环境变量
# 编辑hbase-1.3.1/conf/hbase-site.xml
  <configuration>
    <property>
    	<name>hbase.rootdir</name>
    	<value>hdfs://H11:9000/hbase</value>
    </property>
    <property>
    	<name>hbase.zookeeper.property.dataDir</name>
    	<value>/home/bigdata/var/zookper-3.4.10/data</value>
    </property>
    <property> 
      <name>hbase.cluster.distributed</name>
      <value>true</value>
    </property>
    <property>
      <name>hbase.master</name>
      <value>H11:60000</value>
    </property>
    <property>
      <name>hbase.zookeeper.quorum</name>
      <value>H11,H12,H13</value>
    </property>
  </configuration>
# 复制hbase配置文件到H12、H13
  scp -r /home/hadoop/hbase-1.3.1 root@H22:/home/hadoop
  scp -r /home/hadoop/hbase-1.3.1 root@H23:/home/hadoop
# 启动hbase 
  ./start-hbase.sh # 因为配置了export HBASE_MANAGES_ZK=true，所以会自动启动hbase自带的zookeeper
  # 打开shell命令行
  ./hbase shell
  # url
  http://10.10.10.11:16010
  http://10.10.10.12:16030
  http://10.10.10.13:16030
```

### 