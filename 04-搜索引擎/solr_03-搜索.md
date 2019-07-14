### Solr搜索的工作流程

1 用户/应用输入查询需求（查询字符串）
qt：选择请求处理器，查询为：/select



**Request Handle**
defType：选择一个查询解析器来解析用户查询串，默认使用RequestHander中配置的默认解析器。

**Query Parser**
qf：根据指定的字段进行搜索，默认是所有索引字段。

**Index**
fq：对结果进行过滤
分组聚合等处理
sort：排序
start、rows：分页
转换处理：高亮
wt:选择输出格式

#### SearchHandler介绍

**SearchHandler中配置的 default**



**内核的solrconfig.xml的requestHandler**

|          |                  |                                                              |
| -------- | ---------------- | ------------------------------------------------------------ |
| **元素** | **说明**         | **举例**                                                     |
| <bool>   | 布尔型参数       | <bool name="xx">true</bool>                                  |
| <str>    | 字符串           | <str name="ab">aaaa</str>                                    |
| <int>    | 整型值           | <int name="a">1000</int>                                     |
| <long>   | 长整型           | <long name="b">10000</long>                                  |
| <float>  | 单精度浮点数     | <float name="c">100.89</float>                               |
| <double> | 双精度浮点数     | <double name="e">9.5678</double>                             |
| <arr>    | 命名的有序数组   | <arr name="xxxarry">    <str>aaa</str>    <str>bbb</str></arr> |
| <lst>    | 命名的键值对列表 | <lst name="xxxlts">    <str name="n1">ssss</str>    <str name="n2">aaa</str></lst> |

查询请求在SearcheHandler这个request handler中完成，各个步骤的工作由SearchHandler中组合的组件来完成了（可自定义，在该查询的requesthandler配置元素内配置）。示例，自定义组件组合：

```xml
<arr name="components">
 <str>query</str>
 <str>facet</str>
 <str>mlt</str>
 <str>highlight</str>
 <str>debug</str>
 <str>someothercomponent</str>
</arr>
```

还可在主组件组合前、后加入组件

```xml
<arr name="first-components">
    <str>mycomponent</str>
</arr>

<arr name="last-components">
    <str>myothercomponent</str>
</arr>

```



### 查询语法及解析器详解

#### 查询参数

##### defType

defType用来选择解析参数q指定的主查询串的查询解析器，如未给定默认使用solr的标准查询解析器（defType=lucene）

Solr中提供了三种解析器供选择：
lucene：   solr的Standard Query Parser  标准查询解析器
dismax：   DisMax Query Parser
edismax： Extended DisMax Query Parser (eDismax)

##### sort

指定如何对搜索结果进行排序，asc 表示升序，desc 降序。
Solr可根据如下部分对结果文档进行排序：

文档相关性得分
函数计算的结果
设置了docValues="true"的基本数据类型字段(numerics, string, boolean, dates, etc.) 
存储了docValues的可排序分词索引字段(SortableTextFields)。
单值不分词索引的字段。

对于基本数据类型和SortableTextFields ,如果是多值的，排序规则：
升序：取最小值参与排序；
降序：取最大值进行排序；

如要指定用什么值：则在传参时用sort=field(name,max) or sort=field(name,min) 方式传参

##### start

分页查询的起始行号（从0开始），没传默认为0。

##### rows

查询返回多少行，默认10（可配置）。

##### fq

Filter Query 用来在主查询的结果上进行过滤，不影响相关性评分。Fq对于提速复杂的查询非常有用。因为fq指定的过滤查询结果是独立于主查询被缓存起来的。对于下次查询，如果用到了该过滤查询，则直接从缓存中取出结果进行对主查询的结果进行过滤即可。

 fq的传参说明：

可以一次传传多个fq：
	fq=popularity:[10 TO *]&fq=section:0

也可将多个过滤条件组合在一个fq:
	fq=+popularity:[10 TO *] +section:0

 说明：几个fq就缓存几个过滤结果集

##### fl

fl(field list)，指定结果中返回哪些字段，指定的字段必须是 stored="true" or docValues="true" 的。多个字段用空格或英文逗号间隔。需要评分时通过 score 指定。如果传入的值为*，则stored="true" or docValues="true" and useDocValuesAsStored="true"的字段都会返回。

还可以在返回字段上用：

函数
fl=id,title,product(price,popularity)

文档转换器
fl=id,title,[explain]

别名
fl=id,price:div(price, 2)

##### debug

debug参数用于指定在结果中返回调试信息。支持的参数值如下：

debug=query
debug=timing
debug=results
debug=all

为了向后兼容老版本，可用 debugQuery=true，同 debug=all。

##### explainOther

在一个查询中附带解释另一个查询的评分，在结果中返回它的得分解释。
这可以让我们在topN查询时理解为什么某个文档没有返回。

示例：
q=supervillians&debugQuery=on&explainOther=id:juggernaut

##### timeAllowed

限定查询在多少毫秒内返回，如果到时间了还未执行完成，则直接返回部分结果。

##### omitHeader

true/false ，如设为true，在则响应体中忽略表示查询执行状态信息（如耗时）的头。

##### wt

指定响应的内容格式：json、xml、csv……  SearchHandler根据它选择ResponseWriter。

##### cache

设置是否对查询结果、过滤查询的结果进行缓存。默认是都会被缓存的。如果不需要缓存明确设置 cache=false。
设置部分缓存：q={!cache=false}name:薯片&fq=desc:薯片

##### logParamsList

solr默认会日志记录所有的请求参数，如果不需要记录所有，则通过此参数指定要记录的参数名，如：

logParamsList=q,fq

如果都不记录传入： logParamsList=

##### echoParams

指定在响应体的内容的头部中返回哪些查询参数，可选值：
explicit: 默认，返回显示传入的参数+
all: 应用到查询的所有参数.
none:不返回

#### 查询解析器

Standard Query Parser
DisMax Query Parser
Extended DisMax Query parser

默认使用的是 Standard Query Parser 。通过defType参数可指定。

##### Standard Query Parser

solr标准查询解析器。关键优点：它支持一个健壮且相当直观的语法，允许我们创建各种结构的查询。这个我们在学习lucene时已学过。最大的缺点：它不能容忍语法错误。

**Standard Query Parser  请求参数**

**q**：用标准查询语法定义的查询表达式（查询串、主查询），必需。
**q.op**：指定查询表达式的默认操作， “AND” or “OR”，覆盖默认配置值。
**df**：指定默认查询字段。标准解析器只能使用一个默认查询字段。
**sow**： Split on whitespace 按空格分割，如果设置为true，则会分别对分割出的文本进行分词处理。默认false。

##### 标准查询语法

**Term** 词项表示：
单个词项的表示：     电脑
短语的表示：	 "联想笔记本电脑"

**Field**字段
字段名:
示例： name:“联想笔记本电脑” AND type:电脑
如果name是默认字段，则可写成： “联想笔记本电脑” AND type:电脑
如果查询串是：type:电脑 计算机 手机
注意：**只有第一个是type的值，后两个则是使用默认字段。**

**统配符**
?  单个字符
\* 0 个或多个字符
示例：te?t    test*    te*t

**模糊查询**，词后加 ~
示例：     roam~
模糊查询最大支持两个不同字符。
示例：  roam~1

**正则表达式**
示例： /[mb]oat/

**临近查询**，短语后加 ~移动值
示例： "jakarta apache"~10

**范围查询**
mod_date:[20020101 TO 20030101]       包含边界值
title:{Aida TO Carmen}      不包含边界值

**词项加权**
使该词项的相关性更高，通过^数值来指定加权因子，默认加权因子值是1
如要搜索包含 jakarta apache 的文章，jakarta更相关，则：
jakarta^4 apache
短语也可以： "jakarta apache"^4 "Apache Lucene"

**固定分值**
^=score，通过此字句匹配的文档使用固定的分值。
(description:blue OR color:blue)**^=1.0** text:shoes

**Boolean操作符**
AND, +, OR, NOT ,-
布尔关键字必须全大写，如：AND OR NOT
DisMax query parser 仅支持 +  -

**组合()**
字句组合： (jakarta OR apache) AND website
字段组合： title:(+return +"pink panther")

**转义\\**
对语法字符： + - && || ! ( ) { } [ ] ^ “ ~ * ? : \ /     进行转义。

**注释/\*\*/**
"jakarta apache" /* this is a comment in the middle of a normal query string */ OR jakarta

##### Solr Standard Query Parser 对传统 lucene语法的增强

**在范围查询的边界两端都可以用***
field:[* TO 100] finds all field values less than or equal to 100
field:[100 TO *] finds all field values greater than or equal to 100
field:[* TO *] matches all documents with the field

**允许纯非的查询（限顶级字节）**
**-inStock:false**  -- finds all field values where inStock is not false
**-field:[* TO *]**   -- finds all documents without a value for field

**支持嵌入solr查询（子查询），切入查询可以使用任意的solr查询解析器**
inStock:true OR {!dismax qf='name manu' v='ipod'}

**支持特殊的filter(…) 语法**
说明某个字句的结果要作为过滤查询进行缓存
q=features:songs OR filter(inStock:true)
q=+manu:Apple +filter(inStock:true)
q=+manu:Apple & fq=inStock:true

**如果过滤查询中的某个字句需要独立进行过滤缓存，也可用。**
q=features:songs & fq=+filter(inStock:true) +filter(price:[* TO 100])
q=manu:Apple & fq=-filter(inStock:true) -filter(price:[* TO 100])

**范围查询** (“[a TO z]”), **前缀查询** (“a\*”), **统配符查询**(“a*b”)  **使用规定分值**。

**查询中的时间表示语法**
createdate:1976-03-06T23\:59\:59.999Z
createdate:"1976-03-06T23:59:59.999Z"
createdate:[1976-03-06T23:59:59.999Z TO *]
createdate:[1995-12-31T23:59:59.999Z TO 2007-03-06T00:00:00Z]
timestamp:[* TO NOW]
pubdate:[NOW-1YEAR/DAY TO NOW/DAY+1DAY]
createdate:[1976-03-06T23:59:59.999Z TO 1976-03-06T23:59:59.999Z+1YEAR]
createdate:[1976-03-06T23:59:59.999Z/YEAR TO 1976-03-06T23:59:59.999Z]

#### DisMax Query Parser

DisMax Query Parser 是设计用于处理用户输入的简单短语查询的，它的特点：
只支持查询语法的一个很小的子集：简单的短语查询、+  - 修饰符、AND OR 布尔操作； 
简单的语法，不抛出语法错误异常给用户。 
可以在多个字段上进行短语查询。
可以灵活设置各个查询字段的相关性权重。
可以灵活增加满足某特定查询文档的相关性权重。

DisMax：Maximum Disjunction  最大分离。
DisMax Query 定义：
	一个查询，可以为不同字段设置评分权重，在合并它的查询字句的命中文档时，
	每个文档的分值取各个字句中的最大得分值。

##### DisMax参数

**q**
指定主查询表达式。注意简单短语，不可使用通配符，+号会被当成或处理。

**q.alt**
q.alt 提供一个备选语句，当q没有指定或为空时，执行这个查询。

**qf**
qf指定要查询的字段及权重，多个字段用空格分割
qf="fieldOne^2.3 fieldTwo fieldThree^0.4"

**mm (Minimum Should Match)**
用来指定当q中包含多个字句、各字句是或操作时，最少需多少个字句匹配才算匹配。

**pf (Phrase Fields)**
pf用来设置当某个字段匹配所有查询字句的短语时，该字段的加权权重。
定义格式与qf相同：pf="fieldOne^2.3 fieldTwo fieldThree^0.4"

**ps (Phrase Slop)**
短语的移动因子。当查询给入多个词、短语时。对这些词进行短语匹配的移动因子。

**qs(Query Phrase Slop)**
用户给入的q 主查询中包含短语时，通过qs可指定短语的移动因子。

**tie (Tie Breaker)**
tie参数通常是一个小于1的浮点数，当查询命中多个field的时候，最终的score获得多少将由这个tie参数来进行调节。比如命中了field1，field2这2个field。

如果field1.score= 10，field2.score=3。那么 score = 10 + tie * 3.
也就是说，如果tie=1，最终的score就相当于多个字段得分总和;如果tie=0,那么最终的score就相当于是命中的field的最高分。
通常情况下呢，官方推荐tie=0.1。

**bq (Boost Query)**
bq指定一个加权查询，当主查询中命中的文档符合bq加权查询时，将获得更高的得分。

q=cheese
bq=date:[NOW/DAY-1YEAR TO NOW/DAY]AND ^5.0 
bq=date:[NOW/DAY-1YEAR TO NOW/DAY]^5.0
bq=date:[NOW/DAY-1YEAR TO NOW/DAY]^5.0

可以指定多个bq参数。

**bf (Boost Functions)**
bf用来定义加权函数，然后可在bq中使用加权函数
bf=recip(rord(creationDate),1,1000,1000)
...or...
bq={!func}recip(rord(creationDate),1,1000,1000)

#### Extended DisMax Query Parser

扩展 DisMax Query Parse 使支持更多的标准查询语法。也增加了不少参数。
实际中如有需要，请参考： http://lucene.apache.org/solr/guide/7_3/the-extended-dismax-query-parser.html

#### 函数

olr查询也可使用函数，可用来过滤文档、提高相关性值、根据函数计算结果进行排序、以及返回函数计算结果。在标准查询解析器、dismax、edismax中都可以使用函数。

**函数可以是**

常量：数值或字符串字面值，如 10、”lucene solr”
字段:    name  title
另一个函数：functionName(…)
替代参数：q={!func}min($f1,$f2)&f1=sqrt(popularity)&f2=1

**函数的使用方式**

用作**函数查询**，查询参数值是一个函数表达式，来计算相关性得分或过滤
q={!func}div(popularity,price)&fq={!frange l=1000}customer_ratings

在**排序**中使用
sort=div(popularity,price) desc, score desc

在**结果**中使用
&fl=sum(x, y),id,a,b,c,score&wt=xml

在**加权参数** bf、boost中使用来计算权重
q=dismax&bf="ord(popularity)^0.5 recip(rord(price),1,1000,1000)^0.3"

在**设置评分**计算函数的特殊关键字 _val_ 中使用
q=_val_:mynumericfield    _val_:"recip(rord(myfield),1,2,3)"

##### 数据转换函数

| **函数语法**                                     | **说明**                                                     |
| ------------------------------------------------ | ------------------------------------------------------------ |
| def(x,y)                                         | Default,若x存在，则返回x，否则返回y                          |
| field(fieldName)                                 | 取单值字段的值                                               |
| map(x,min,max,tartget)map(x,min,max,target,else) | 如果x落在在min、max之间，返回target，否则返回x 或 else（当指定了x时） |
| ms(time2,time1)ms(time1)ms()                     | 返回两个时间相隔的毫秒数time2-time1time2没有则为now返回当前时间的毫秒数 （1970/1/1） |
| ord(fieldName)                                   | 对于一个字段，它所有的值都将会按照字典顺序排列，这个函数返回你要查询的那个特定的值在这个顺序中的排名(从1开始)。这个字段，必须是非multiValued的，当没有值存在的时候，将返回0。示例：一个字段x包含的词项有("apple","banana","pear")，field(x) 包含apple的文档将返回1 |
| rord(fieldName)                                  | ord的逆序                                                    |
| scale(x,number1,number2)                         | 基于所有文档中x的最大值和最小值，介于number1与number2之间，对每个文档的x值进行缩放。 |
| top(x)                                           | 从顶层IndexReader取值                                        |

##### 数学函数

| **函数语法**           | **说明**                                                     |
| ---------------------- | ------------------------------------------------------------ |
| abs(x)                 | 取绝对值                                                     |
| acos(x)                | 取x的反余弦                                                  |
| asin(x)                | 取x的反正弦                                                  |
| atan(x)                |                                                              |
| atan2(x,y)             | 返回直角坐标(x,y)转换为极坐标的方位角                        |
| cbrt(x)                | x的立方根                                                    |
| ceil(x)                | 向上取整                                                     |
| cos(x)                 |                                                              |
| cosh(x)                | x的双曲线余弦值                                              |
| deg(x)                 | 将x的弧度转为度数                                            |
| div(x,y)               | x除以y                                                       |
| e()                    |                                                              |
| exp(x)                 | 以自然常数e为底的指数函数，即e的x次方                        |
| floor(x)               | 向下取整                                                     |
| hypo(x,y)              | sqrt(x2+y2) 返回直角三角形的斜边长                           |
| linear(m,x,b)          | f(x)=m*x + b 返回线性函数的值                                |
| ln(x)                  | x的自然对数                                                  |
| log(x)                 |                                                              |
| pi()                   |                                                              |
| pow(x,y)               | x的y次方                                                     |
| product(x,…n)mul(x,…n) | 求乘积                                                       |
| rad(x)                 | 转弧度                                                       |
| recip(x,m,a,b)         | a/(m*x + b) 互逆函数实现。其中，m、a、b是常量，x是变量或者一个函数。当a=b，并且x>=0的时候，这个函数的最大值是1，值的大小随着x的增大而减小。 |
| rint(x)                | 将x舍入为最近的整数                                          |
| sin(x)                 |                                                              |
| sinh(x)                | x的双曲正弦值                                                |
| sqrt(x)                | x的平方根                                                    |
| sub(x,y)               | x-y                                                          |
| sum(x,…n)add(x,…n)     |                                                              |
| tan(x)                 | 正切                                                         |
| tanh(x)                | 双曲正切                                                     |

##### 相关性函数

| **函数语法**                                     | **说明**                                                     |
| ------------------------------------------------ | ------------------------------------------------------------ |
| docfreq(fieldName,term)                          | fieldName字段包含词项term的文档数（文档频次）                |
| idf(fieldName,value)                             | 字段值的逆文档频次                                           |
| maxdoc()                                         | 索引中的文档数，包含尚未清理的删除文档                       |
| norm(fieldName)                                  | fieldName字段存储在索引中的范数                              |
| numdocs()                                        | 索引中的文档数，不包含尚未清理的删除文档                     |
| query(subquery,defaultScore)                     | 与subquery匹配的文档得分，defaultScores是不匹配subquery的文档得分 |
| sumtotaltermfreq(field)sttf(field)               | 索引中该字段被索引的分词数                                   |
| termfreq(fieldName,term)                         | 词项在文档的该字段中出现的频次                               |
| tf(fieldName,term)                               | 字段中词项的tf因子值                                         |
| totaltermfreq(fieldName,term)ttf(fieldName,term) | 词项在指定字段中的总词频（所有文档）                         |

##### 布尔函数

| **函数语法**               | **说明**                                                   |
| -------------------------- | ---------------------------------------------------------- |
| and(x,y)                   | x、y都为true，则为true                                     |
| exists(x)                  | 若x有值存储返回true                                        |
| if(x,trueValue,falseValue) | x有trueValue与falseValue两个值，若x为true，则返回trueValue |
| not(x)                     |                                                            |
| or(x,y)                    |                                                            |
| xor(x,y)                   |                                                            |

##### 距离函数

| **函数语法**                                            | **说明**                                                     |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| dist(power,x1,…n1,x2,…n2)                               | 根据power定义的距离度量，计算2个向量/点之间的距离。最常见的power值包括：0— 稀疏计算1— 曼哈顿距离2— 欧几里得距离无穷—无限范数（向量中的最大值） |
| sqedist(x1,…n1,x2,…n2)                                  | 相比dist(2,….) 这个函数消除了平方根计算，如果只是进行相关性排序或相关度调整，而不需要精确的距离计算，它更有效。 |
| hsin(radiusInRM,isDegrees,x1,y1,x2,y2)                  | 在球面上计算两点的距离                                       |
| geohash(lat,lon)                                        | 计算经纬度的geohash值，geohash是地理位置的特殊字段串编码。该函数可以为ghhsin函数提供输入参数 |
| Ghhsin(radiusInKM,geohash1,geohash2)                    | 对两个geohash值使用半正失函数                                |
| strdist(s1,s2,distType)strdist(s1,s2,”ngram”,ngramSize) | 计算两个字符串的字符相似度或间隔距离，介于0（不相似）与1（完全相同）之间。distTypede的可取值为：jw—jaro-winkler距离edit—编辑距离ngram—编辑距离的ngram版本 |
| geodist(sfield,lat,lon)geodist(sfield,pt)geodist()      | 返回地球上两点间的距离，一个通过空间字段sfield指定，另一个通过坐标指定。 |



