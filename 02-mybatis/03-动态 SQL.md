## 动态SQL

### 1、if

### 2、choose, when, otherwise

```xml
<choose>
    <when test=""></when>
    <when test=""></when>
    <when test=""></when>
  	<otherwise></otherwise>
</choose>
```

### 3、where

where标签会去除掉第一个多出来的and或or

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG 
  <where> 
    <if test="state != null">
         state = #{state}
    </if> 
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

### 4、trim

`prefix`: 添加前缀

`prefixOverrides`： 前缀覆盖，去掉字符串前面多余的字符

`suffix`：添加后缀

`suffixOverrides`：后缀覆盖，去掉字符串后面多余的字符

```xml
<trim prefix="" 
      prefixOverrides="" 
      suffix="" suffixOverrides="">
</trim>
```

### 5、foreach

```xml
<foreach collection="list" index="i" item="item" open="(" close=")" separator=",">
</foreach>
```

### 6、bind

`bind` 元素可以从 OGNL 表达式中创建一个变量并将其绑定到上下文。比如：

```xml
<select id="selectBlogsLike" resultType="Blog">
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
  SELECT * FROM BLOG
  WHERE title LIKE #{pattern}
</select>
```

### 7、内置参数

`_databaseId`

`_parameter`

