## StringBuilder & StringJoiner

java中字符串被设计成不可变对象，对字符串的很作操作会得到一个新的字符串，看起来好像源字符串被修改了，实际上仅仅是创建了新的字符串对象，源字符串没有任何变化。

正因为如此，也产生了一下副作用，比如：字符串拼接。少量的字符串拼接还没什么，但是大量的字符串拼接，或者字符串频繁的拼接，会在内存中生成大量的中间字符串对象。为此javaAPI专门提供了此类操作的工具类`StringBuilder `、`StringJoiner`

### StringBuilder 

和String一样，StringBuilder底层存储数据的也是一个char数组。不同的是StringBuilder里的char数组是可变的，能被重新赋值，这也就保证了StringBuilder 的动态扩展性。

拼接字符串时直接调用方法`append(Object obj)`即可，在这个方法里字符串会被拆解成一个个的字符保存在char数组中。当需要最终结果时，只需调用`toString()`。StringBuilder的toString方法是被重写过的，里面的实现也非常简单：以char数据为参数，调用String的构造方法生成了一个新的字符串返回。

### StringBuffer

说到StringBuilder 就不得不提一下StringBuffer。实际上StringBuffer在JDK1.0就已经有了，StringBuilder直到JDK1.5才出现的。这两者中大的不同就是：StringBuffer是同步的，所有的方法都加上了关键字`synchronized`。所以StringBuffer的方法每次调用都会有获取、释放锁的过程，频繁的获取、释放锁是一种不小的开销，而且很多场景下同步是完全没有必要的。

开过过程中基本上用的都是`StringBuilder `，特别是在局部作用域下拼接字符串，推荐使用`StringBuilder `。

### StringJoiner

从JDK1.8版本起，新增了一个字符串工具类`StringJoiner`，这个类也是用作字符串拼接的，底层使用的`StringBuilder`。这个在拼接字符串的同时，也能指定字符串之间的分隔符，以及拼接结果字符串的前缀和后缀。

```java
import java.util.StringJoiner;

public class StringJoinerTest {
	public static void main(String[] args) {
		StringJoiner sj = new StringJoiner(",");
		sj.add("A").add("B").add("c");
		System.out.println(sj.toString()); // A,B,c

		StringJoiner sj2 = new StringJoiner( ",", "{", "}");
		sj2.add("1").add("2").add("3");
		System.out.println(sj2.toString()); // {1,2,3}
	}
}
```

