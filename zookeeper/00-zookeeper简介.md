## zookeeper

zookeeper是一个开源的分布式协调服务，由雅虎创建并开源。zookeeper的适用场景非常多，如：数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、配置维护、分布式同步、分布式锁和分布式队列等。

### zookeeper核心概念

#### **1、集群角色**

+ Leader
+ Follower
+ Observer

一个 ZooKeeper 集群同一时刻只能有一个 Leader，其他都是 Follower 或 Observer。zooKeeper 默认只有 Leader 和 Follower 两种角色，没有 Observer 角色。为了使用 Observer 模式，在任何想变成Observer节点的配置文件中加:peerType=observer，并在所有 server 的配置文件中，配置成 observer 模式的 server 的那行配置追加 :observer

#### **2 、节点读写服务分工**

1、zooKeeper 集群的Leader 是所有的Follower 节点通过选举过程选定的一台服务器，Leader服务器为客户端提供读和写服务。2、Follower 和 Observer 都能提供读服务，不能提供写服务。两者唯一的区别在于：Observer机器不参与 Leader 选举过程，也不参与写操作的**过半写成功**策略，因此Observer可以在不影响写性能的情况下提升集群的读性能。

#### **3 . Session**

Session 是指客户端会话，也就是客户端和 ZooKeeper服务器之间的TCP长连接。ZooKeeper对外的服务端口默认是2181。从第一次连接建立开始，客户端会话的生命周期也开始了，通过这个连接，客户端能够通过心跳检测和服务器保持有效的会话，也能够向 ZooKeeper 服务器发送请求并接受响应，同时还能通过该连接接收来自服务器的 Watch 事件通知。

Session 的 SessionTimeout 值用来设置一个客户端会话的超时时间。当由于服务器当压力太大、网络故障或是客户端主动断开连接等各种原因导致客户端连接断开时，只要在 SessionTimeout 规定的时间内能够重新连接上集群中任意一台服务器，那么之前创建的会话仍然有效。

#### **4 . 数据节点**

按照持久性可分为：持久节点和临时节点

+ 持久节点：一旦这个数据节点被创建了，除非主动进行移除操作，否则这个数据节点将一直保存在 ZooKeeper上。

+ 临时节点：生命周期跟客户端会话绑定，一旦客户端会话失效，那么这个客户端创建的所有临时节点都会被移除

按照有序性可分为：有序节点和无序节点

#### **5 . 状态信息**

每个节点除了存储数据内容之外，还存储了节点本身的一些状态信息。用 get 命令可以同时获得某个节点的内容和状态信息。在 zooKeeper中，version属性是用来实现乐观锁机制中的**写入校验**的（保证分布式数据原子性操作）。

123: 第一行节点数据
cZxid: 创建节点时的事务ID
ctime：节点创建时间
mZxid：修改节点时的事务ID
mtime：修改节点时间
pZxid：父节点事务ID
cversion：节点版本，每修改一个版本号加1
dataVersion：节点数据版本，每修改一个版本号加1
aclVersion：
ephemeralOwner：
dataLength：节点数据长度
numChildren：子节点数量

#### **6 .事物操作**

在ZooKeeper中，能改变ZooKeeper服务器状态的操作称为事务操作。一般包括数据节点创建与删除、数据内容更新和客户端会话创建与失效等操作。对应每一个事务请求，ZooKeeper都会为其分配一个全局唯一的事务ID，用 ZXID 表示，通常是一个64位的数字。每一个 ZXID对应一次更新操作，从这些 ZXID 中可以间接地识别出 ZooKeeper 处理这些事务操作请求的全局顺序。

#### 7 .Watcher(事件监听器)

是 ZooKeeper 中一个很重要的特性。ZooKeeper允许用户在指定节点上注册一些 Watcher，并且在一些特定事件触发的时候，ZooKeeper 服务端会将事件通知到感兴趣的客户端上去。该机制是 ZooKeeper 实现分布式协调服务的重要特性。









