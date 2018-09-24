## Scala的映射（Map）

在Scala中，把哈希表这种数据结构叫做映射

### 1、构建映射

```scala
// 用箭头的方式创建映射
val scoreMap = Map("yuwen" -> 130, "yingyu" -> 100);
println(scoreMap) // Map(yuwen -> 130, yingyu -> 100)
// 用元组的方式创建映射
scoreMap = Map(("yuwen", 130), ("yingyu", 100));
println(scoreMap) // Map(yuwen -> 130,  -> 100)
```

### 2、获取映射中的值

```scala
// 方式一
println(scoreMap("yuwen")) // 130
// 方式二
println(scoreMap.get("yuwen")); // Some(130)
println(scoreMap.get("yuwen").get); // 130
// 方式二： getOrElse有就返回对应的值，没有就返回默认值
println(scoreMap.getOrElse("yuwen", 0)); /130
```

注意：在Scala中，有两种Map，

一个是immutable包下的Map，**该Map中的内容不可变**；通常创建的映射都是不可变的;

另一个是mutable包下的Map，该Map中的内容可变 ;

```scala
val scoreMap = Map("yuwen" -> 130, "yingyu" -> 100);
// 修改映射的内容
scoreMap("yuwen") = 140
// 追加内容
scoreMap += ("shuxue", 135)
```

通常我们在创建一个集合是会用val这个关键字修饰一个变量（相当于java中的final），那么就意味着**该变量的引用不可变**; 该引用中的内容是不是可变，取决于这个引用指向的集合的类型

1：是否可以修改值

​	Mutable  可以修改map里面的值

​	Immutable 不可以修改里面的值

2：是否可以追加元素

​	mutable var/val 都可以追加元素

​	imtable var 可以追加，val不可以追加

