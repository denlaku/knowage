#### ConcurrentSkipListMap

时间复杂度 O(logN)，使用空间换时间

最底层维护了调表所有的元素；每上面一层链表都是下面一层的子集；

调表内所有元素都是排序的，并且在查找过程中是跳跃式的；

高并发环境下，只需部分锁，性能较好；

#### 无锁：比较交换（CAS）

cas(V, E, N)

V 表示要更新的表里
E 表示预期的值
N 表示新值

#### 无锁的线程安全整数：AtomicInteger、AtomicLong

线程安全，使用CAS指令进行

#### 无锁的对象引用：AtomicReference

compareAndSet(V expect, V update)

对普通对象的引用，保证在修改对象引用的时候线程安全。

不能监控对象变化的过程

#### 带有时间戳的对象引用：AtomicStampedReference

compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp)

#### AtomicIntegerFieldUpdater

#### AtomicLongFieldUpdater

#### AtomicReferenceFieldUpdater

#### AtomicReferenceArray

#### 生产者-消费者模式

BlockingQueue
	offer
	take

#### 高性能的生产者-消费者：无锁的实现

无锁的缓存框架：Disruptor

```
<groupId>com.lmax</groupId>
  <artifactId>disruptor</artifactId>
  <version>3.4.2</version>
```



