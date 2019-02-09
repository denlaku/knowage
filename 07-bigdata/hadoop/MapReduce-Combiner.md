## MapReduce-Combiner

combiner其实属于优化方案，由于带宽限制，应该尽量减少map和reduce之间的数据传输数量。它在Map端把同一个key的键值对合并在一起并计算，计算规则与reduce一致，所以combiner也可以看作特殊的Reducer。

执行combiner操作要求开发者必须在程序中设置了combiner(程序中通过job.setCombinerClass(myCombine.class)自定义combiner操作)。

Combiner组件是用来做局部汇总的，就在mapTask中进行汇总；Reducer组件是用来做全局汇总的，最终的，最后一次汇总。