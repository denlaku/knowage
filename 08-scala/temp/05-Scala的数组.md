# Scala的数组

## 一、定长数组和变长数组

```scala
object ArrayTest {
  def main(args: Array[String]) {
    //初始化一个长度为8的定长数组，其所有元素均为0
    val arr1 = new Array[Int](8)
    //直接打印定长数组，内容为数组的hashcode值
    println("arr1="+arr1)
    //将数组转换成数组缓冲，就可以看到原数组中的内容了
    //toBuffer会将数组转换长数组缓冲
    println("arr1.toBuffer="+arr1.toBuffer)
    //注意：如果new，相当于调用了数组的apply方法，直接为数组赋值
    //初始化一个长度为10的定长数组
    val arr2 = Array[Int](10)
    println("arr2.toBuffer="+arr2.toBuffer)
    //定义一个长度为3的定长数组
    val arr3 = Array("hadoop", "storm", "spark")
    //使用()来访问元素
    println("arr3(2)="+arr3(2))

    //////////////////////////////////////////////////
    //变长数组（数组缓冲）
    //如果想使用数组缓冲，需要导入import scala.collection.mutable.ArrayBuffer包
    var ab = ArrayBuffer[Int]()
    //向数组缓冲的尾部追加一个元素
    //+=尾部追加元素
    ab += 1
    //追加多个元素
    ab += (2, 3, 4, 5)
    //追加一个数组++=
    ab ++= Array(6, 7)
    //追加一个数组缓冲
    ab ++= ArrayBuffer(8,9)
    //打印数组缓冲ab

    //在数组某个位置插入元素用insert
    ab.insert(0, -1, 0)
    //删除数组某个位置的元素用remove
    ab.remove(8, 2)
    println("ab="+ab)
  }
}
```

## 二、遍历数组

（1）增强for循环
（2）好用的until会生成脚标，**0 until 10 包含0不包含10**

```scala
def main(args: Array[String]) {
    //初始化一个数组
    val arr = Array(1,2,3,4,5,6,7,8)
    //增强for循环
    for(i <- arr)
      print(i+"\t")

    println()
    //好用的until会生成一个Range
    //reverse是将前面生成的Range反转
    for(i <- (0 until arr.length).reverse)
      print(arr(i))
}
```

## 3、数组转换

**yield**关键字将原始的数组进行转换会产生一个新的数组，原始的数组不变

```scala
def main(args: Array[String]) {
    //定义一个数组
    val arr = Array(1, 2, 3, 4, 5, 6, 7, 8, 9)
    //将偶数取出乘以10后再生成一个新的数组
    val res = for (e <- arr if e % 2 == 0) yield e * 10
    println(res.toBuffer)

    //更高级的写法,用着更爽
    //filter是过滤，接收一个返回值为boolean的函数
    //map相当于将数组中的每一个元素取出来，应用传进去的函数
    val r = arr.filter(_ % 2 == 0).map(_ * 10)
    println(r.toBuffer)
}
```

### 4、常用数组的算法

