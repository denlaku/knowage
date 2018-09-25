# Scala正则表达式

Scala 通过 scala.util.matching 包中的 **Regex** 类来支持正则表达式。
以下实例演示了使用正则表达式查找单词 **Scala** :

```scala
import scala.util.matching.Regex

object Test {
	def main(args: Array[String]) {
		val pattern = "Scala".r("i")
		val str = "Scala is Scalable and cool scala"
		val first = pattern findFirstIn str
		println(first)
		println(first.mkString)
		
		val all = pattern findAllIn str
		println(all)
		println(all.mkString(","))
        
        pattern = new Regex("abl[ae]\\d+")
	}
}
```

