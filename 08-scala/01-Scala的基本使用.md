# Scala的基本使用

## 一、Scala概述

scala是一门多范式编程语言，集成了面向对象编程和函数式编程等多种特性。
scala运行在虚拟机上，并兼容现有的Java程序。
Scala源代码被编译成java字节码，所以运行在JVM上，并可以调用现有的Java类库。

## 二、第一个Scala程序

Scala语句末尾的分号可写可不写

```scala
object HelloScala {
    def main(args:Array[String]):Unit = {
        println("Hello Scala!")
    }
}
```

运行过程需要先进行编译， 编译之后生成2个文件：
HelloScala.class
HelloScala$.class

## 三、Scala的基本语法

```scala
/*
* Scala基本语法:
* 区分大小写
* 类名首字母大写
* 方法名称第一个字母小写
* 程序文件名应该与对象名称完全匹配
* scala程序从main方法开始处理，程序的入口
* 
* Scala注释：分为多行/**/ 单行 //
* 换行符：Scala是面向行的语言，语句可以用分号（；）结束或换行符（println()）
* 
* 定义包有两种方法：
*   1、package com.ahu
*      class HelloScala
*   2、package com.ahu{
*       class HelloScala
*     }
* 引用：import java.awt.Color
* 如果想要引入包中的几个成员，可以用selector（选取器）:
*  import java.awt.{Color,Font}
* 重命名成员
*  import java.util.{HashMap => JavaHashMap}
* 隐藏成员 默认情况下，Scala 总会引入 java.lang._ 、 scala._ 和 Predef._，所以在使用时都是省去scala.的
* import java.util.{HashMap => _, _} //引入了util包所有成员，但HashMap被隐藏了
*/
```





