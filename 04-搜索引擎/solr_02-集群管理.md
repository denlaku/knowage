在linux系统上安装solrCloud

https://lucene.apache.org/solr/guide/7_3/taking-solr-to-production.html#taking-solr-to-production

从安装包中解出安装脚本

```shell
tar xzf solr-7.3.0.tgz solr-7.3.0/bin/install_solr_service.sh --strip-components=2
```

安装脚本参数说明：

```shell
./install_solr_service.sh -help
```

-i  指定软件安装目录。默认 /opt
-d 指定数据目录（solr主目录）：内核存储目录 。默认 /var/solr
-u  指定要创建的拥有solr的用户名，出于安全考虑，不应以root来运行。默认 solr
-s  指定系统服务名。默认 solr
-p  指定端口。默认 8983

安装(以root身份运行安装脚本)

```shell
# 默认安装目录和数据目录
./install_solr_service.sh solr-7.3.0.tgz
# 指定安装目录和数据目录
./install_solr_service.sh solr-7.3.0.tgz -i /opt -d /var/solr -u solr -s solr -p 8983
```

**solr服务的配置文件**
**系统服务脚本**： /etc/init.d/solr   请查看该脚本内容，看系统启动时是如何启动solr服务实例的。

```shell
SOLR_INSTALL_DIR=/opt/solr # Solr安装目录
SOLR_ENV=/etc/default/solr.in.sh # 环境参数配置文件
RUNAS=solr # 以哪个用户来运行
```

**环境参数配置文件**（官方叫法：include file）
它将覆盖 bin/solr启停控制脚本中的配置参数。我们通过该文件来配置修改solr服务实例的运行配置。
查看 /etc/default/solr.in.sh

```shell
# 调整solr实例的内存，默认solr使用512M的堆内存，生产环境下肯定需要调大。
SOLR_HEAP="512m"
SOLR_JAVA_MEM="-Xms512m -Xmx512m"
# 配置ZK_HOST，让服务实例以solrCloud模式运行
ZK_HOST=zk1,zk2:2182,zk3:2188
# 设置SOLR服务的主机名，在solrCloud模式下强烈建议配置。不设置则使用的是ip
SOLR_HOST=solr1.example.com

# 设置ZooKeeper chroot
# Solr默认使用zookeeper的/为其根目录，在多应用共用zookeeper的情况下，
# 为避免冲突，应该在单独的子节点（如/solr）下来存储solr的配置信息。这个节点需事先创建好
# 创建znode的命令： solr zk mkroot /solr -z <ZK_node>:<ZK_PORT>
ZK_HOST=zk1,zk2:2182,zk3:2188/solr

SOLR_PID_DIR=/var/solr # Solr进程id目录
SOLR_HOME=/var/solr/data # Solr主目录   
SOLR_PORT="8983" # 运行端口

LOG4J_PROPS=/var/solr/log4j.properties # 日志配置
SOLR_LOGS_DIR=/var/solr/logs
```

**覆盖solrconfig.xml中的设置**

```xml
<autoSoftCommit>
  <maxTime>${solr.autoSoftCommit.maxTime:-1}</maxTime>
</autoSoftCommit>
```

solrconfig.xml文件中类似于${solr.PROPERTY:DEFAULT_VALUE}
启动solr的时候，都可以用java属性覆盖。例如：

```shell
bin/solr start -Dsolr.autoSoftCommit.maxTime=10000
```

也可以配置到/etc/default/solr.in.sh文件中

```shell
SOLR_OPTS="$SOLR_OPTS -Dsolr.autoSoftCommit.maxTime=10000"
```



#### 集合管理

##### 创建集合

```shell
./solr create -c collection01 -d _default -shards 2 -replicationFactor 2 -p 8983
# 参照sample_techproducts_configs
./solr create -c collection02 -d sample_techproducts_configs -shards 2 -replicationFactor 2 -p 8983
```

##### 删除集合

```shell
./solr delete -c collectionName -p port
```

##### 集合管理API

https://lucene.apache.org/solr/guide/7_3/collections-api.html

##### 创建集合API

```
/solr/admin/collections?action=CREATE&name=newCollection&numShards=2&replicationFactor=2&wt=xml
```

常用参数说明：
name：指定集合名
numShards：初始分片数
replicationFactor：分片备份因子
collection.configName：参照的配置集名称（zookeeper的/configs下的子）
maxShardsPerNode：每个节点上允许的最大分片数，默认1
autoAddReplicas：true/false，在共享文件系统中可自动增加备份，默认false

##### 拆分分片

```
/admin/collections?action=SPLITSHARD&collection=name&shard=shardID
```

##### 删除备份

```shell
/solr/admin/collections?action=DELETEREPLICA&collection=test2&shard=shard2&replica=core_node3&wt=xml
```

##### 添加备份

```
/solr/admin/collections?action=ADDREPLICA&collection=test2
&shard=shard2&node=192.167.1.2:8983_solr&wt=xml
```

##### 添加备份

```
/solr/admin/collections?action=ADDREPLICA&collection=test2
&shard=shard2&node=192.167.1.2:8983_solr&wt=xml
```



#### 内核的主要配置文件

**内核属性文件**(core.properties)，是slor服务实例在solr主目录中自动发现内核的标识文件。
**内核配置文件**(solrconfig.xml)
**内核模式文件**(managed-schema)

##### core.properties

内核属性文件，是slor服务实例在solr主目录中自动发现内核的标识文件。
发现的规则：遍历主目录下的子目录树来查找core.properties，直到在一支上找到core.properties文件。

不合法的目录结构

```
./cores/core1/core.properties 
./cores/core1/coremore/core5/core.properties
```

合法的目录结构

```
./cores/somecores/core1/core.properties
./cores/somecores/core2/core.properties
./cores/othercores/core3/core.properties
./cores/extracores/deepertree/core4/core.properties
```

**core.properties 中可以配置的属性**

| Property      | Description                                                  |
| ------------- | ------------------------------------------------------------ |
| name          | 内核名                                                       |
| config        | 内核配置文件名，默认：solrconfig.xml.                        |
| schema        | 内核的模式文件名，默认是 schema.xml。但是请注意：如果你使用的是默认的“managed schema”，则在该处指定的模式文件名，只会被读取一次，并备份，转为 manageed schema使用。 |
| dataDir       | 内核的数据目录（存储索引），可以是绝对路径名，也可是相对内核目录的相对路径。默认值为相对路径 data。 |
| configSet     | 如需要用定义的配置集来配置内核，则指定配置集的名称           |
| properties    | 内核的属性文件名，可以是绝对路径名，也可是相对内核目录的相对路径 |
| transient     | 设置内核是否是瞬时的，如果设为true(瞬时的)，则当solr服务实例达到最大的transientCacheSize，内核可能被卸载（按最近最少使用来卸载）。默认 false。在SolrCloud模式下不要设为true。 |
| loadOnStartup | true 或没设置，则内核在solr启动时会被加载。在solrCloud模式下不要设置为false。 |
| coreNodeName  | 内核节点名，唯一标识存放这个备份的节点，仅在SolrCloud 模式下使用。这个名字默认是自动生成的，但通过手动设置该属性的值，我们可以手动地用一个新的内核替换已存在的备份。例如：当要替换一台硬件故障的机器时，我们可以在一台新的机器上（新的主机名、端口）根据备份恢复。 |
| ulogDir       | SolrCloud 模式下，存储内核更新日志的目录，可以是绝对或相对目录。 |
| shard         | 属于哪个分片(SolrCloud).                                     |
| collection    | 属于哪个集合(SolrCloud).                                     |
| roles         | Future param for SolrCloud or a way for users to mark nodes for their own use. |



#### 安全配置

##### Solr安全涉及的方面

Authentication or authorization  认证鉴权
Enabling SSL        使用 https 协议
If using SolrCloud, ZooKeeper Access Control     zookeeper的访问控制

##### Solr提供了多种认证鉴权插件

Kerberos Authentication Plugin
Basic Authentication Plugin
Rule-Based Authorization Plugin
Custom authentication or authorization plugin

##### 启用安全

Solr要求以在solr主目录下加入security.json 文件来启用安全控制，在solrCloud模式下，如在zookeeper中加入/security.json 。在security.json中定义认证、鉴权安全插件

```json
{
"authentication":{ 
   "blockUnknown": true, 
   "class":"solr.BasicAuthPlugin",
   "credentials":{"solr":"IV0EHq1OnNrj6gvRCwvFwTrZ1+z1oBbnQdiVC3otuq0= Ndd7LKvVBAaZIF0QAVi1ekCfAJXr1GGfLtRUXhgrF8c="} 
},
"authorization":{
   "class":"solr.RuleBasedAuthorizationPlugin",
   "permissions":[{"name":"security-edit",
      "role":"admin"}], 
   "user-role":{"solr":"admin"} 
}}
```

zookeeper中加入security.json

方式一：直接通过zk的客户端来添加节点
方式二：用 solr提供的操作脚本

```shell
server/scripts/cloud-scripts/zkcli.sh -zkhost localhost:2181 -cmd put /security.json 
'{"authentication": {"class": "org.apache.solr.security.KerberosPlugin"}}'
```

方式三：用 bin/solr zk 来操作

```shell
/solr zk cp file:path_to_local_security.json zk:/security.json -z localhost:9983
```

**Basic Authentication Plugin**

基于用户名密码的认证插件，官网参考：
https://lucene.apache.org/solr/guide/7_3/basic-authentication-plugin.htm

**Rule-Based Authorization Plugin**
基于用户、角色、权限规则的鉴权插件，官网参考：
https://lucene.apache.org/solr/guide/7_3/rule-based-authorization-plugin.html

#### 用户管理

##### API

v1: http://localhost:8983/solr/admin/authentication
v2: http://localhost:8983/api/cluster/security/authentication

**新增或修改密码**

```shell
curl --user solr:SolrRocks http://localhost:8983/api/cluster/security/authentication 
-H 'Content-type:application/json' -d '{"set-user": {"tom":"tom", "harry":"harry"}}'
```

**删除用户**

```shell
curl --user solr:SolrRocks http://localhost:8983/api/cluster/security/authentication 
-H 'Content-type:application/json' -d '{"delete-user": ["tom", "harry"]}'
```

**在SolrJ中使用基本认证**
在每个请求中都需要带上用户名密码

```java
SolrRequest req ;//create a new request object
req.setBasicAuthCredentials(userName, password);
solrClient.request(req);
```

查询示例

```java
QueryRequest req = new QueryRequest(new SolrQuery("*:*"));
req.setBasicAuthCredentials(userName, password);
QueryResponse rsp = req.process(solrClient);
```

