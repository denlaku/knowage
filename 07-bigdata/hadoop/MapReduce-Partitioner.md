## MapReduce-Partitioner

在进行MapReduce计算时，有时候需要把最终的输出数据分到不同的文件中，比如按照省份划分的话，需要把同一省份的数据放到一个文件中；按照性别划分的话，需要把同一性别的数据放到一个文件中。我们知道最终的输出数据是来自于Reducer任务。那么，如果要得到多个文件，意味着有同样数量的Reducer任务在运行。Reducer任务的数据来自于Mapper任务，也就说Mapper任务要划分数据，对于不同的数据分配给不同的Reducer任务运行。Mapper任务划分数据的过程就称作Partition。负责实现划分数据的类称作Partitioner。

### getPartition()三个参数

HashPartitioner是处理Mapper任务输出的，getPartition()方法有三个形参，源码中**key、value分别指的是Mapper任务的输出，numReduceTasks指的是设置的Reducer任务数量，默认值是1**。那么任何整数与1相除的余数肯定是0。也就是说getPartition(…)方法的返回值总是0。也就是Mapper任务的输出总是送给一个Reducer任务，最终只能输出到一个文件中。

　　据此分析，如果想要最终输出到多个文件中，在Mapper任务中对数据应该划分到多个区中。那么，我们只需要按照一定的规则让getPartition(…)方法的返回值是0,1,2,3…即可。

