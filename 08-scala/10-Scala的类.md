# Scala的类

### 1、类的定义

scala语言中没有static成员存在，但是scala允许以某种方式去使用static成员。这个就是伴生机制，所谓伴生，就是在语言层面上，把static成员和非static成员用不同的表达方式，class和object，但双方具有相同的package和name，但是最终编译器会把他们编译到一起，这是纯粹从语法层面上的约定。通过javap可以反编译看到。

```scala
//在Scala中，类并不用声明为public。
//Scala源文件中可以包含多个类，所有这些类都具有公有可见性。
class ClassDemo {
  //用val修饰的变量是只读属性，有getter但没有setter
  //（相当与Java中用final修饰的变量）
  val id = 666
  //用var修饰的变量既有getter又有setter
  var name = "huangbo"
  //类私有字段,只能在类的内部使用
  var age = 24
  //对象私有字段,访问权限更加严格的，ClassDemo类的方法只能访问到当前对象的字段
  private[this] val address = "三里屯"
}
```

### 2、构造器

**注意：主构造器会执行类定义中的所有语句**

```scala
/**
  *每个类都有主构造器，主构造器的参数直接放置类名后面，与类交织在一起
  */
class Person(val name:String,val age:Int) {
  //主构造器会执行类定义中的所有语句
  println("Hello Spark")

  val x = 1
  if(x > 1){
    println("666")
  }else if(x < 1){
    println("哈哈。。。")
  }else{
    println("呵呵。。。")
  }

  private var address = "BJ"
  //用this关键字定义辅助构造器
  def this(name:String,age:Int,address:String){
    //每个辅助构造器必须以主构造器或其他的辅助构造器的调用开始
    this(name,age)
    println("执行辅助构造器")
    this.address = address
  }
}

object Person{
  def main(args: Array[String]): Unit = {
    val p = new Person("dengchao",33,"SH")
  }
}
```

**总结**：

主构造方法：
   1）与类名交织在一起
   2）主构造方法运行，导致类名后面的大括号里面的代码都会运行

辅助构造方法：
   1）必须名字叫this
   2) 必须以调用主构造方法或者是其他辅助构造方法开始。
   3）里面的属性不能写修饰符

### 3、对象

#### 单例对象

在Scala中没有静态方法和静态字段，但是**可以使用object这个语法结构来达到同样的目的**

1.存放工具方法和常量
2.高效共享单个不可变的实例=单例模式

**总结：**

1）object里面的方法都是静态方法
2）Object里面的字段都是静态字段
3）它本身就是一个单例，(因为不需要去new)

### 4、伴生对象

在同一个文件中，在Scala的类中，与类名相同的对象叫做伴生对象，类和伴生对象之间可以相互访问私有的方法和属性。

```scala
class Dog {
  val id = 666
  private var name = "道哥"
  def printName(): Unit ={
    //在Dog类中可以访问伴生对象Dog的私有属性
    println(Dog.CONSTANT + name)
  }
}

/**
  * 伴生对象
  */
object Dog{
  //伴生对象中的私有属性
  private var CONSTANT = "汪汪汪。。。"
  //主方法
  def main(args: Array[String]): Unit = {
    val dog = new Dog
    //访问私有的字段name
    dog.name = "道哥666"
    dog.printName()

  }
}
```

**总结：**

伴生对象和伴生类可以互相访问私有属性和私有方法。

### 5、apply方法

通常我们会在类的伴生对象中定义apply方法，当遇到类名(参数1,...参数n)时apply方法会被调用。例如Array类，我们可有直接用`Array(1, 2, 3)`来创建一个Array对象，就是用为在Array类的伴生对象中定义了apply方法，实际上Array对象是在apply中被创建的。

再举一个列子

```scala
class ApplyTest {
  
}

object ApplyTest {
  def main(args: Array[String]): Unit = {
    // 这里首先会调用apply方法，在apply方法中创建ApplyTest对象
    // 如果没有定义apply方法，这里就直接写成ApplyTest()，必须用new ApplyTest()语法
    var a = ApplyTest() 
    println(a)
  }

  def apply(): ApplyTest = {
    println("===apply===");
    new ApplyTest();
  }
}
```

### 6、继承

Scala中，让子类继承父类，与Java一样，也是使用**extends**关键字
继承就代表，子类可以从父类继承父类的field和method；然后子类可以在自己内部放入父类所没有，子类特有的field和method；使用继承可以有效复用代码
子类可以覆盖父类的field和method；但是如果父类用final修饰，则该类是无法被继承的; 如果父类的field和method用final修饰, field和method是无法被覆盖的

Scala中，**如果子类要覆盖一个父类中的非抽象方法，则必须使用override关键字**
override关键字可以帮助我们尽早地发现代码里的错误，
比如：override修饰的父类方法的方法名我们拼写错了；
比如要覆盖的父类方法的参数我们写错了等等；
此外，**在子类覆盖父类方法之后，如果我们在子类中就是要调用父类的被覆盖的方法**呢？那就可以使用**super关键字**，显式地指定要调用父类的方法

```scala
class People {
  private var name = "始皇帝"
  def getName = name
}

class Student extends People{
  private var score = 59
  def getScore = score

  override def getName: String = super.getName + ":嬴政"
}

object Test{
  def main(args: Array[String]): Unit = {
    val student = new Student
    println(student.getName)
  }
}
```

### 7、抽象类

如果在父类中，有某些方法无法立即实现，而需要依赖不同的子来来覆盖，重写实现自己不同的方法实现。
此时可以将父类中的这些方法不给出具体的实现，只有方法签名，这种方法就是抽象方法。
而一个类中如果有一个抽象方法，那么类就必须用abstract来声明为抽象类，此时抽象类是不可以实例化的。

**在子类中覆盖抽象类的抽象方法时，不需要使用override关键字**

```scala
abstract class AbstractDemo(name:String) {
  def sayHello:Unit
}
```

```scala
class StudentDemo(name:String) extends AbstractDemo(name){
  def sayHello: Unit = println("Hello " + name)
}

object StudentDemo{
  def main(args: Array[String]): Unit = {
    val li = new StudentDemo("Li")
    li.sayHello
  }
}
```

### 8、扩展类

在Scala中扩展类的方式和Java一样都是使用extends关键字

### 9、重写方法

在Scala中重写一个非抽象的方法必须使用override修饰符





