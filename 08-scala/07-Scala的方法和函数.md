## Scala方法和函数

### 一、定义方法

方法的返回值类型可以不写，编译器可以自动推断出来，但是**对于递归函数，必须指定返回类型。**
**如果不写等号，代表没有返回值。**

```scala
def f1(x: Int, y: Int):Int = {
    return x * y;
}
```

### 二、定义函数

```scala
val f1 = (x: Int, y: Int) => x*y
```

### 三、方法和函数的区别

在函数式编程语言中，函数是“头等公民”，**它可以像任何其他数据类型一样被传递和操作**
案例：首先定义一个方法，再定义一个函数，然后将函数传递到方法里面

```scala
object TestScala {
  //定义一个方法
  //方法m2参数要求是一个函数，函数的参数必须是两个Int类型
  //返回值类型也是Int类型
  def m1(f:(Int,Int) => Int) : Int = {
    f(2,6)
  }
  //定义一个函数f1，参数是两个Int类型，返回值是一个Int类型
  val f1 = (x:Int,y:Int) => x+y
  //再定义一个函数f2
  val f2 = (m:Int,n:Int) => m*n
  //main方法
  def main(args: Array[String]): Unit = {
    //调用m1方法，并传入f1函数
    val r1 = m1(f1)
    println("r1="+r1)
    //调用m1方法，并传入f2函数
    val r2 = m1(f2)
    println("r2="+r2)
  }
}
```

### 四、将方法转换成函数

```scala
def f1(x: Int, y: Int) = {
	x + y
};
// 将方法转换成函数
var f2 = f1 _
print(f2(20, 30));
```

### 五、传名函数和传值函数

Scala的解释器在解析函数参数(function arguments)时有两种方式：先计算参数表达式的值(reduce the arguments)，再应用到函数内部；或者是将未计算的参数表达式直接应用到函数内部。前者叫做传值调用（call-by-value），后者叫做传名调用（call-by-name）。

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

### 五、闭包

闭包是一个函数，返回值依赖于声明在函数外部的一个或多个变量。
闭包通常来讲可以简单的认为是可以访问一个函数里面局部变量的另外一个函数。



## 柯理化及作用

### 一、概念

柯里化(currying, 以逻辑学家Haskell Brooks Curry的名字命名)指的是将原来接受两个参数的函数变成新的接受一个参数的函数的过程。新的函数返回一个以原有第二个参数作为参数的函数。 在Scala中方法和函数有细微的差别，通常编译器会自动完成方法到函数的转换。

### 二、Scala中柯里化的形式

Scala中柯里化方法的定义形式和普通方法类似，区别在于柯里化方法拥有多组参数列表，每组参数用圆括号括起来，例如：

```scala
def mysum(x: Int)(y: Int) = x + y
println(mysum(20)(30)) // 50

var mysum1 = mysum _
println("mysum1: " + mysum1)
var mysum2 = mysum1(20)
println("mysum2: " + mysum2)
var mysum3 = mysum2(30)
println("mysum3: " + mysum3)
```

mysum方法拥有两组参数，分别是(x: Int)和(y: Int)。
mysum方法对应的柯里化函数类型是：

```scala
var fsum = (x: Int) => (y: Int) => x + y
println(fsum(20)(30)) // 50

var fsum1 = fsum(20)
println(fsum1)
var fsum2 = fsum1(20)
println(fsum2)
```

