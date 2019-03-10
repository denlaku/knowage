## 优化HBase

**1、随机读密集型**

优化方向：高效利用缓存和更好的索引

-  增加缓存使用的堆的百分比，通过参数 hfile.block.cache.size 配置。
- 减少MemStore占用的百分比，通过hbase.regionserver.global.memstore.lowerLimit和hbase.regionserver.global.memstore.upperLimit来调节。
- 使用更小的数据块，使索引的粒度更细。
- 打开布隆过滤器，以减少为查找指定行的Key Value对象而读取的HFile的数量。
- 设置激进缓存，可以提升随机读性能。
- 关闭没有被用到随机读的列族，提升缓存命中率。

**2、顺序读密集型**

优化方向：减少使用缓存。

- 增大数据块的大小，使每次硬盘寻道时间取出的数据更多。
- 设置较高的扫描器缓存值，以便在执行大规模顺序读时每次RPC请求扫描器可以取回更多行。 参数 hbase.client.scanner.caching 定义了在扫描器上调用next方法时取回的行的数量。
- 关闭数据块的缓存，避免翻腾缓存的次数太多。通过Scan.setCacheBlocks(false)设置。
- 关闭表的缓存，以便在每次扫描时不再翻腾缓存。

**3、写密集型**

优化方向：不要太频繁刷写，合并或者拆分。

- 调高底层存储文件（HStoreFile）的最大大小，region越大意味着在写的时候拆分越少。通过参数 hbase.hregion.max.filesize设置。
- 增大MemStore的大小，通过参数hbase.hregion.memstore.flush.size调节。刷写到HDFS的数据越多，生产的HFile越大，会在写的时候减少生成文件的数量，从而减少合并的次数。
- 在每台RegionServer上增加分配给MemStore的堆比例。把upperLimit设为能够容纳每个region的MemStore乘以每个RegionServer上预期region的数量。
- 垃圾回收优化，在hbase-env.sh文件里设置，可以设置初始值为：-Xmx8g  -Xms8g  -Xmn128m  -XX:+UseParNewGC  -XX:+UseConcMarkSweepGC

　　　-XX:CMSInitiatingOccupancyFraction=70

- 打开MemStore-Local Allocation Buffer这个特性，有助于防止堆的碎片化。 通过参数hbase.hregion.memstore.mslab.enabled设置

**4、混合型**

优化方向：需要反复尝试各种组合，然后运行测试，得到最佳结果。

影响性能的因素还包括：

- 压缩：可以减少集群上的IO压力
- 好的行键设计
- 在预期集群负载最小的时候手工处理大合并
- 优化RegionServer处理程序计数