1、准备阶段，三套linux环境，IP规划如下

```shell
10.10.10.21 H21
10.10.10.22 H22
10.10.10.23 H23
```

2、编辑/etc/hostname文件，修改hostname

```shell
vi /etc/hostname
# 10.10.10.21的hostname分别修改为H21
# 10.10.10.22的hostname分别修改为H22
# 10.10.10.23的hostname分别修改为H23
```

3、 修改host

```shell
vi /etc/hosts
每个环境中的/etc/hosts文件都加上
10.10.10.21 H21
10.10.10.22 H22
10.10.10.23 H23
```

4、 关闭防火墙

```shell
# 
systemctl stop firewalld
# 
systemctl disable firewalld
```

5、 关闭SELinux

```shell
# 编辑 SELinux 配置文件
vim /etc/selinux/config
# 改状态
SELINUX=disabled
```

5 SSH设置
进入H30，执行命令 rpm -qa|grep ssh 查看ssh是否安装,如果没有安装下. （每套linux环境都需要安装）
进入root目录，如果没有.ssh，执行命令 mkdir .ssh 创建创建此目录 （每套linux环境都需要创建.ssh目录）

```shell
#进入root目录
cd /root
#创建.ssh目录
mkdir .ssh
#设置权限
chmod 700 .ssh
#检查
ls -al
#开始创建SSH密钥
#创建，后面3个回车
ssh-keygen -t rsa

#复制id_rsa.pub 到authorized_keys, 在.ssh目录执行命令
cat id_rsa.pub >> authorized_keys

#如果是普通用户
ssh-copy-id -i H02
ssh-copy-id -i H03
#拷贝H01中的ssh到H02、H03
scp /root/.ssh/authorized_keys root@H02:/root/.ssh/
scp /root/.ssh/authorized_keys root@H03:/root/.ssh/
```

6 安装JDK

```shell
# 上传jdk-8u144-linux-x64.tar.gz至目录/usr/lib, 解压安装包:
tar xvf jdk-8u144-linux-x64.tar.gz
# 编辑/etc/profile文件，配置JAVA_HOME
vi /etc/profile
export JAVA_HOME=/lib/java/jdk1.8.0_144
export JRE_HOME=/lib/java/jdk1.8.0_144/jre
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
# 执行source命令
source /etc/profile
```

7 安装hadoop

```shell
# 解压安装包
tar xvf hadoop-2.8.1.tar.gz
# 配置HADOOP_HOME, 在/etc/profile文件的最后加上
vi /etc/profile
export HADOOP_HOME=/root/hadoop-2.8.1
export PATH=$PATH:$HADOOP_HOME/bin
# 执行source命令
source /etc/profile
# 查看hadoop版本
hadoop version 
# 修改 vi hadoop-env.sh
将JAVA_HOME修改为 /lib/java/jdk1.8.0_144
# 修改 vi yarn-env.sh
将JAVA_HOME修改为 /lib/java/jdk1.8.0_144

# 创建文件夹
mkdir -p /home/hadoop/tmp/hadoop281/tmp
mkdir -p /home/hadoop/tmp/hadoop281/hdfs/name
mkdir -p /home/hadoop/tmp/hadoop281/hdfs/data

# 编辑 vi slaves 删除里边的localhost 修改内容为
H02
H03
#（H01作为master，H02、H03作为slave）
```

编辑 vi core-site.xml

```xml
<configuration>
	<property>
	  <name>fs.default.name</name>
	  <value>hdfs://H21:9000</value>
	  <description>HDFS的URI，文件系统://namenode标识:端口号</description>
	</property>

	<property>
	  <name>hadoop.tmp.dir</name>
	  <value>/home/hadoop/tmp/hadoop281/tmp</value>
	  <description>namenode上本地的hadoop临时文件夹</description>
	</property>

	<property>
	  <name>hadoop.proxyuser.root.groups</name>
	  <value>root</value>
	</property>

	<property>
	  <name>hadoop.proxyuser.root.hosts</name>
	  <value>H01</value>
	</property>
</configuration>
```

编辑 vi hdfs-site.xml

```xml
<configuration>
	<property>
	  <name>dfs.name.dir</name>
	  <value>/home/hadoop/tmp/hadoop281/hdfs/name</value>
	  <description>namenode上存储hdfs名字空间元数据 </description> 
	</property>

	<property>
	  <name>dfs.data.dir</name>
	  <value>/home/hadoop/tmp/hadoop281/hdfs/data</value>
	  <description>datanode上数据块的物理存储位置</description>
	</property>

	<property>
	  <name>dfs.replication</name>
	  <value>2</value>
	  <description>副本个数，配置默认是3,应小于datanode机器数量</description>
	</property>

	<property>  
	  <name>dfs.permissions</name>  
	  <value>false</value>  
	</property>
</configuration>
```

编辑 vi mapred-site.xml, 复制 cp mapred-site.xml.template mapred-site.xml

```xml
<configuration>
	<property>
	  <name>mapreduce.framework.name</name>
	  <value>yarn</value>
	</property>
</configuration>
```

编辑 vi yarn-site.xml

```xml
<configuration>
	<property>
	  <name>yarn.resourcemanager.hostname</name> 
	  <value>10.10.10.11</value>
	</property>

	<property> 
	  <name>yarn.nodemanager.aux-services</name> 
	  <value>mapreduce_shuffle</value> 
	</property>

	<property>
	  <name>yarn.resourcemanager.address</name>
	  <value>10.10.10.11:8032</value>
	</property>

	<property>
	  <name>yarn.resourcemanager.scheduler.address</name>
	  <value>10.10.10.21:8030</value>
	</property>

	<property>
	  <name>yarn.resourcemanager.resource-tracker.address</name>
	  <value>10.10.10.21:8031</value>
	</property>
	<property>
	  <name>yarn.resourcemanager.admin.address</name>
	  <value>10.10.10.21:8033</value>
	</property>

	<property>
	  <name>yarn.resourcemanager.webapp.address</name>
	  <value>10.10.10.21:8099</value>
	</property>
</configuration>
```

```shell
# 复制配置文件到H01、H03
# 在/root/hadoop-2.8.1/etc/hadoop中执行
scp * hadoop@H22:/home/hadoop/hadoop-2.8.1/etc/hadoop
scp * hadoop@H23:/home/hadoop/hadoop-2.8.1/etc/hadoop

# 格式化namenode，执行 
hadoop namenode -format

# 启动
./start-dfs.sh
./start-yarn.sh
```


	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	