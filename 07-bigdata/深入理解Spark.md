## SparkContext的初始化

**SparkConf**： 通过ConcurrentHashMap维护Spark的各种配置信息。SparkContext初始化过程中会复制SparkConf参数，所以Spark启动之后配置参数时不能修改

```scala
 _conf = config.clone()
```

创建事件监听器

```scala
_listenerBus = new LiveListenerBus(_conf)
```

创建应用状态存储

```scala
_statusStore = AppStatusStore.createLiveStore(conf)
listenerBus.addToStatusQueue(_statusStore.listener.get)
```

创建Spark执行环境

```scala
_env = createSparkEnv(_conf, isLocal, listenerBus)
SparkEnv.set(_env)
```

创建Status Tracker，监听任务和state进程的API

```scala
_statusTracker = new SparkStatusTracker(this, _statusStore)
```

创建SparkUI

```scala
_ui =
if (conf.getBoolean("spark.ui.enabled", true)) {
    Some(SparkUI.create(Some(this), _statusStore, _conf, _env.securityManager, appName, "",
                        startTime))
} else {
    // For tests, do not enable the UI
    None
}
```

创建心跳接收，用于接收来自Exccutor的心跳

```scala
_heartbeatReceiver = env.rpcEnv.setupEndpoint(
      HeartbeatReceiver.ENDPOINT_NAME, new HeartbeatReceiver(this))
```

创建schedulerBackend和TaskScheduler

```scala
val (sched, ts) = SparkContext.createTaskScheduler(this, master, deployMode)
_schedulerBackend = sched
_taskScheduler = ts
```

创建DAGScheduler

```scala
_dagScheduler = new DAGScheduler(this)
```

创建ContextCleaner并启动

```scala
_cleaner =
if (_conf.getBoolean("spark.cleaner.referenceTracking", true)) {
    Some(new ContextCleaner(this))
} else {
    None
}
_cleaner.foreach(_.start())
```

### 创建执行环境SparkEnv

创建SecurityManager

```scala
val securityManager = new SecurityManager(conf, ioEncryptionKey)
```

创建RpcEnv

```scala
val rpcEnv = RpcEnv.create(systemName, bindAddress, advertiseAddress, port.getOrElse(-1), conf,
      securityManager, numUsableCores, !isDriver)
```

创建BroadcastManager

```scala
val broadcastManager = new BroadcastManager(isDriver, conf, securityManager)
```

创建Map任务输出跟踪器MapOutputTracker 

```scala
val mapOutputTracker = if (isDriver) {
    new MapOutputTrackerMaster(conf, broadcastManager, isLocal)
} else {
    new MapOutputTrackerWorker(conf)
}
```

ShuffleManager

```scala
val shuffleManager = instantiateClass[ShuffleManager](shuffleMgrClass)
```

MemoryManager

```scala
val memoryManager: MemoryManager =
if (useLegacyMemoryManager) {
    new StaticMemoryManager(conf, numUsableCores)
} else {
    UnifiedMemoryManager(conf, numUsableCores)
}
```

NettyBlockTransferService

```scala
val blockTransferService =
      new NettyBlockTransferService(conf, securityManager, bindAddress, advertiseAddress,
        blockManagerPort, numUsableCores)
```

BlockManagerMaster

```scala
val blockManagerMaster = new BlockManagerMaster(registerOrLookupEndpoint(
      BlockManagerMaster.DRIVER_ENDPOINT_NAME,
      new BlockManagerMasterEndpoint(rpcEnv, isLocal, conf, listenerBus)),
      conf, isDriver)
```

BlockManager

```scala
val blockManager = new BlockManager(executorId, rpcEnv, blockManagerMaster,
      serializerManager, conf, memoryManager, mapOutputTracker, shuffleManager,
      blockTransferService, securityManager, numUsableCores)
```

### Map任务输出跟踪器MapOutputTracker 



## 存储体系

### BlockManager

DiskBlockManager

```scala
val diskBlockManager = {
    // Only perform cleanup if an external service is not serving our shuffle files.
    val deleteFilesOnStop =
    !externalShuffleServiceEnabled || executorId == SparkContext.DRIVER_IDENTIFIER
    new DiskBlockManager(conf, deleteFilesOnStop)
}
```

BlockInfoManager

```scala
private[storage] val blockInfoManager = new BlockInfoManager
```

MemoryStore

```scala
private[spark] val memoryStore =
	new MemoryStore(conf, blockInfoManager, serializerManager, memoryManager, this)
```

DiskStore

```scala
private[spark] val diskStore = new DiskStore(conf, diskBlockManager, securityManager)
memoryManager.setMemoryStore(memoryStore)
```

ShuffleClient

```scala
private[spark] val shuffleClient = if (externalShuffleServiceEnabled) {
    val transConf = SparkTransportConf.fromSparkConf(conf, "shuffle", numUsableCores)
    new ExternalShuffleClient(transConf, securityManager,
    	securityManager.isAuthenticationEnabled(), conf.get(config.SHUFFLE_REGISTRATION_TIMEOUT))
} else {
    blockTransferService
}
```

blockManagerId & shuffleServerId

```scala
val id =
      BlockManagerId(executorId, blockTransferService.hostName, blockTransferService.port, None)
```

```scala
shuffleServerId = if (externalShuffleServiceEnabled) {
    logInfo(s"external shuffle service port = $externalShuffleServicePort")
    BlockManagerId(executorId, blockTransferService.hostName, externalShuffleServicePort)
} else {
    blockManagerId
}
```

### 磁盘管理器DiskBlockManager



### 磁盘存储DiskStore



### 内存存储MemoryStore



### 磁盘写入实现DiskBlockObjectWriter

### IndexShuffleBlockResolver

### ShuffleManager



## 任务提交与执行

#### BroadcastManager

#### BroadcastFactory -> TorrentBroadcastFactory

#### Broadcast -> TorrentBroadcast

#### ChunkedByteBufferOutputStream

#### ChunkedByteBufferInputStream

