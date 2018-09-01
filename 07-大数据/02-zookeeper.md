```shell
# 前提：安装zookeeper之前需要安装JDK
# 解压安装包 
  tar xvf zookeeper-3.4.10.tar.gz
# 将ZOOKEEPER_HOME添加到环境变量
  vi /et/profile
  export PATH=$PATH:$ZOOKEEPER_HOME/bin
# 修改 vi /etc/profile 配置ZOOKEEPER_HOME
  export ZOOKEEPER_HOME=/home/hadoop/zookeeper-3.4.10	
# 创建zoo.cfg文件 
  cp zoo_sample.cfg zoo.cfg
# 编辑zoo.cfg
  vi zoo.cfg
  # 修改dataDir
  dataDir=/home/hadoop/tmp/zookper3410/data
  # 在zoo.cfg文件最后添加
  server.21=10.10.10.21:2888:3888
  server.22=10.10.10.22:2888:3888
  server.23=10.10.10.23:2888:3888
  ## 2888 
  ## 3888是选举酸苦
  # dataDir目录中创建名称为myid文件，并添加内容：21
  vi myid
# 启动zk服务
  ./zkServer.sh start 
# 启动zk客户端
  ./zkCli.sh
  ./zkCli.sh -server H21:2181
```

