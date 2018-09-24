## scala的元组

映射是K/V对偶的集合，对偶是元组的最简单形式，元组可以装着多个不同类型的值

### 1、创建元组

```scala
val t = ("hadoop", 123, 3.124)
println(t) // (hadoop,123,3.124)
```

### 2、获取元组中的值

获取元组中元素可以使用下划线加脚标；但是需要注意的是元组中的元素脚标是从1开始的

```scala
val t = ("hadoop", 123, 3.124)
println(t._1) // hadoop
println(t._2) // 123
println(t._3) // 3.124
```

### 3、将对偶的元组转换成映射

```scala
val arr = Array(("yuwen", 120), ("shuxue", 130));
println(arr.toMap); // Map(yuwen -> 120, shuxue -> 130)
```

### 4、拉链操作

**zip**命令可以将多个值绑定在一起

```scala
val scores = Array(120, 130, 99);
val names = Array("yuwen", "shuxue", "yingyu")
val ns = names.zip(scores);
println(ns.toMap) // Map(yuwen -> 120, shuxue -> 130, yingyu -> 99)
```

**注意：如果两个数组的元素个数不一致，拉链操作后生成的数组的长度为较小的那个数组的元素个数**

```scala
val scores = Array(120, 130, 99);
val names = Array("yuwen", "shuxue")
val ns = names.zip(scores);
println(ns.toMap) //Map(yuwen -> 120, shuxue -> 130)
```

















