# 序列Seq

### 不可变的序列 import scala.collection.immutable._

在Scala中列表要么为空（**Nil表示空列表**）要么是一个head元素加上一个tail列表。

**9 :: List(5, 2)  :: 操作符是将给定的头和尾创建一个新的列表**

**注意：:: 操作符是右结合的，如9 :: 5 :: 2 :: Nil相当于 9 :: (5 :: (2 :: Nil))**

```scala
def main(args: Array[String]): Unit = {
    //创建一个不可变集合
    val list1 = List(1,2,3)
    //将0插入到集合list1的前面生成一个新的list
    val list2 = 0 :: list1
    val list3 = list1.:: (0)
    val list4 = 0 +: list1
    val list5 = list1. +: (0)
    println("list2=" + list2)
    println("list3=" + list3)
    println("list4=" + list4)
    println("list5=" + list5)

    //将3插入到集合list1的后面生成一个新的list
    val list6 = list1 :+ 3
    println("list6="+list6)

    //将2个list集合合并成一个新的list集合
    val list0 = List(7,8,9)
    println("list0="+list0)
    val list7 = list1 ++ list0
    println("list7="+list7)

    //将list0插入到list1前面生成一个新的集合
    val list8 = list0 ++: list1
    println("list8="+list8)

    //将list1插入到list0前面生成一个新的集合
    val list9 = list1 ++: list0
    println("list9="+list9)
}
```

#### 可变的序列 import scala.collection.mutable._

```scala
def main(args: Array[String]): Unit = {
    //构建一个可变序列
    val list1 = ListBuffer[Int](1,2,3)
    //创建一个空的可变序列
    val list2 = new ListBuffer[Int]
    //向list2中追加元素，注意：没有生成新的集合
    list2 += 6
    list2.append(7)
    println("list2=" + list2)

    //将list2中的元素追加到list1中，注意：没有生成新的集合
    list1 ++= list2
    print("list1=" + list1)

    //将list1和list2合并成一个新的list集合，注意：生成新的集合
    val list3 = list1 ++ list2
    println("list3=" + list3)

    //将元素追加到list1后面生成一个新的集合
    val list4 = list1 :+ 9
    print("list4=" + list4)
}
```



