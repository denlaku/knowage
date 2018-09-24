# Scala 模式匹配

一个模式匹配包含了一系列备选项，每个都开始于关键字 **case**。每个备选项都包含了一个模式及一到多个表达式。箭头符号 **=>** 隔开了模式和表达式。

以下是一个简单的整型值模式匹配实例：

```scala
object Test {
   def main(args: Array[String]) {
      println(matchTest(3))

   }
   def matchTest(x: Int): String = x match {
      case 1 => "one"
      case 2 => "two"
      case _ => "many"
   }
}
```

match 对应 Java 里的 switch，但是写在选择器表达式之后。即： **选择器 match {备选项}。**
match 表达式通过以代码编写的先后次序尝试每个模式来完成计算，只要发现有一个匹配的case，剩下的case不会继续匹配。
接下来我们来看一个不同数据类型的模式匹配：

```scala
object Test {
   def main(args: Array[String]) {
      println(matchTest("two"))
      println(matchTest("test"))
      println(matchTest(1))
      println(matchTest(6))

   }
   def matchTest(x: Any): Any = x match {
      case 1 => "one"
      case "two" => 2
      case y: Int => "scala.Int"
      case _ => "many"
   }
}
```

实例中第一个 case 对应整型数值 1，第二个 case 对应字符串值 two，第三个 case 对应类型模式，用于判断传入的值是否为整型，相比使用isInstanceOf来判断类型，使用模式匹配更好。第四个 case 表示默认的全匹配备选项，即没有找到其他匹配时的匹配项，类似 switch 中的 default。

## 使用样例类

使用了case关键字的类定义就是就是样例类(case classes)，样例类是种特殊的类，经过优化以用于模式匹配。

```scala
object Test {
   def main(args: Array[String]) {
       val alice = new Person("Alice", 25)
    val bob = new Person("Bob", 32)
       val charlie = new Person("Charlie", 32)
   
    for (person <- List(alice, bob, charlie)) {
        person match {
            case Person("Alice", 25) => println("Hi Alice!")
            case Person("Bob", 32) => println("Hi Bob!")
            case Person(name, age) =>
               println("Age: " + age + " year, name: " + name + "?")
         }
      }
   }
   // 样例类
   case class Person(name: String, age: Int)
}
```

在声明样例类时，下面的过程自动发生了：

- 构造器的每个参数都成为val，除非显式被声明为var，但是并不推荐这么做；
- 在伴生对象中提供了apply方法，所以可以不使用new关键字就可构建对象；
- 提供unapply方法使模式匹配可以工作；
- 生成toString、equals、hashCode和copy方法，除非显示给出这些方法的定义。



#### 1)常量模式(constant patterns) 包含常量变量和常量字面量

```scala
object MatchTest {
  def main(args: Array[String]): Unit = {
    constantPatterns("x") // is x
    constantPatterns("y") // is y
  }

  def constantPatterns(x: String) {
    x match {
      case "x" => println("is x")
      case "y" => println("is y")
      case _   => println("not match")
    }
  }
}
```

常量模式和普通的 if 比较两个对象是否相等(equals) 没有区别，并没有感觉到什么威力

#### 2) 变量模式(variable patterns)

```scala
object MatchTest {
  def main(args: Array[String]): Unit = {
    variablePatterns(List("baidu", "alibaba")) // baidu
    variablePatterns(List("baidu", "360", "alibaba")) // 360
  }

  def variablePatterns(xs: List[String]) {
    xs match {
      case List(x, _)    => println(x)
      case List(_, x, _) => println(x)
      case _             => println("not match")
    }
  }
}
```

确切的说单纯的变量模式没有匹配判断的过程，只是把传入的对象给起了一个新的变量名。
不过这里有个约定，对于变量，要求必须是以小写字母开头，否则会把它对待成一个常量变量。

变量模式通常不会单独使用，而是在多种模式组合时使用
里面的x就是对匹配到的第一个元素用变量x标记。

#### 3) 通配符模式(wildcard patterns)

```scala
object MatchTest {
  def main(args: Array[String]): Unit = {
    wildcardPatterns(List(1, 2)) // List(1, 2)
    wildcardPatterns(List(1, 3)) // List(1, _)
    wildcardPatterns(List(3, 4, 1)) // case List(3, 4, _)
    wildcardPatterns(List(5, 4)) // case List(5, 4, _*)
  }

  def wildcardPatterns(x: List[Int]) {
    x match {
      case List(1, 2)     => println("List(1, 2)")
      case List(1, _)     => println("List(1, _)")
      case List(3, 4, _)  => println("case List(3, 4, _)")
      case List(5, 4, _*) => println("case List(5, 4, _*)")
      case _              => println("not match")
    }
  }
}
```

通配符通常用于代表所不关心的部分，它不像变量模式可以后续的逻辑中使用这个变量。

#### 4) 构造器模式(constructor patterns)

```scala
object MatchTest {
  def main(args: Array[String]): Unit = {
    val tree = Tree(TreeNode("root", TreeNode("left", null, null), 
                             TreeNode("right", null, null)))
    constructorPatterns(tree.root) // bingo
  }
  
  def constructorPatterns(x: Node) {
    x match {
      case TreeNode(_, TreeNode("left", _, _), TreeNode("right", null, null)) =>
        println("bingo")
    }
  }
  

  //抽象节点
  trait Node
  //具体的节点实现，有两个子节点
  case class TreeNode(v: String, left: Node, right: Node) extends Node
  //Tree，构造参数是根节点
  case class Tree(root: TreeNode)

}
```

#### 5) 类型模式(type patterns)

类型模式很简单，就是判断对象是否是某种类型。

```scala
object MatchTest {
  def main(args: Array[String]): Unit = {
    typePattern("") // String
    typePattern(100) // Int
    typePattern(List(1, 2)) // List
  }

  def typePatterns(x: Any) {
    x match {
      case _: List[_] => println("List")
      case _: String  => println("String")
      case _: Int     => println("Int")
      case _          => println("unknow")
    }
  }

}
```

函数typePatterns的参数x的类型不能太具体了，例如x的类型为Int， 那么就没什么意义了

#### 6) 变量绑定模式 (variable binding patterns)

```scala
object MatchTest {
  def main(args: Array[String]): Unit = {
    val tree = Tree(TreeNode("root", TreeNode("left", null, null), 
                             TreeNode("right", null, null)))
    println(variableBindingPatterns(tree.root))
  }
  
  def variableBindingPatterns(x: Node): Node = {
    x match {
      case TreeNode(_, leftNode@TreeNode("left",_,_), _) => leftNode 
    }
  }
  

  //抽象节点
  trait Node
  //具体的节点实现，有两个子节点
  case class TreeNode(v: String, left: Node, right: Node) extends Node
  //Tree，构造参数是根节点
  case class Tree(root: TreeNode)
}
```

用`@`符号绑定 leftNode变量到匹配到的左节点上，只有匹配成功才会绑定

---

## for表达式中的模式匹配

在for表达式中

> for(x <- collection) { balabala } 

直觉上以为 x 就是个用于迭代每一个元素的局部变量。

#### 变量绑定

```scala
#
for(i@2 <- List(1,2,3) ) {println(i)}
```

将i绑定到常量模式2上，List(1,2,3)中只有2能匹配

```scala
#// 过滤出女性的名字
for ((name, "female") <- Set("wang" -> "male", "zhang" -> "female")) {
      print(name)
    }
```

同样，还可以类型模式在从集合过滤时按类型条件。

```scala
for ((k, v: Int) <- List(("A" -> 2), ("B" -> "C"))) {
    println(k)
}
```















