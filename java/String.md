## String

在我所接触的编程语言中，字符串都是比比可少的，占据着非常重要的地位，java也是如此。在java中字符串是对象，而且还是不可变对象，对应着类`java.lang.String`，在日常的编程中用的非常多。

### String的特性

String类实现了字符序列接口`java.lang.CharSequence`，被关键字final修饰，属于不可扩展的类，也就是说String不能有子类。

字符串是不可变对象，字符串一旦被创建就不会发生任何改变。不管是字符串拼接还是截取，都只是生成了新的字符串对象，源字符串没有发生任何改变。总之，针对字符串的任何操作都不会改变这个字符串对象。至于为什么字符串被设计成不可变对象，个人以为其中很重要的一点便是为了安全考虑。字符串使用如此频繁，有意无意的改动必定会导致各种奇异的问题。

String的底层用char数组`value`存储数据，并且此value字段被final关键字修复，一旦赋值就不能再次被赋值，这也印证了上面说的`字符串是不可变对象`。

字符串类型变量的声明可以通过字面量的形式，也可以通过显示调用构造函数的形式。

补充一点：在java9以后的版本中，String的底层用byte数组存储数据，每个字符串所能容纳的字符几乎减少一半，但据说性能上有很大的提升。

### String常用方法

- boolean equals(Object anObject)
  将此字符串与指定的对象比较。

-  int length()
  返回该字符串的长度

- int compareToIgnoreCase(String str)
  按字典顺序比较两个字符串，不考虑大小写。

- String toLowerCase()
  使用默认语言环境的规则将此 String 中的所有字符都转换为小写。

- String toUpperCase()
  使用默认语言环境的规则将此 String 中的所有字符都转换为大写

- String trim()
  返回字符串的副本，忽略前导空白和尾部空白。

- String replaceAll(String regex, String replacement)
  使用给定的 replacement 替换此字符串所有匹配给定的正则表达式的子字符串。

- boolean startsWith(String prefix)
  测试此字符串是否以指定的前缀开始。

- int indexOf(String str)
   返回指定子字符串在此字符串中第一次出现处的索引。

- int lastIndexOf(String str)
  返回指定子字符串在此字符串中最右边出现处的索引。

- boolean endsWith(String suffix)
  测试此字符串是否以指定的后缀结束。

- String intern()
  返回字符串对象的规范化表示形式。

- static String valueOf(primitive data type x)
  返回给定data type类型x参数的字符串表示形式。

- static String format(String format, Object... args) 

  格式化字符

### String的intern()方法

intern用来返回常量池中的某字符串。如果常量池中已经存在该字符串，则直接返回常量池中该对象的引用；否则，在常量池中加入该对象，然后返回引用。说简单一点，该方法告诉JVM把当前字符串对象缓存起来，以便后续的复用。

事实上这个方法在平时的应用非常少，但是了解一下可以加深对字符串实现机制的理解。看下面的例子：

```java
public class StringTest {

	public static void main(String[] args) {

		String s1 = "a";
		String s2 = "a";
		System.out.println(s1 == s2); // true

		s1 = "b";
		s2 = new String("b").intern();
		System.out.println(s1 == s2); // true

		s1 = "a" + "b";
		s2 = "ab";
		System.out.println(s1 == s2); // true
		
		s1 = "c";
		s2 = new String("c");
		s2.intern();
		System.out.println(s1 == s2); // false

		s1 = "d";
		s2 = new String("d");
		System.out.println(s1 == s2); // false

		s1 = new String("e");
		s2 = new String("e");
		System.out.println(s1 == s2); // false
		
		s1 = "e" + "f";
		s2 = new String("e") + new String("f");
		System.out.println(s1 == s2); // false

	}
}
```

从上面的例子可以看出，在以下场景中，字符串对象会被缓存：

- 显示调用String的intern方法的时候; 
- 直接声明字符串字面常量的时候，例如: String s1 = "aaa";
-  字符串直接常量相加的时候，例如: String s1 = "a" + "b";  