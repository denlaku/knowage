## XML配置

configuration 配置

### 1、属性（properties）

这些属性都是可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递。

properties标签有两个属性：

uri: 引用磁盘或者网络中的文件

resource：引用类路径下的文件

```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

然后其中的属性就可以在整个配置文件中被用来替换需要动态配置的属性值。比如:

```xml
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
```

如果属性在不只一个地方进行了配置，那么 MyBatis 将按照下面的顺序来加载：

- 在 properties 元素体内指定的属性首先被读取。
- 然后根据 properties 元素中的 resource 属性读取类路径下属性文件或根据 url 属性指定的路径读取属性文件，并覆盖已读取的同名属性。
- 最后读取作为方法参数传递的属性，并覆盖已读取的同名属性

### 2、设置（settings）

MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

### 3、类型别名（typeAliases）

为 Java 类型设置一个短的名字。

```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>
```

也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如:

```xml
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。若有注解，则别名为其注解值。

```java
@Alias("author")
public class Author {
}
```

@Alias 可以避免别名重复从而导致冲突。例如：domain.blog包下有一个Author类，domain.blog包的子包下还有一个Author类，这种场景就会出现别名冲突。

### 4、类型处理器(typeHandlers)

无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时， 都会用类型处理器将获取的值以合适的方式转换成 Java 类型。

### 5、对象工厂（objectFactory）



### 6、插件（plugins）

MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

- Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
- ParameterHandler (getParameterObject, setParameters)
- ResultSetHandler (handleResultSets, handleOutputParameters)
- StatementHandler (prepare, parameterize, batch, update, query)

### 7、配置环境（environments）

**transactionManager**: 事务管理器

```xml
<environments default="development">
  <environment id="development">
      <transactionManager type="JDBC" />
      <dataSource type="POOLED">
          <property name="driver" value="${db.driverClassName}" />
          <property name="url" value="${db.url}" />
          <property name="username" value="${db.username}" />
          <property name="password" value="${db.password}" />
      </dataSource>
  </environment>
</environments>
```

### 8、databaseIdProvider

支持多数据库厂商。

MyBatis 会加载不带 `databaseId` 属性和带有匹配当前数据库 `databaseId` 属性的所有语句。 

如果同时找到带有 `databaseId` 和不带 `databaseId` 的相同语句，则后者会被舍弃。

```xml
<databaseIdProvider type="DB_VENDOR">
    <property name="MySQL" value="mysql"/>
</databaseIdProvider>
```

### 9、映射器(mappers)

mappers用来注册mapper文件

mapper标签的三个属性：

`resource`： 加载类路径下的mapper文件

`uri`：加载磁盘或者网络中的mapper文件

`class`：引用接口，这里需要分两种情况

- 有sql映射文件，映射文件名必须和接口同名
- 没有sql映射文件，所有sql都是用@Select @Insert等注解写在接口方法上 （不推荐）

相对于类路径的资源引用：

```xml
<mappers>
    <mapper resource="com/denlaku/mybatis/dao/EmployeeDao.xml" />
</mappers>
```

映射器接口实现类的完全限定类名，有对应的sql映射文件

```xml
<mappers>
    <mapper class="com.denlaku.mybatis.dao.EmployeeDao" />
</mappers>
```

映射器接口实现类的完全限定类名，没有sql映射文件，所有sql都是用@Select @Insert等注解写在接口方法上 （**不推荐**）

```xml
<mappers>
    <mapper class="com.denlaku.mybatis.dao.EmployeeDao" />
</mappers>
```

```java
public interface EmployeeDao {
	@Select("select id, email, gender from t_employee where id = #{id, jdbcType=INTEGER}")
	Employee getEmpById(Integer id);
}
```

将包内的映射器接口实现全部注册为映射器

```xml
<mappers>
  <package name="com.denlaku.mybatis.dao"/>
</mappers>
```





























































