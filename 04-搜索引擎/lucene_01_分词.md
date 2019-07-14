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

#### TokenStreamComponents createComponents(String fieldName)

是Analizer中唯一的抽象方法，扩展点。通过提供该方法的实现来实现自己的Analyzer。

参数说明：fieldName，如果我们需要为不同的字段创建不同的分词处理器组件，则可根据这个参数来判断。否则，就用不到这个参数。

返回值为 TokenStreamComponents  分词处理器组件。

我们需要在createComponents方法中创建我们想要的分词处理器组件。

输入流设置给了source

#### TokenStreamComponents

分词处理器组件：这个类中封装有供外部使用的TokenStream分词处理器。提供了对source(源)和sink（供外部使用分词处理器）两个属性的访问方法。

#### TokenStream

分词处理器，负责对输入文本完成分词、处理。

Tokenizer: 分词器，输入Reader字符流的TokenStream，完成从流中分出分项

TokenFilter: 分项过滤器，输入是另一个TokenStream，完成对从上一个TokenStream中流出的token的特殊处理。TokenFilter是一个典型的装饰器模式，如果我们需要对分词进行各种处理，只需要按我们的处理顺序一层层包裹即可（每一层完成特定的处理）。不同的处理需要，只需不同的包裹顺序、层数。

#### Attribute

1、自定义的属性接口 MyAttribute  继承 Attribute
2、自定义的属性实现类必须继承 AttributeImpl,并实现自定义的接口MyAttribute
3、自定义的属性实现类必须提供无参构造方法
4、为了让默认工厂能根据自定义接口找到实现类，实现类名需为 接口名+Impl 。

```java
// 接口
public interface MyAttribute implements Attribute {
    
}
// 实现类
public class MyAttributeImpl extends AttributeImpl implements MyAttribute {
    
}
```

#### TokenStream 的使用步骤

我们在应用中并不直接使用分词器，只需为索引引擎和搜索引擎创建我们想要的分词器对象。但我们在选择分词器时，会需要测试分词器的效果，就需要知道如何使用得到的分词处理器TokenStream，使用步骤：
1、从tokenStream获得你想要获得分项属性对象（信息是存放在属性对象中的）
2、调用 tokenStream 的 reset() 方法，进行重置。因为tokenStream是重复利用的。
3、循环调用tokenStream的incrementToken()，一个一个分词，直到它返回false
4、在循环中取出每个分项你想要的属性值。
5、调用tokenStream的end()，执行任务需要的结束处理。
6、调用tokenStream的close()方法，释放占有的资源。

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

