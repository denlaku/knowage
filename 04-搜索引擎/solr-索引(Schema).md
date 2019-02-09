### Schema介绍

Schema：模式，是集合/内核中字段的定义，让solr知道集合/内核包含哪些字段、字段的数据类型、字段该索引存储。

Schema 的定义方式

Solr中提供了两种方式来配置schema，两者只能选其一
默认方式，通过Schema API 来实时配置，模式信息存储在 内核目录的conf/managed-schema文件中。
传统的手工编辑conf/schema.xml的方式，编辑完后需重载集合/内核才会生效。

#### managed-schema文件的构成

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<schema version="1.6">
	<field .../>  
	<dynamicField .../>
	<uniqueKey>id</uniqueKey>
	<copyField .../>
	<fieldType ...>
		<analyzer type="index">
			<tokenizer .../>
			<filter ... />
		</analyzer>
		<analyzer type="query">
			<tokenizer.../>
			<filter ... />
		</analyzer>
	</fieldType>
</schema>

```

#### 字段Field

```xml
<field name="name" type="text_general" indexed="true" stored="true"/> 
<field name="includes" type="text_general" indexed="true" stored="true" 
       termVectors="true" termPositions="true" termOffsets="true" />
```

**字段属性说明**

**name**：字段名，必需。由字母、数字、下划线构成，不能以数字开头。以下划线开头和结尾的名字为保留字段名_text\_
**type**：字段的fieldType名，必需。为 FieldType定义的name 属性值。
**default**：默认值，如果提交的文档中没有该字段的值，则自动会为文档添加这个默认值。非必需。

**时间字段**

Solr中提供的时间字段类型（ DatePointField, DateRangeField,废除的TrieDateField ）是以时间毫秒数来存储时间的。要求字段值以ISO-8601标准格式来表示时间：YYYY-MM-DDThh:mm:ssZ

Z表示是UTC时间（注意：就没有时区了）。
秒上可以带小数来表示毫秒，超出精度部分会被忽略：
公元前：在前面加减号 -
9999后，在前面加加号 +

查询时如果是直接的时间串，需要用转移符转义：
datefield:1972-05-20T17\:33\:18.772Z  // 冒号需要转义
datefield:"1972-05-20T17:33:18.772Z" // 冒号不用转义，因为用了引号
datefield:[1972-05-20T17:33:18.772Z TO *] // 冒号不用转义，因为在中括号内

**DateRangeField** 时间范围

DateRangeField用来支持对时间段数据的索引，它遵守上一页讲到的时间格式，支持两种时间段表示方式：
方式一：截断日期，它表示整个日期跨度的精确指示。
方式二：范围语法 [ TO ]   { TO }

2000-11      	表示2000年11月整个月.
2000-11T13    	表示200年11月每天的13点这一个小时
-0009        	 公元前10年，0000是公元前1年。
[2000-11-01 TO 2014-12-01] 		日到日
[2014 TO 2014-12-01] 	2014年开始到2014-12-01止.
[* TO 2014-12-01] 		2014-12-01(含）前.

**时间数学表达式**

Solr中还支持用 NOW +- 时间的数学表达式来灵活表示时间。语法 NOW +- 带单位的时间数，/单位 截断。可用来表示时间段。

NOW+2MONTHS
NOW-1DAY
NOW/HOUR
NOW+6MONTHS+3DAYS/DAY
1972-05-20T17:33:18.772Z+6MONTHS+3DAYS/DAY

NOW在查询中使用时，可为NOW指定值
q=solr&fq=start_date:[* TO NOW]&NOW=1384387200000

```xml
<field name="birth" type="dt_type" docValues="false" indexed="true" stored="true"/>
<fieldType name="dt_type" class="solr.DatePointField" sortMissingLast="true" omitNorms="true" />

<field name="v_date" type="dt_range" docValues="false" indexed="true" stored="true"/>
<fieldType name="dt_type" class="solr.DatePointField" sortMissingLast="true" omitNorms="true" />
```

时间点(birth)和时间段(v_date)示例

```json
{
    "id":"3",
    "name":"中国",
    "birth":"2019-01-19T23:46:21.346Z",
    "v_date":"[2000-11-01 TO 2014-12-01]"
},
```



#### 字段类型FieldType

http://lucene.apache.org/solr/guide/7_3/field-types-included-with-solr.html

```xml
<fieldType name="managed_en" class="solr.TextField" positionIncrementGap="100">
  <analyzer type="index">
	<tokenizer class="solr.StandardTokenizerFactory"/>
	<filter class="solr.ManagedStopFilterFactory" managed="english" />
	<filter class="solr.ManagedSynonymGraphFilterFactory" managed="english" />
	<filter class="solr.FlattenGraphFilterFactory"/>
  </analyzer>
  <analyzer type="query">
	<tokenizer class="solr.StandardTokenizerFactory"/>
	<filter class="solr.ManagedStopFilterFactory" managed="english" />
	<filter class="solr.ManagedSynonymGraphFilterFactory" managed="english" />
  </analyzer>
</fieldType>
```

**fieldType的可选属性说明**

| Property                                                     | Description                                                  | 可选值     | 默认值 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------- | ------ |
| indexed                                                      | 是否索引                                                     | true/false | TRUE   |
| stored                                                       | 是否存储                                                     | true/false | TRUE   |
| docValues                                                    | 是否为该字段创建字段的列式存储（docValues)                   | true/false | FALSE  |
| sortMissingFirst sortMissingLast                             | 根据该字段排序时，没有该字段值的文档是排在前面还是后面       | true/false | FALSE  |
| multiValued                                                  | 字段是否是多值的                                             | true/false | FALSE  |
| omitNorms                                                    | 是否忽略标准化。对于所有 primitive (non-analyzed) field types, such as int, float, data, bool, and string，默认是true. Only full-text fields or fields need norms. | true/false | *      |
| omitTermFreqAndPositions                                     | 是否忽略词项的词频和位置                                     | true/false | *      |
| omitPositions                                                | 忽略词项的位置信息                                           | true/false | *      |
| termVectors<br />termPositions <br />termOffsets <br />termPayloads | 词项向量的存储                                               | true/false | FALSE  |
| required                                                     | 字段是否必需                                                 | true/false | FALSE  |
| useDocValuesAsStored                                         | 如果字段设置了存储docValues，而字段的stored=false，可以设置该属性为true，从而可以在搜索结果中返回该字段的值（从docValues中取值） 。 | true/false | TRUE   |
| large                                                        | 标识该字段的值是大尺寸的，从而对该字段值进行懒加载，只有值 < 512KB 的才会被缓存。 这个属性要求 stored="true" and multiValued="false". 主要作用就是不在内存中缓存大字段。 | true/false | FALSE  |

**FieldType的Analyzer**

对于 solr.TextField or solr.SortableTextField 字段类型，需要为其定义分析器。

```xml
<fieldType name="nametext" class="solr.TextField">
  <analyzer class="org.apache.lucene.analysis.core.WhitespaceAnalyzer"/>
</fieldType>
```

可以直接通过class属性指定分析器类，必须继承org.apache.lucene.analysis.Analyzer 。
也可灵活地组合分词器、过滤器：

```xml
<fieldType name="nametext" class="solr.TextField">
  <analyzer>
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StandardFilterFactory"/>
    <filter class="solr.LowerCaseFilterFactory"/>
    <filter class="solr.StopFilterFactory"/>
  </analyzer>
</fieldType>
```

如果该类型字段索引、查询时需要使用不同的分析器，则需区分配置analyzer

```xml
<fieldType name="nametext" class="solr.TextField">
  <analyzer type="index">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.LowerCaseFilterFactory"/>
    <filter class="solr.KeepWordFilterFactory" words="keepwords.txt"/>
    <filter class="solr.SynonymFilterFactory" synonyms="syns.txt"/>
  </analyzer>
  <analyzer type="query">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
</fieldType>
```

Solr中提供的tokenizer: http://lucene.apache.org/solr/guide/7_3/tokenizers.html
Solr中提供的 fiter： http://lucene.apache.org/solr/guide/7_3/filter-descriptions.html

**EnumFieldType枚举字段类型**

EnumFieldType  用于字段值是一个枚举集，且排序顺序可预定的情况，如新闻分类这样的字段。定义非常简单：

```xml
<fieldType name="priorityLevel" class="solr.EnumFieldType" docValues="true" 
           enumsConfig="my_priority_enum.xml" enumName="priority"/>
```

enumsConfig：指定枚举值的配置文件，绝对路径或相对 内核conf/的相对路径
enumName：指定配置文件的枚举名。排序顺序是按配置的顺序。
docValues : 枚举类型字段必须设置 true;

枚举文件示例my_priority_enum.xml

```xml
<enumsConfig>
	<enum name="priority">
		<value>Level1</value>
		<value>Level2</value>
		<value>Level3</value>
	</enum>
</enumsConfig>
```

**停用词过滤器Stop Filter**
停用词：索引中不存储的词

```xml
<analyzer>
  <tokenizer class="solr.StandardTokenizerFactory"/>
  <filter class="solr.StopFilterFactory" words="stopwords.txt"/>
</analyzer>
```

words属性指定停用词文件的绝对路径或相对 conf/目录的相对路径
停用词定义语法: 一行一个

**同义词过滤器Synonym Graph Filter**
同义词过滤、标准化应该在查询阶段做，而不应该在索引阶段做
如果同义词比较多，会增大索引的数据量
如果出现了新词，还需要重建索引

```xml
<analyzer type="index">
  <tokenizer class="solr.StandardTokenizerFactory"/>
  <filter class="solr.SynonymGraphFilterFactory" synonyms="mysynonyms.txt"/>
  <filter class="solr.FlattenGraphFilterFactory"/> 
</analyzer>
<analyzer type="query">
  <tokenizer class="solr.StandardTokenizerFactory"/>
  <filter class="solr.SynonymGraphFilterFactory" synonyms="mysynonyms.txt"/>
</analyzer>
```

同义词定义语法:
一类一行，
=>表示标准化为后面的

```xml
couch,sofa,divan
teh => the
huge,ginormous,humungous => large
small => tiny,teeny,weeny
```

#### 动态字段 dynamic Field

如果模式中有近百个字段需要定义，其中有很多字段的定义是相同，重复地定义是不是很烦？
可不可以定一个规则，字段名以某前缀开头或结尾的是相同的定义配置，
那这些重复字段就只需要配置一个，保证提交的字段名称遵守这个前缀、后缀即可。
这就是动态字段。
如：整型字段都是一样的定义，则可以定义一个动态字段如下：

```xml
<dynamicField name="*_i" type=“my_int" indexed="true" stored="true"/>
```

也可以是前缀，如    name=“i_*”

#### 复制字段CopyField

复制字段允许将一个或多个字段的值填充到一个字段中。它的用途有两种：
1、将多个字段内容填充到一个字段，来进行搜索
2、对同一个字段内容进行不同的分词过滤，创建一个新的可搜索字段
定义方式：
1、先定义一个普通字段

```xml
<field name="cc_all" type="zh_CN_text" indexed="true" stored="false" multiValued="false" />
```

2、定义复制字段

```xml
<copyField source="cat" dest="cc_all"/>
<copyField source="name" dest="cc_all"/>
```

复制字段时，source也可以是动态字段

#### uniqueKey 唯一键

指定用作唯一键的字段，非必需。

```xml
<uniqueKey>id</uniqueKey>
```

唯一键字段不可以是保留字段、复制字段，且不能分词。

#### 相关性计算

如果默认的相关性计算模型BM25Similarity不满足应用的特殊需要，可在schema中指定全局的或字段类型局部的相关性计算类

```xml
<similarity class="solr.SchemaSimilarityFactory">
  <str name="defaultSimFromFieldType">text_dfr</str>
</similarity>
<fieldType name="text_dfr" class="solr.TextField">
  <analyzer ... />
  <similarity class="solr.DFRSimilarityFactory">
    <str name="basicModel">I(F)</str>
    <str name="afterEffect">B</str>
    <str name="normalization">H3</str>
    <float name="mu">900</float>
  </similarity>
</fieldType>
```

#### 集成中文分词器IKAnalyzer 

1、在原来学习lucene集成IKAnalyzer的基础上，为IkAnalyzer实现一个TokenizerFactory（继承它），接收useSmart参数。

```java
public class IKAnalyzer4Lucene7Factory extends TokenizerFactory {
	
	private boolean userSmart;

	public IKAnalyzer4Lucene7Factory(Map<String, String> args) {
		super(args);
		String userSmart = args.get("userSmart");
		if ("true".equals(userSmart)) {
			this.userSmart = true;
		}
	}

	@Override
	public Tokenizer create(AttributeFactory factory) {
		return new IKTokenizer4Lucene7(this.userSmart);
	}

}
```

2、将这三个类打成jar，如  IKAnalyzer-lucene7.3.jar
3、将这个jar和 IKAnalyzer的jar 拷贝到web应用的lib目录下
4、将三个配置文件拷贝到应用的classes目录下
5、在schema中定义一个FieldType，使用IKAnalyzer适配类

```xml
<field name="zhName" type="zh_cn_text" indexed="true" stored="true" />
<fieldType name="zh_cn_text" class="solr.TextField">
    <analyzer type="index">
        <tokenizer class="com.denlaku.lucene.analyzer.IKAnalyzer4Lucene7Factory" userSmart="true"/>
        <filter class="solr.LowerCaseFilterFactory"/>
    </analyzer>
    <analyzer type="query">
        <tokenizer class="com.denlaku.lucene.analyzer.IKAnalyzer4Lucene7Factory" userSmart="true" />
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
        <filter class="solr.LowerCaseFilterFactory"/>
    </analyzer>
</fieldType>
```

#### Schema API

**更新 Schema**

发送 post请求到 /collection/schema  ，以JSON格式提交数据，在json中说明你要进行的更新操作及对应的数据（一次请求可进行多个操作），操作的定义见下页。

更新操作定义

 add-field: 添加一个新字段.
delete-field: 删除一个字段.
replace-field: 替换一个字段，修改.
add-dynamic-field: 添加一个新动态字段.
delete-dynamic-field: 删除一个动态字段
replace-dynamic-field: 替换一个已存在的动态字段
add-field-type: 添加一个fieldType.
delete-field-type: 删除一个fieldType.
replace-field-type: 更新一个存在的fieldType
add-copy-field: 添加一个复制字段规则.
delete-copy-field: 删除一个复制字段规则.

**V1、V2两个版本API说明**

V1老版本的api，V2新版本的API，当前两个版本的API都支持，将来会统一到新版本。两个版本的API只是请求地址上的区别，参数没区别。

V1： http://localhost:8983/solr/gettingstarted/schema
V2： http://localhost:8983/api/cores/gettingstarted/schema

#### FieldType字段类别操作

**添加一个字段类别** add-field-type

一个Analyzer

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field-type" : {
     "name":"myNewTxtField",
     "class":"solr.TextField",
     "positionIncrementGap":"100",
     "analyzer" : {
        "tokenizer":{
           "class":"solr.WhitespaceTokenizerFactory" },
        "filters":[{
           "class":"solr.WordDelimiterFilterFactory",
           "preserveOriginal":"0" }]}}
}' http://localhost:8983/solr/gettingstarted/schema
```

两个Analyzer

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field-type":{
     "name":"myNewTextField",
     "class":"solr.TextField",
     "indexAnalyzer":{
        "tokenizer":{
           "class":"solr.PathHierarchyTokenizerFactory",
           "delimiter":"/" }},
     "queryAnalyzer":{
        "tokenizer":{
           "class":"solr.KeywordTokenizerFactory" }}}
}' http://localhost:8983/api/cores/gettingstarted/schema
```

**删除一个字段类别** delete-field-type

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-field-type":{ "name":"myNewTxtField" }
}' http://localhost:8983/api/cores/gettingstarted/schema
```

**替换一个字段类别** replace-field-type

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field-type":{
     "name":"myNewTxtField",
     "class":"solr.TextField",
     "positionIncrementGap":"100",
     "analyzer":{
        "tokenizer":{
           "class":"solr.StandardTokenizerFactory" }}}
}' http://localhost:8983/api/cores/gettingstarted/schema
```

#### Field 字段操作

**添加一个字段** add-field

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field":{
     "name":"sell_by",
     "type":"pdate",
     "stored":true }
}' http://localhost:8983/api/cores/gettingstarted/schema
```

**删除一个字段** delete-field

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-field" : { "name":"sell_by" }
}' http://localhost:8983/api/cores/gettingstarted/schema
```

**替换一个字段** replace-field

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-field":{
     "name":"sell_by",
     "type":"date",
     "stored":false }
}' http://localhost:8983/api/cores/gettingstarted/schema
```

#### dynamicField 动态字段操作

**添加一个动态字段** add-dynamic-field

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-dynamic-field":{
     "name":"*_s",
     "type":"string",
     "stored":true }
}' http://localhost:8983/api/cores/gettingstarted/schema
```

**删除一个动态字段** delete-dynamic-field

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-dynamic-field":{ "name":"*_s" }
}' http://localhost:8983/api/cores/gettingstarted/schema
```

**替换一个动态字段** replace-dynamic-field

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "replace-dynamic-field":{
     "name":"*_s",
     "type":"text_general",
     "stored":false }
}' http://localhost:8983/solr/gettingstarted/schema
```

#### copyField 复制字段操作

**添加复制字段** add-copy-field

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-copy-field":{
     "source":"shelf",
     "dest":[ "location", "catchall" ]}
}' http://localhost:8983/api/cores/gettingstarted/schema
```

**删除复制字段** delete-copy-field

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "delete-copy-field":{ "source":"shelf", "dest":"location" }
}' http://localhost:8983/api/cores/gettingstarted/schema
```

#### 一次请求多个操作

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field-type":{
     "name":"myNewTxtField",
     "class":"solr.TextField",
     "positionIncrementGap":"100",
     "analyzer":{"tokenizer":{
           "class":"solr.WhitespaceTokenizerFactory" },
        "filters":[{
           "class":"solr.WordDelimiterFilterFactory",
           "preserveOriginal":"0" }]}},
   "add-field" : {
      "name":"sell_by",
      "type":"myNewTxtField",
      "stored":true }
}' http://localhost:8983/solr/gettingstarted/schema
```

```shell
curl -X POST -H 'Content-type:application/json' --data-binary '{
  "add-field":[
     { "name":"shelf",
       "type":"myNewTxtField",
       "stored":true },
     { "name":"location",
       "type":"myNewTxtField",
       "stored":true }]
}' http://localhost:8983/solr/gettingstarted/schema
```

#### 获取schema信息

**获取整个schema**

```shell
GET /api/cores/mycore/schema
GET /collection/schema
```

可以通过wt请求参数指定返回的格式：json，xml， schema.xml

```shell
http://localhost:8983/api/cores/mycore/schema?wt=xml
```

**获取字段**

```shell
GET /collection/schema/fields
GET /collection/schema/fields/fieldname
```

请求参数有：
wt:   json/xml
fl：指定需要返回的字段名，以逗号或空格间隔
showDefaults：true/false ，是否返回字段的默认属性
includeDynamic：true/false，在path中带有fieldname  或指定了 fl的情况下才有用。

**获取动态字段**

```shell
GET /collection/schema/dynamicfields
GET /collection/schema/dynamicfields/name

GET /api/cores/mycore/schema/fields
GET /api/cores/mycore/schema/dynamicfields
```

可用请求参数：wt、showDefaults

```shell
http://localhost:8983/api/cores/mycore/schema/dynamicfields?wt=xml
```

**获取复制字段**

```shell
GET /collection/schema/copyfields
```

可用请求参数：wt、 source.fl、 dest.fl

**获取字段类别**

```shell
GET /collection/schema/fieldtypes
GET /collection/schema/fieldtypes/name
```

可用请求参数：wt、showDefaults

```shell
http://localhost:8983/api/cores/mycore/schema/fieldtypes?wt=xml
```

