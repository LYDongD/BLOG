## mybatis 使用攻略~~~~

### 参考文档

[mybatis3中文文档](http://www.mybatis.org/mybatis-3/zh/)

### 动态sql

1 foreach 元素

* 不仅可以遍历数组，还可以遍历字典等可迭代对象

```
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="value" index="key" collection="map"
      open="(" separator="," close=")">
        #{value}
  </foreach>
</select>

```

2 bind元素

* bind可以绑定一个变量，并用OGNL表达式实现变量值，可增强映射语句的可读性

模糊查询中，绑定一个变量pattern表示匹配模式：

```
<select id="selectBlogsLike" resultType="Blog">
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
  SELECT * FROM BLOG
  WHERE title LIKE #{pattern}
</select>

```



