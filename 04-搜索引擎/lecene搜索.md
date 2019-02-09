#### IndexReader  索引读取器

Open一个读取器，读取的是该时刻点的索引视图。如果后续索引发生改变，需重新open一个读取器。

```java
DirectoryReader.open(IndexWriter indexWriter)
DirectoryReader.open(Directory)
DirectoryReader.openIfChanged(DirectoryReader)
```

IndexReader分为两类：

​	CompositeReader 复合读取器
​	LeafReader 叶子读取器

叶子读取器：支持获取stored fields, doc values, terms（词项）, and postings （词项对应的文档）
复合读取器：多个读取器的复合。只可直接用它获取stored fields 。
			在内部通过CompositeReader.getSequentialSubReaders 得到里面的叶子读取器来获取其他数据。
DirectoryReader是复合读取器

**IndexReader**

```java
getDocCount(String)
getReaderCacheHelper()
getRefCount()
getSumDocFreq(String)
getSumTotalTermFreq(String)
getTermVector(int, String)
getTermVectors(int)
hasDeletions()
incRef()
leaves()
maxDoc()
numDeletedDocs()
numDocs()
registerParentReader(IndexReader)
totalTermFreq(Term)
tryIncRef()
```

**LeafReader**

```java
checkIntegrity()
docFreq(Term)
getBinaryDocValues(String)
getContext()
getCoreCacheHelper()
getDocCount(String)
getFieldInfos()
getLiveDocs()
getNormValues(String)
getNumericDocValues(String)
getPointValues(String)
getSortedDocValues(String)
getSortedNumericDocValues(String)
getSortedSetDocValues(String)
getSumDocFreq(String)
getSumTotalTermFreq(String)
postings(Term)
postings(Term, int)
terms(String)
totalTermFreq(Term)
```

**IndexSearcher**

应用通过调用它的search(Query,int)重载方法在一个IndexReader上实现搜索。出于性能的考虑，请使用一个IndexSearcher实例，除非索引发生变化。如索引更新了则通过DirectoryReader.openIfChanged(DirectoryReader)  取得新的读取器，再创建新的搜索器。

```java
search(Query, int)
search(Query, int, Sort)
search(Query, int, Sort, boolean, boolean)
search(Query, Collector)
search(Query, CollectorManager<C, T>)
searchAfter(ScoreDoc, Query, int)
searchAfter(ScoreDoc, Query, int, Sort)
searchAfter(ScoreDoc, Query, int, Sort, boolean, boolean)
```

TopDocs  搜索命中的结果集   （Top-N）

TopFieldDocs  按字段排序的搜索命中结果集

ScoreDoc

#### Query查询

TermQuery  词项查询
BooleanQuery  布尔查询
WildcardQuery 通配符查询
PhraseQuery 短语查询
PrefixQuery 前缀查询
MultiPhraseQuery 多重短语查询
FuzzyQuery 模糊查询

RegexpQuery 正则查询
TermRangeQuery 词项范围查询
PointRangeQuery 点范围查询
ConstantScoreQuery 
DisjunctionMaxQuery 
MatchAllDocsQuery
SpanNearQuery临近查询

#### QueryParser详解

传统解析器-单默认字段   QueryParser

```java
// 使用的分词器
Analyzer analyzer = new IKAnalyzer4Lucene7(true);
// 要搜索的默认字段
String defaultFiledName = "name";
// 查询生成器（解析输入生成Query查询对象）
QueryParser parser = new QueryParser(defaultFiledName, analyzer);
// 通过parse解析输入，生成query对象
Query query1 = parser.parse(
		"(name:\"联想笔记本电脑\" OR simpleIntro:英特尔) AND type:电脑 AND price:999900");
```

传统解析器-多默认字段  MultiFieldQueryParser

```java
// 传统查询解析器-多默认字段
String[] multiDefaultFields = { "name", "type", "simpleIntro" };
MultiFieldQueryParser multiFieldQueryParser = new MultiFieldQueryParser(
		multiDefaultFields, analyzer);
// 设置默认的组合操作，默认是 OR
multiFieldQueryParser.setDefaultOperator(Operator.OR);
Query query4 = multiFieldQueryParser.parse("笔记本电脑 AND price:1999900");
```

新解析框架的标准解析器：StandardQueryParser

```java
StandardQueryParser queryParserHelper = new StandardQueryParser(analyzer);
// 设置默认字段
// queryParserHelper.setMultiFields(CharSequence[] fields);
// queryParserHelper.setPhraseSlop(8);
// Query query = queryParserHelper.parse("a AND b", "defaultField");
Query query5 = queryParserHelper.parse(
	"(\"联想笔记本电脑\" OR simpleIntro:英特尔) AND type:电脑 AND price:1999900","name");

```

未分词的字段，应直接使用基本查询API加入到查询中，而不应使用查询解析器；
对于普通文本字段，使用查询解析器，而其他值字段：如 时间、数值，则应使用基本查询API

**查询语法规则**

Term 词项
	单个词项： 电脑
	没有引号，不会进行分词

短语
	短语的表示： "联想笔记本电脑"
	有引号，会分词

Field字段
	示例：name:“联想笔记本电脑” AND type:电脑
	如果字段是默认字段，则字段名可以省略

通配符
	? 单个字符
	0或多个字符
	通配符不可以写在开头

正则表达式
	示例： /xxx/

模糊查询
	示例： roam~
	模糊查询最大支持两个不同字符，词后加 ~

临近查询
	示例： "jakarta apache"~10
	短语后加 ~移动值

范围查询
	mod_date:[20020101 TO 20030101]       包含边界值
	title:{Aida TO Carmen}      不包含边界值

词项加权
	通过 ^数值来指定加权因子，默认加权因子值是1
	示例：jakarta^4 apache
	短语也可以： "jakarta apache"^4 "Apache Lucene"

Boolean 操作符
	Lucene支持的布尔操作： AND, “+”, OR, NOT ,"-"

​	OR:  "jakarta apache" jakarta 相当于 "jakarta apache" OR jakarta

​	AND："jakarta apache" AND "Apache Lucene"

​	+： 必须包含，如 +jakarta lucene

​	NOT 非："jakarta apache" NOT "Apache Lucene“
​		注意：NOT不可单项使用
​		NOT “Apache Lucene“     不可行

​	-  同NOT ："jakarta apache"  -"Apache Lucene“

组合 ()
	字句组合：(jakarta OR apache) AND website
	字段组合：title:(+return +"pink panther")

转义   \
	对语法字符： + - && || ! ( ) { } [ ] ^ “ ~ * ? : \ /     进行转义。
	如要查询包含 (1+1):2 
     		\(1\+1\)\:2 

​	



