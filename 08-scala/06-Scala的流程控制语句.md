## Scala流程控制语句

### 条件表达式

Scala的的条件表达式比较简洁，例如：

```scala
def main(args: Array[String]): Unit = {
    val x = 1
    //判断x的值，将结果赋给y
    val y = if (x > 0) 1 else -1
    //打印y的值
    println("y=" + y)

    //支持混合类型表达式
    val z = if (x > 1) 1 else "error"
    //打印z的值
    println("z=" + z)

    //如果缺失else，相当于if (x > 2) 1 else ()
    //如果条件成立 m = 1; 否则 m = ()
    val m = if (x > 2) 1
    println("m=" + m)

    //在scala中每个表达式都有值，scala中有个Unit类，写做(),相当于Java中的void
    val n = if (x > 2) 1 else ()
    println("n=" + n)

    //if和else if
    val k = if (x < 0) 0
    else if (x >= 1) 1 else -1
    println("k=" + k)
}
```

总结：
1）if条件表达式它是有返回值的
2）返回值会根据条件表达式的情况会进行自动的数据类型的推断; 支持混合类型表达式。

### 块表达式

```scala
def main(args: Array[String]): Unit = {
    val x = 0
    val result = {
      if(x < 0)
      else if(x >= 1)
        -1
      else
        "error"
    }
    println(result) // error
  }
```

### 循环

#### （1）while循环

```scala
def main(args: Array[String]): Unit = {
    var n = 10
    while ({
      n > 0
    }) {
      println(n)
      n -= 1
    }
}
```

#### （2）for循环

for循环语法结构：

> for (i <- 表达式/数组/集合)

```scala
def main(args: Array[String]): Unit = {
    //for(i <- 表达式),表达式1 to 10返回一个Range（区间）
    //每次循环将区间中的一个值赋给i
    for (i <- 1 to 10)
      print(i+"\t")

    //for(i <- 数组)
    println()
    val arr = Array("a", "b", "c")
    for (i <- arr)
      println(i)

    //高级for循环
    //每个生成器都可以带一个条件，注意：if前面没有分号
    for(i <- 1 to 3; j <- 1 to 3 if i != j)
      print((10 * i + j) + " ")
    println()

    //for推导式：如果for循环的循环体以yield开始，则该循环会构建出一个集合
    //每次迭代生成集合中的一个值
    val v = for (i <- 1 to 10) yield i * 10
    println(v)
}
```

**总结**：
1）在scala里面没有运算符，都有的符号其实都是方法。
2）在scala里面没有++  -- 的用法
3）for( i  <-  表达式/数组/集合)
4）在for循环里面我们是可以添加if表达式
5）有两个特殊表达式需要了解：

> To  1 to 3   1 2 3
> Until  1 until 3  12

6）如果在使用for循环的时候，for循环的时候我们需要获取，我们可以是使用yield关键字。