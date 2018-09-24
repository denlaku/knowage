## 访问修饰符

Scala 访问修饰符基本和Java的一样，分别有：private，protected，public。
如果没有指定访问修饰符，默认情况下，Scala 对象的访问级别都是 public。

### 私有(Private)成员

用 private 关键字修饰，带有此标记的成员仅在包含了成员定义的类或对象内部可见，同样的规则还适用内部类。
Scala 中的 private 限定符，比 Java 更严格，在嵌套类情况下，外层类甚至不能访问被嵌套类的私有成员。

```scala
class Outer {
  class Inner{
    def start() = println("start")
    def end() = println("end")
    private def pause() = println("pause")
  }

  new Inner().start()
  new Inner().end()
  new Inner().pause() // 报错

```

在上面的代码里start和end两个方法被定义为public类型，可以通过任意Inner实例访问；pause被显示定义为private，这样就不能在Inner类外部访问它。执行这段代码，就会如注释处声明的一样，会在该处报错。

### 保护(Protected)成员

scala 中对保护（Protected）成员的访问比 java 更严格一些。因为它只允许保护成员在定义了该成员的的类的子类中被访问。而在java中，用protected关键字修饰的成员，除了定义了该成员的类的子类可以访问，同一个包里的其他类也可以进行访问。

```scala
class Super {
    protected def f(){println("f")}
}

class Sub extends Super {
    f() //OK
}

class Other {
    (new Super).f() //Error:f不可访问
}
```

上例中，Sub 类对 f 的访问没有问题，因为 f 在 Super 中被声明为 protected，而 Sub 是 Super 的子类。相反，Other 对 f 的访问不被允许，因为 other 没有继承自 Super。而后者在 java 里同样被认可，因为 Other 与 Sub 在同一包里。

### 公共(Public)成员

public访问控制为Scala定义的缺省方式,所有没有使用private和 protected修饰的成员(定义的类和方法)都是“公开的”,这样的成员可以在任何地方被访问。Scala不需要使用public来指定“公开访问”修饰符。
**注意**：Scala中定义的类和方法默认都是public的，但在类中声明的属性默认是private的。

### 作用保护域

作用域保护：Scala中，访问修饰符可以通过使用限定词强调。
private[x] 或者 protected[x]
private[x]：这个成员除了对[...]中的类或[...]中的包中的类及他们的伴生对象可见外，对其他的类都是private。

```scala
package bobsrockets {
    package navigation {
        //如果为private class Navigator,则类Navigator只会对当前包navigation中所有类型可见。
        //即private默认省略了[X],X为当前包或者当前类或者当前单例对象。
        //private[bobsrockets]则表示将类Navigator从当前包扩展到对bobsrockets包中的所有类型可见。
        private[bobsrockets] class Navigator {
            protected[navigation] def useStarChart() {}
            class LegOfJourney {
                private[Navigator] val distance = 100
            }
            private[this] var speed = 200
        }
    }
    package launch {
        import navigation._
		object Vehicle {
			//private val guide：表示guide默认被当前单例对象可见。
			//private[launch] val guide：表示guide由默认对当前单例对象可见扩展到对launch包中所有类型可见。
            private[launch] val guide = new Navigator
        }
    }
}
```

在这个例子中,类Navigator使用 private[bobsrockets] 来修饰,这表示这个类可以被bobsrockets包中所有类型访问,比如通常情况下 Vehicle无法访问私有类型Navigator,但使用包作用域之后,Vechile 中可以访问Navigator。