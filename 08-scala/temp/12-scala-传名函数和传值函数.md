# scala-传名函数和传值函数

Scala的解释器在解析函数参数(function arguments)时有两种方式：先计算参数表达式的值(reduce the arguments)，再应用到函数内部；或者是将未计算的参数表达式直接应用到函数内部。前者叫做传值调用（call-by-value），后者叫做传名调用（call-by-name）。

### 传值函数和传名函数

```scala
object Test01 {

  // 传值函数
  def m1(a: Int, b: Int) = {
  }
  // 传名函数
  def m2(a: Int, b: => Int) = {
  }

  def main(args: Array[String]): Unit = {
    var f1 = (a: Int, b: Int) => {
      println("run")
      a + b
    }
   
    // 这里因为是传值，所以会执行表达式f1(200, 300)， 并打印书run
    m1(100, f1(200, 300)) 

    // 这里是传名，而且在方法m2中也没有用到第二个参数，所以不会执行表达式f1(200, 300)
    m2(100, f1(200, 300))

  }
}
```







