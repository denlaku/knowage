## XML映射文件

### 1、select

常用属性

`id`: 在命名空间中唯一的标识符，可以被用来引用这条语句。

`resultType`: 返回的期望类型的类的完全限定名或别名。如果是集合，那应该是集合可以包含的类型，而不能是集合本身。resultType 和resultMap不能同时使用。

`statementType`: STATEMENT，PREPARED 或 CALLABLE 的一个，默认值PREPARED。

### 2、insert, update 和 delete

insert获取自增主键

```xml
<insert id="insert" useGeneratedKeys="true" keyProperty="">
</insert>
```

如果是oracle：

```xml
<!--BEFORE-->
<insert id="insert">
  	<!--
	keyProperty: 查出的主键封装给哪个属性
	order="BEFORE"： 主键查询SQL在插入SQL之前执行
	      "AFTER"： 主键查询SQL在插入SQL之后执行
	-->
    <selectKey keyProperty="id" order="BEFORE">
        select seq.nextval from dual
    </selectKey>
</insert>

<!--AFTER-->
<insert id="insert">
  	<!--
	keyProperty: 查出的主键封装给哪个属性
	order="BEFORE"： 主键查询SQL在插入SQL之前执行
	      "AFTER"： 主键查询SQL在插入SQL之后执行
	-->
    <selectKey keyProperty="id" order="AFTER">
        select seq.currval from dual
    </selectKey>
</insert>
```

### 3、sql

抽取可重用的SQL片段, 与include标签配合使用

`include ` 除了可以引用sql片段外，还可以自定义属性

```xml
<include refid="">
	<property name="testColumn" value="cbd"/>
</include>
```

include定义的属性只能通过${testColumn}的方式进行引用

### 4、参数（Parameters）

`单个参数`: 基本类型，对象类型，MyBatis可直接使用这个参数，不需要经过任何处理。

​	**集合类型**

​	mybatis会把参数封装成map，key为collection；对于List类型 collection 和 list 都是有效的key

​	**数组**

​	mybatis会把参数封装成map，key为array

`多个参数`: 任意多个参数，都会被MyBatis重新包装成一个Map传入。

​		Map的key是param1，param2，0，1…，值就是参数的值。

`命名参数`: 使用@Param为参数起个名字

### 5、结果集（Result Maps）

1、关联查询

```xml
<resultMap type="emp" id="EmpAndDept">
    <result column="id" property="id" />
    <result column="firstName" property="firstName" />
    <result column="lastName" property="lastName" />
    <result column="email" property="email" />
    <result column="gender" property="gender" />
    <association property="dept" javaType="dept>
        <result column="deptId" property="id"/>
        <result column="deptName" property="name"/>
    </association>
    <collection property="cars" ofType="car">
        <result column="carId" property="id"/>
        <result column="carName" property="name"/>
    </collection>
</resultMap>
```
2、分步查询

```xml
<resultMap type="emp" id="EmpAndDept">
    <result column="id" property="id" />
    <result column="firstName" property="firstName" />
    <result column="lastName" property="lastName" />
    <result column="email" property="email" />
    <result column="gender" property="gender" />
    <association property="dept" column="{id=dept_id}" 
                 select="com.denlaku.mybatis.dao.DeptDao.getDeptById">
    </association>
    <collection property="cars" column="id" select="com.denlaku.mybatis.dao.CarDao.findCarListByEmpId">
    </collection>
</resultMap>
```

分步查询如果需要传递多个参数，可以用`column="{id=dept_id}" ` 的形式

分步查询开启懒加载，修改全局配置文件settings

```xml
<settings>
    <setting name="lazyLoadingEnabled" value="true" />
    <setting name="aggressiveLazyLoading" value="false" />
</settings>
```



3、鉴别器 （discriminator ）

```xml
<resultMap type="emp" id="EmpAndDept">
    <result column="id" property="id" />
    <result column="firstName" property="firstName" />
    <result column="lastName" property="lastName" />
    <result column="email" property="email" />
    <result column="gender" property="gender" />
    <discriminator javaType="string" column="gender">
        <case value="F" resultType="emp">
            <association property="dept" column="dept_id" 
                select="com.denlaku.mybatis.dao.DeptDao.getDeptById">
            </association>
        </case>
    </discriminator>
</resultMap>
```

