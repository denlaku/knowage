#### Analyzer

分析器，分词器的核心API。构建真正对文本进行分词处理的TokenStream（分词处理器）。
通过调用它的如下两个方法，得到输入文本的分词处理器。

```java
public final TokenStream tokenStream(String fieldName, Reader reader)
public final TokenStream tokenStream(String fieldName, String text)
```

Analyzer$TokenStreamComponents

AttributeSource
	TokenStream
		Tokenizer
		TokenFilter

AbstractAnalysisFactory
	TokenizerFactory
	TokenFilterFactory

#### TokenStreamComponents 

分词处理器组件：这个类中封装有供外部使用的TokenStream分词处理器。提供了对source(源)和sink（供外部使用分词处理器）两个属性的访问方法。

createComponents(String fieldName) 是Analizer中唯一的抽象方法，扩展点。

#### TokenStream

分词处理器，负责对输入文本完成分词、处理。

Tokenizer: 分词器，输入是Reader字符流的TokenStream，完成从流中分出分项

TokenFilter: 分项过滤器，它的输入是另一个TokenStream，完成对从上一个

**Attribute**

自定义的属性接口必须继承Attribute
实现类必须提供无参构造方法，实现类名必须接口名+Impl

```java
// 接口
public interface MyAttribute implements Attribute {
    
}
// 实现类
public class MyAttributeImpl implements MyAttribute {
    
}
```

#### IKAnalyzer

Ik中默认的停用词很少，我们往往需要扩展它。可从网址： https://github.com/cseryp/stopwords 下载一份比较全的停用词。

IK词典、停用词扩展：

1、在类目录下创建IK的配置文件：IKAnalyzer.cfg.xml
2、在配置文件中增加配置扩展停用词文件的节点。 创建扩展词文件 ext.dic， 一行一个词。
3、在配置文件中增加配置扩展词文件的节点。 创建扩展停用词文件 my_ext_stopword.dic， 一行一个词。
配置文件IKAnalyzer.cfg.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">ext.dic</entry>

	<!--用户可以在这里配置自己的扩展停止词字典 -->
	<entry key="ext_stopwords">stopwords.txt</entry>
</properties>
```

