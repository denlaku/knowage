# Set

### 不可变的Set

```scala
def main(args: Array[String]): Unit = {
    //创建一个空的set
    val set1 = new HashSet[Int]()
    println("set1=" + set1)
    //将元素和set合并生成一个新的set，原有set不变
    val set2 = set1 + 4
    println("set2=" + set2)
    //set中元素不能重复
    val set3 = set1 ++ Set(5,6,7)
    println("set3=" + set3)
    val set4 = Set(4,3,2) ++ set2
    println("set4=" + set4)
}
```

### 可变的Set

```scala
def main(args: Array[String]): Unit = {
    //创建一个可变的set
    val set1 = new mutable.HashSet[Int]()
    println("list1=" + set1)
    //向set1中添加元素
    set1 += 4
    //add等价于+=
    set1.add(5)
    set1 ++= Set(6,7,8)
    println("set1=" + set1)
    //删除一个元素
    set1 -= 8
    set1.remove(7)
    println("set1=" + set1)
}
```



