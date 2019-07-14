## 结构化数据导入DIH

### Data import Handler (DIH)

在solrconfig.xml 中定义DIH

```xml
<requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
  <lst name="defaults">
    <str name="config">/path/to/my/DIHconfigfile.xml</str>
  </lst>
</requestHandler>
```

#### DIH核心概念

Datasource       数据源
Entity	         实体：数据库中的表、视图
Processor	         处理器：执行从实体中加载数据
Transformer     转换器：对处理器加载的数据进行转换处理

### 从关系数据库导入

#### dataSource 数据源

单数据源

```xml
<dataSource type="JdbcDataSource" driver="com.mysql.jdbc.Driver" 
            url="jdbc:mysql://localhost/dbname" user="db_username" password="db_password"/>
```

多数据源

```xml
<dataSource type="JdbcDataSource" name="ds-1" driver="com.mysql.jdbc.Driver" 
            url="jdbc:mysql://db1-host/dbname" user="db_username" password="db_password"/>
<dataSource type="JdbcDataSource" name="ds-2" driver="com.mysql.jdbc.Driver" 
            url="jdbc:mysql://db2-host/dbname" user="db_username" password="db_password"/>
```

name、type 是通用属性，type默认是jdbcDataSource
其他属性是非固定的，不同type可有不同的属性，可随意扩展。

#### 配置document对应的实体

<document> 下可包含一个或多个 <entity>数据实体

##### entity 数据实体通用属性

name (required) : 标识实体的唯一名
processor : 当数据源是非RDBMS 时，必须指定处理器。(默认是SqlEntityProcessor)
transformer : 要应用在该实体上的转换器。
 dataSource : 当配置了多个数据源时，指定使用的数据源的名字。
pk : 实体的主键列名。只有在增量导入时才需要指定主键列名。和模式中的唯一键是两个不同的东西。
rootEntity : 默认document元素的子entity是rootEntity，如果把rootEntity属性设为false值，则它的子会被作为rootEntity（依次类推）。rootEntity返回的每一行会创建一个document。
onError : (abort|skip|continue) . 当处理entity的行为document的过程中发生异常该如何处理：默认是 abort，放弃导入。skip：跳过这个文档，continue：继续索引该文档。
preImportDeleteQuery : 在全量导入前，如需要进行索引清理cleanup，可以通过此属性指定一个清理的索引删除查询，否则用的是‘*:*’（删除所有）。只有<document>的直接子Entity设置此属性有效。
postImportDeleteQuery : 指定全量导入后需要进行索引清理的delete查询。只有<document>的直接子Entity设置此属性有效.

##### SqlEntityProcessor 的 entity 属性

query (required) : 从数据库中加载实体数据用的SQL语句。
parentDeltaQuery ： 指定增量关联父实体的pk的查询SQL。
deletedPkQuery ：仅用于增量导入，被删除实体的pk查询SQL。
deltaQuery : 仅用于增量导入，指定增量数据pk的查询SQL。
deltaImportQuery : (仅用于增量导入) .指定增量导入实体数据的查询SQL。如果没有指定该查询语句，solr将使用query属性指定的语句，经修改后来查询加载增量数据（这很容易出错）。在该语句中往往需要引用deltaQuery查询结果的列值，通过 ${dih.delta.<column-name>} 来引用，如：select * from tbl where id=${dih.delta.id}

solrconfig.xml文件中添加

```xml
<lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-dataimporthandler-\d.*\.jar" />
  
<requestHandler name="/dataimport" class="solr.DataImportHandler">
    <lst name="defaults">
        <str name="config">db-data-config.xml</str>
    </lst>
</requestHandler>
```

db-data-config.xml文件

```xml
<dataConfig>
    <dataSource driver="com.mysql.jdbc.Driver" url="jdbc:mysql://10.10.10.2:3306/solr?characterEncoding=utf8&amp;useSSL=false&amp;serverTimezone=UTC" user="root" password="denlaku" />
    <document>
		<entity name="product" query="select * from t_product"
			deltaQuery="select id from t_product where id=${dataimporter.request.id}"
			deltaImportQuery="select * from t_product where id>${dih.delta.id}">
			<field column="id" name="id" />
			<field column="name" name="name" />
			<field column="desc" name="desc" />
			<field column="price" name="price" />
			<field column="cats" name="cat" />
		</entity>
    </document>
</dataConfig>
```

**full-import :** 
请求启动全量导入：http://<host>:<port>/solr/dataimport?command=full-import
     返回导入正在进行中的状态信息，导入会在一个新线程中开启（可能会需要一定时间完成导入）。导入完成后，导入的开始时间将存入到conf/dataimport.properties 文件中，用于后面的增量导入。增量导入完成后也会存入增量开始的时间到这个文件，用于下一次增量导入。
	全量导入期间并不会阻塞solr查询。

可附加的参数有：
        entity : 指定导入<document> 下的哪些实体（必须是直接子）. 如果没有给定该参数，则是其下所有子entity。
        clean : (default 'true') 指定是否在导入前清理索引
        commit : (default ‘true’) 指定导入后是否提交
        optimize : (default ‘true’ up to Solr 3.6, ‘false’ afterwards). 是否进行优化。
        debug : (default ‘false’). 是否以调试模式运行，开发下使用。
            调试模式下不会提交，除非明确指定commit=true. 



**delta-import** 
 请求启动增量导入http://<host>:<port>/solr/dataimport?command=delta-import . 
支持的附加参数： clean, commit, optimize and debug 同 full-import。
status : 导入是异步进行的，通过下面的请求获得导入的进度、结果状态信息：http://<host>:<port>/solr/dataimport.
reload-config : 请求重新加载dih配置文件：http://<host>:<port>/solr/dataimport?command=reload-config .
abort : 请求中止当前的操作：http://<host>:<port>/solr/dataimport?command=abort 

**增量导入中的特殊变量${dataimporter.last_index_time}**

这个变量是上次导入的开始时间。默认存储在conf/请求处理器名.properties文件中。我们可以在<dataConfig>下配置一个propertyWriter元素来设置它。

```xml
<propertyWriter dateFormat="yyyy-MM-dd HH:mm:ss" type="SimplePropertiesWriter" 
                directory="data" filename="my_dih.properties" locale="en_US" />
```

**DIH配置文件中使用请求参数**

如果你的DIH配置文件中需要使用请求时传人的参数，可用${dataimporter.request.paramname}表示引用请求参数。

```xml
<dataSource driver="org.hsqldb.jdbcDriver" url="${dataimporter.request.jdbcurl}" 
	user="${dataimporter.request.jdbcuser}" password="${dataimporter.request.jdbcpassword}" />
```

