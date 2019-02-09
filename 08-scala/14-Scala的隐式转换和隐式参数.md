# Scala的隐式转换和隐式参数

### 一、概念

Scala 2.10引入了一种叫做隐式类的新特性。隐式类指的是用**implicit**关键字修饰的类。在对应的作用域内，带有这个关键字的类的主构造函数可用于隐式转换。

隐式转换和隐式参数是Scala中两个非常强大的功能，利用隐式转换和隐式参数，你可以提供优雅的类库，对类库的使用者隐匿掉那些枯燥乏味的细节。

### 二、作用

隐式的对类的方法进行增强，丰富现有类库的功能

### 三、隐式参数

1）关键字：implicit
2）隐式的东西只能在object里面才能使用
3）作用域

### 四、隐式转换函数

是指那种以implicit关键字声明的带有单个参数的函数。
可以通过：**:implicit –v**这个命令显示所有做隐式转换的类。

### 五、隐式转换的发生的时机

1、当一个对象去调用某个方法，但是这个对象并不具备这个方法

```scala
var v = 1.to(10)
println(v) // Range(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
```

1是Int类型，从to方法看，Int应该有to方法; 
打开Int类的源码查看，并没有Int本身并没有to方法，发现Int继承了AnyVal类;
查看AnyVal类，发现AnyVal类同样没有to方法，而AnyVal类继承了Any类;
Any类里面没有to方法，而在RichInt里面有to方法

**快学Scala示例值File和RichFile示例**

不使用隐式转换时，使用装饰模式进行读取

```scala
import java.io.File

import scala.io.Source

class RichFile(val file : File) {
  //定义一个read方法，返回String类型
  def read():String = Source.fromFile(file.getPath).mkString
}

object RichFile{
  //隐式转换方法（将原有的File类型转成了file类型，在用的时候需要导入相应的包）
  //implicit def file2RichFile(file:File) = new RichFile(file)
}

object MainApp{

  def main(args: Array[String]): Unit = {
    val file = new File("D:\\student.txt")
    //装饰模式，显示的增强(本来想实现：val contents = file.read()，但是却使用RichFile的方式，所以是显示的增强)
    val rf = new RichFile(file)
    val str = rf.read()
    print(str)
  }

}
```

使用隐式转换方式

```scala
import java.io.File

import scala.io.Source

class RichFile(val file : File) {
  //定义一个read方法，返回String类型
  def read():String = Source.fromFile(file.getPath).mkString
}

object RichFile{
  //隐式转换方法（将原有的File类型转成了file类型，在用的时候需要导入相应的包）
  implicit def file2RichFile(file:File) = new RichFile(file)
}

object MainApp{

  def main(args: Array[String]): Unit = {
    //目的是使用File的时候不知不觉的时候直接使用file.read()方法，所以这里就要做隐式转换
    val file = new File("D:\\student.txt")
    //导入隐式转换，._将它下满的所有的方法都导入进去了。
    import RichFile._
    //这里没有的read()方法的时候，它就到上面的这一行中的找带有implicit的定义方法
    val str = file.read()
    //打印读取的内容
    println(str)
  }

}
```

**超人示例**

```scala
class Man(val name:String)
class SuperMan {
  def fly(): Unit ={
    println("我要上天")
  }
}

object SuperMan{
  //隐式转换，将Man转换为SuperMan
  implicit def man2SuperMan(man:Man) = new SuperMan
  def main(args: Array[String]): Unit = {
    new Man("灰太狼").fly
  }
}
```

2、调用某个方法的时候，这个方法确实也存在，存入的参数类型不匹配

**售票厅卖票**

老人和小孩是特殊人群，有单独的买票窗口

```scala
//特殊人群（儿童和老人）
class SpecialPerson(var name:String)
//儿童
class Children(var name:String)
//老人
class Older(var name:String)
//青年工作者
class Worker(var name:String)

//特殊人群买票窗口
class TicketHouse{
  def buyTicket(p:SpecialPerson): Unit ={
    println(p.name + "买到票了")
  }
}

object MyPredef{
  //隐式转换，将儿童转换为特殊人群
  implicit def children2SpecialPerson(c:Children)=new SpecialPerson(c.name)
  //隐式转换，将老人转换为特殊人群
  implicit def older2SpecialPerson(o:Older)=new SpecialPerson(o.name)
}

object TestBuyTicket{
  def main(args: Array[String]): Unit = {
    //导入MyPredef类中的所有隐式转换
    import MyPredef._
    val house = new TicketHouse
    //测试儿童买票
    val children = new Children("wangbaoqiang")
    house.buyTicket(children)
    //测试老人买票
    val older = new Older("xuzheng")
    house.buyTicket(older)
    //测试青年工作者买票
    val worker = new Worker("huangbo")
    //house.buyTicket(worker)//放开的话会报错
  }
}
```

### 3、视图边界

```scala
class Person(val name : String) {
  def sayHello: Unit = {
    println("Hello, my name is " + name)
  }
  //2个人交朋友
  def mkFridens(p:Person): Unit ={
    sayHello
    p.sayHello
  }
}

class Student(name : String) extends Person(name)
class Dog(val name : String)
//聚会时2个人交朋友
class Party[T <% Person](p1:Person,p2:Person){
  p1.mkFridens(p2)
}

object Test{
  //隐式转换，将狗转换成人
  implicit def dog2Person(dog:Dog):Person={
    new Person(dog.name)
  }

  def main(args: Array[String]): Unit = {
    val huangxiaoming = new Person("huangxiaoming")
    val angelababy = new Student("angelababy")
    new Party[Person](huangxiaoming,angelababy)

    println("------------------------------------------------")

    val erlangshen = new Person("erlangshen")
    val xiaotianquan = new Dog("xiaotianquan")
    new Party[Person](erlangshen,xiaotianquan)
  }
}
```

