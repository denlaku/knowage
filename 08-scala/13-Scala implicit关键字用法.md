# scala implicit 关键字用法总结

implicit 关键字是scala中一个比较有特点的关键字，他保证了scala在很多时候没有一些不必要的代码冗余，使得scala在很多时候看起来更加简洁，同时使得scala的一些库在设计的时候，可以有更加直观的操作方法

# implicit function 隐式函数

第一种implicit的用法，是将其加在function定义的前面，形式为:

```scala
implicit def int2String(someInt: Int): String = {
  //...
}
```

这种用法可以用来进行implicit conversion，隐式转换，也就是说，编译器可以选择在合适的时候调用这些函数来进行一个转换，来保证类型的正确性，比如我可以通过定义一个implicit的转换函数将java的类型转换为scala的类型，这样在需要scala类型但是却使用java类型作为参数的时候，编译器会自动加入这个转换函数.

```scala
import scala.language.implicitConversions
//这个import是为了避免出现warning
object Test extends App {
  implicit def conv(a: Int) = {
    println("in conv")
    a.toString
  }
  def say(b: String) = println(b)
  say(5)
}
//输出结果:
// in conv
// 5
//这说明过程是say(conv(5))
//原因是编译器在检查的时候发现需要一个String类型的参数，但是代入的是一个Int，于是
//他会在范围内寻找implicit的function，找到了符合这个要求的String => Int的function，于是调用
//【注意】： 只能有一个符合转换要求的函数，否则会报错
```

# implicit parameter(隐式参数) 

隐式参数是在函数中，将参数标志出implicit，形式为:

```scala
def func(implicit x: Int)
def func2(x: Int)(implicit y: Int)
def func3(implicit x: Int, y: Int)
var func3 = (implicit x: Int)
```

这三种形式是有区别的，在参数中`implicit`只能出现一次，而在此之后，所有的参数都会变为implicit。

- func: x是implicit的
- func2: 只有y是implicit的
- func3: **x和y**都是implicit的 

注意避免以下几种错误写法:

```scala
//以下三种情况无法编译通过
def err(x: Int, implicit y: Int)
def err(implicit x: Int)(implicit y: Int)
def err(implicit x: Int)(y: Int)
```

# implicit value (隐式值)

```scala
implicit object Test
implicit val x = 5
implicit var y
```

这种用法的作用主要是**两种用法**(隐式参数和隐式值)搭配起来来达到一个效果，隐式参数表明这个参数是可以缺少的，也就是说在调用的时候这个参数可以不用出现，那么这个值由什么填充呢。 那就是用隐式的值了，以下的例子说明了这一点:

```scala
object Test extends App {
  abstract class Sayable {
    def say
  }
  implicit object hello extends Sayable{
    override def say() = {
      println("im in hello")
    }
  }
  def func(implicit x: Sayable) {
    x.say
  }
  func

  implicit val impVal = 5
  def func1(implicit x: Int) = {
    println(x)
  }

  func1
}
```

因为object的类型并不是object的名字，所以使用了一个抽象class来指明type

在调用func的时候，没有代入参数，其参数是由编译器检查之后决定的，而这里决定的就是唯一的可能，hello那个object，所以这里的say调用的就是hello object里的say

在调用func1的时候，同样没有代入参数，需要一个Int作为参数，编译器寻找值的时候寻找到impVal是implicit的值，所以这里选择impVal作为他的值，输出了5

## implicit class 隐式类

```scala
implicit class MyClass(x: Int)
```

这里的作用主要是其主构造函数可以作为隐式转换的参数，相当于其主构造函数可以用来当做一个implicit的function

```scala
object Test extends App {
  implicit class MyName(x: Int) {
    println("im in cons")
    val y = x
  }

  def say(x: MyName) = {
    println(x)
    println(x.y)
  }

  say(5)
}
```

这里的MyName是一个隐式类，其主构造函数可以用作隐式转换，所以say需要一个MyName类型的参数，但是调用的时候给的是一个Int，这里就会调用MyName的主构造函数转换为一个MyName的对象，然后再println其y的值

# 总结

我们在这里总结了implicit关键字在scala当中主要的几种用法，虽然没有举出实例，但是基本上对其语法结构和含义有了一些基本的了解，具体的使用还需要我们在实际情况中去体会什么时候需要这样的用法，这样使得代码更加简洁

