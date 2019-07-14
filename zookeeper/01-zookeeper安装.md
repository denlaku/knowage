## zookeeper安装

### 一、单机安装

#### 1、安装JDK

zookeeper由java语言开发，需要运行在JVM上。如果系统上之前没有安装，那第一步就需要安装jdk。

#### 2、下载安装包

下载地址：`https://www.apache.org/dyn/closer.cgi/zookeeper/`
这个页面里有很多镜像地址，选一个下载即可。

#### 3、上传并解压zookeeper安装包到指定目录

这里的版本：`zookeeper-3.4.10.tar.gz`
安装路径：`/home/hadoop/`
解压：`tar xvf zookeeper-3.4.10.tar.gz`

#### 4、设置环境变量

```shell
# 编辑profile文件
vi /et/profile
# 配置ZOOKEEPER_HOME
export ZOOKEEPER_HOME=/home/bigdata/zookeeper-3.4.10
# 将ZOOKEEPER_HOME加入环境变量
export PATH=$PATH:$ZOOKEEPER_HOME/bin
# 使设置及时生效 
source /etc/profile 
```

#### 5、创建并修改zookeeper配置文件

```shell
# 进入zookeeper配置文件目录
cd zookeeper-3.4.10\conf
# 创建zoo.cfg文件
cp zoo_sample.cfg zoo.cfg
# 编辑zoo.cfg
vi zoo.cfg
# 修改dataDir
dataDir=/tmp/zookper3410/data
```

#### 6、启动zookeeper服务端

```shell
# 进入zookeeper的bin目录
cd zookeeper-3.4.10\bin
# 启动zookeeper的server
./zkServer.sh start 
```

#### 7、zookeeper客户端连接服务端

```shell
# 进入zookeeper的bin目录
cd zookeeper-3.4.10\bin
# 启动zk客户端 
./zkCli.sh # 连接本机上zookeeper server
./zkCli.sh -server 10.10.10.11:2181 # 连接远程主机上的zookeeper server
```

### 二、集群安装

集群安装需要多个虚拟机，这里准备3个，如下：

+ 虚拟机1: 10.10.10.21
+ 虚拟机2: 10.10.10.22
+ 虚拟机3: 10.10.10.23

开始安装！

#### 1、修改配置文件

配置文件在单机版的基础上再加上如下内容

```shell
server.11=10.10.10.11:2888:3888
server.12=10.10.10.12:2888:3888
server.13=10.10.10.13:2888:3888
```

上面的三行表示三个不同的zookeeper节点
**11、12、13**分别是三个zookeeper节点的id。节点的id可以是任意的数字，但是必须唯一。
**2888** ：follower port
**3888**：election port 

#### 2、创建myid文件

每个zookeeper节点都必须在dataDir中创建myid文件，并在myid文件中加上此节点的id

#### 3、启动集群

完成以上步骤后，分别启动每个节点，zookeeper集群出来了。

