# Scala的柯理化及作用

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



























