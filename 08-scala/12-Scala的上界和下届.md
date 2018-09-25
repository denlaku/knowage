# Scala的上界和下届

## 一、泛型

### 1、泛型的介绍

泛型用于指定方法或类可以接受任意类型参数，参数在实际使用时才被确定，泛型可以有效地增强程序的适用性，使用泛型可以使得类或方法具有更强的通用性。泛型的典型应用场景是集合及集合中的方法参数，可以说同java一样，scala中泛型无处不在，具体可以查看scala的api。

### 2、泛型类、泛型方法 

**泛型类**：指定类可以接受任意类型参数。 
**泛型方法**：指定方法可以接受任意类型参数。

### 3、示例

#### 1、定义泛型类

```scala
/* 下面的意思就是表示只要是Comparable就可以传递,下面是类上定义的泛型
  */
class GenericTest1[T <: Comparable[T]] {

  def choose(one:T,two:T): T ={
    //定义一个选择的方法
    if(one.compareTo(two) > 0) one else two
  }

}

class Boy(val name:String,var age:Int) extends Comparable[Boy]{
  override def compareTo(o: Boy): Int = {
    this.age - o.age
  }
}

object GenericTestOne{
  def main(args: Array[String]): Unit = {
    val gt = new GenericTest1[Boy]
    val huangbo = new Boy("huangbo",60)
    val xuzheng = new Boy("xuzheng",66)
    val boy = gt.choose(huangbo,xuzheng)
    println(boy.name)
  }
}
```

#### 2、定义泛型方法

```scala
class GenericTest2{
  //在方法上定义泛型
  def choose[T <: Comparable[T]](one:T,two:T): T = {
    if(one.compareTo(two) > 0) one else two
  }
}

class Boy(val name:String,var age:Int) extends Comparable[Boy]{
  override def compareTo(o: Boy): Int = {
    this.age - o.age
  }
}

object GenericTestTwo{
  def main(args: Array[String]): Unit = {
    val gt = new GenericTest2
    val huangbo = new Boy("huangbo",60)
    val xuzheng = new Boy("xuzheng",66)
    val boy = gt.choose(huangbo,xuzheng)
    println(boy)
  }
}
```

## 二、上界和下届

### 1、介绍

在指定泛型类型时，有时需要界定泛型类型的范围，而不是接收任意类型。比如，要求某个泛型类型，必须是某个类的子类，这样在程序中就可以放心的调用父类的方法，程序才能正常的使用与运行。此时，就可以使用上下边界Bounds的特性； 
Scala的上下边界特性允许泛型类型是某个类的子类，或者是某个类的父类；

**(1) U >: T**
这是类型下界的定义，也就是U必须是类型T的父类(或本身，自己也可以认为是自己的父类)。

**(2) S <: T**
这是类型上界的定义，也就是S必须是类型T的子类（或本身，自己也可以认为是自己的子类)。

### 2、示例

#### （1）上界示例

参照上面的泛型方法

#### （2）下界示例

```scala
class GranderFather
class Father extends GranderFather
class Son extends Father
class Tongxue

object Card{
  def getIDCard[T >: Son](person:T): Unit ={
    println("OK,交给你了")
  }
  def main(args: Array[String]): Unit = {
    getIDCard[GranderFather](new Father)
    getIDCard[GranderFather](new GranderFather)
    getIDCard[GranderFather](new Son)
    //getIDCard[GranderFather](new Tongxue)//报错，所以注释
  }
}
```

## 三、协变和逆变

对于一个带类型参数的类型，比如 `List[T]`
**对A及其子类型B**:
**如果满足 List[B]也符合 List[A]的子类型，那么就称为covariance(协变)；**
**如果 List[A]是 List[B]的子类型，即与原来的父子关系正相反，则称为contravariance(逆变)。**

协变

```scala
____            　　_____________ 
|     |             |             |
|  A  |             |  List[ A ]  |
|_____|             |_____________|
   ^                       ^ 
   |                       | 
 _____               _____________ 
|     |             |             |
|  B  |             |  List[ B ]  |
|_____|             |_____________|
```

逆变

```scala
____            　　_____________ 
|     |             |             |
|  A  |             |  List[ B ]  |
|_____|             |_____________|
   ^                       ^ 
   |                       | 
 _____               _____________ 
|     |             |             |
|  B  |             |  List[ A ]  |
|_____|             |_____________|
```

在声明Scala的泛型类型时，“+”表示协变，而“-”表示逆变。

- C[+T]：如果A是B的子类，那么C[A]是C[B]的子类。
- C[-T]：如果A是B的子类，那么C[B]是C[A]的子类。
- C[T]：无论A和B是什么关系，C[A]和C[B]没有从属关系

根据Liskov替换原则，如果A是B的子类，那么能适用于B的所有操作，都适用于A。
让我们看看这边Function1的定义，是否满足这样的条件。假设Bird是Animal的子类，那么看看下面两个函数之间是什么关系：

```scala
def f1(x: Bird): Animal // instance of Function1[Bird, Animal]
def f2(x: Animal): Bird // instance of Function1[Animal, Bird]
```

在这里f2的类型是f1的类型的子类。为什么？

所以我们说，函数的参数类型是逆变的，而函数的返回类型是协变的。

那么我们在定义Scala类的时候，是不是可以随便指定泛型类型为协变或者逆变呢？答案是否定的。通过上面的例子可以看出，如果将Function1的参数类型定义为协变，或者返回类型定义为逆变，都会违反Liskov替换原则，因此，Scala规定，协变类型只能作为方法的返回类型，而逆变类型只能作为方法的参数类型。类比函数的行为，结合Liskov替换原则，就能发现这样的规定是非常合理的。

**总结：参数是逆变的或者不变的，返回值是协变的或者不变的。**

