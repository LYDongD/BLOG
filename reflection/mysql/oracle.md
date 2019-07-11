## oracle 常用SQL

### 查看元信息

1 查看表注释

```
select * 
from user_tab_comments 
where Table_Name='xxx';

```

2 查看列注释

```
select * 
from user_col_comments 
where Table_Name='xxx'
order by column_name;

```

3 查看表的所有列定义

```
select * 
from user_tab_columns 
where Table_Name='xxx' 
order by column_name;

```

4 查看oracle序列

```
select * from user_sequences;

```

### 插入

1 使用中间表完成批量插入(中间表：select xxx dual + union)

```
insert into xxx t
  (o_id, o_name, addtime)
  ((select 1, 'ykp', sysdate from dual) union
   (select 2, 'ykp', sysdate from dual) union
   (select 3, 'ykp', sysdate from dual) union
   (select 4, 'ykp', sysdate from dual));

```

对应mybatis mapper statement 的写法

```
<insert id="addBoxUsedClientPost" parameterType="java.util.List">
      INSERT INTO xxx(
       UUID,
       EDID,
       USED_TM,
       BOX_COUNT,
       BIG_BOX_COUNT,
       MIDDLE_BOX_COUNT,
       SMALL_BOX_COUNT,
       POST_BOX_ID,
       CABINET_ID,
       CABINET_TYPE_POSTION
) 
<foreach collection="list" item="item" index="index" separator="union all">
	select 
	 #{item.uuid},
     #{item.edId},
     TO_DATE(#{item.usedTime},'yyyy-mm-dd hh24:mi:ss'),
     #{item.boxCount},
     #{item.bigBoxCount},
     #{item.middleBoxCount},
     #{item.smallBoxCount},
     #{item.postBoxId},
     #{item.cabinetId},
     #{item.cabinetTypePostion}
	from dual
</foreach>
  </insert>

```

### 查看记录 

1 限制查询行数

使用rownum过滤条件

```
select * from xxx where rownum < 5;

```

2 查看某个字段重复的行

查看xxx中EDID重复的记录

```
select *from xxx where EDID IN (SELECT EDID FROM xxx GROUP BY EDID HAVING count(*) > 1);

```
