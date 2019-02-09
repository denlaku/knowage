# 特质(trait)

### 1、将特质作为接口使用

```scala
/**
 * // Scala中的Triat是一种特殊的概念
// 首先我们可以将Trait作为接口来使用，此时的Triat就与Java中的接口非常类似
// 在triat中可以定义抽象方法，就与抽象类中的抽象方法一样，只要不给出方法的具体实现即可
// 类可以使用extends关键字继承trait，注意，这里不是implement，而是extends，
// 在scala中没有implement的概念，无论继承类还是trait，统一都是extends
// 类继承trait后，必须实现其中的抽象方法，实现时不需要使用override关键字
// scala不支持对类进行多继承，但是支持多重继承trait，使用with关键字即可
 */
trait HelloTrait {
  def sayHello(name:String)
}

trait MakeFriendsTrait{
  def makeFriends(w:Worker)
}
class Worker(var name:String) extends HelloTrait with MakeFriendsTrait{
  def sayHello(name:String) = println("hello ,"+name)
  def makeFriends(w:Worker)=println("hello, my name is"+name+" you name is"+w.name)
}

object Test{
  def main(args: Array[String]) {
    val p1=new Worker("xiaoma");
    val p2=new Worker("linghuchong")
    p1.sayHello("lihuchong")
    p1.makeFriends(p2)
  }
}
```

### 2、在trait中定义具体方法

```scala
/**
 // Scala中的Triat可以不是只定义抽象方法，还可以定义具体方法，此时trait更像是包含了通用工具方法的东西
// 有一个专有的名词来形容这种情况，就是说trait的功能混入了类
// 举例来说，trait中可以包含一些很多类都通用的功能方法，比如打印日志等等，spark中就使用了trait来定义了通用的日志打印方法
 */
trait Logger {
  def log(message:String) = println(message)
}

class Person(val name: String) extends Logger{
  def makeFridends(p:Person): Unit ={
    println("I'm"+name+" i'm glade to make friends with you"+p.name);
    log("makeFridends method invoked!!")
  }
}
object Test{
  def main(args: Array[String]) {
    val p1=new Person("linpingzhi")
    val p2=new Person("yuelingshan");
    p1.makeFridends(p2)
  }
}
```

### 3、在trait中定义具体字段

```scala
/**
 * // Scala中的Triat可以定义具体field，此时继承trait的类就自动获得了trait中定义的field
// 但是这种获取field的方式与继承class是不同的：如果是继承class获取的field，实际是定义在父类中的；
  而继承trait获取的field，就直接被添加到了类中
 */
trait Person {
  val eyeNum:Int=2
}
class Student(val name:String) extends Person{
  def sayHello()=println("Hi,I'm "+name +"I have "+eyeNum+"eyes !" )
}
object Test{
  def main(args: Array[String]) {
    val s=new Student("zhangsanfeng")
    s.sayHello();
  }
}
```

### 4、 在trait中定义抽象字段

```scala
/**
 * // Scala中的Triat可以定义抽象field，而trait中的具体方法则可以基于抽象field来编写
// 但是继承trait的类，则必须覆盖抽象field，提供具体的值
 */
trait sayHello {
  val msg:String
  def sayHello(name:String)=println(msg +" , "+name)
}

class Person(val name:String) extends sayHello{
  val msg:String = "hello"
  def makeFriends(p:Person): Unit ={
    sayHello(p.name)
    println("I'm"+name +" I want to make frieds with you")
  }
}

object Test{
  def main(args: Array[String]) {
    val p1=new Person("zhangwuji")
    val p2=new Person("zhangsanfeng")
    p1.makeFriends(p2)

  }
}
```

### 5、为实例对象混入trait

```scala
/**
有时我们可以在创建类的对象时，指定该对象混入某个trait，
这样，就只有这个对象混入该trait的方法，而类的其他对象则没有
 */
trait Logged {
  def log(msg:String){}

}
trait AMyLogger extends Logged{
  override def log(msg:String): Unit ={
    println("test:"+msg)
  }
}
trait BMyLogger extends Logged{
  override def log(msg:String): Unit ={
    println("log:"+msg)
  }
}
class Person(val name:String) extends AMyLogger{
  def sayHello(): Unit ={
    println("Hi ,i'm " + name)
    log("sayHello is invoked!")
  }
}
object  Test{
  def main(args: Array[String]) {
    val p1=new Person("liudehua")
    p1.sayHello() 

    val p2=new Person("zhangxueyou") with BMyLogger
    p2.sayHello()
  }
}
```

### 6、trait调用链

```scala
/**
// Scala中支持让类继承多个trait后，依次调用多个trait中的同一个方法，
// 只要让多个trait的同一个方法中，在最后都执行super.方法即可
// 类中调用多个trait中都有的这个方法时，首先会从最右边的trait的方法开始执行，然后依次往左执行，形成一个调用链条
// 这种特性非常强大，其实就相当于设计模式中的责任链模式的一种具体实现依赖
 */
trait Handler {
  def handler(data:String){}
}
trait DataValidHandler extends Handler{
  override def handler(data:String): Unit ={
    println("check data:"+data)
    super.handler(data)
  }
}
trait SignatureValidHandler extends Handler{
  override def handler(data:String): Unit ={
    println("check signatrue:"+data)
    super.handler(data)
  }
}
class  Person(val name:String) extends SignatureValidHandler with DataValidHandler{
  def sayHello={
    println("Hello "+name)
    handler(name)
  }
}

object Test{
  def main(args: Array[String]) {
    val p=new Person("lixiaolong");
    p.sayHello
  }
}
```

