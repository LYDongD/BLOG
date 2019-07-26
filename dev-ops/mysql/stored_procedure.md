## 一段存储过程分析

### 查看存储过程

oracle:

```
//查看列表
select * from user_procedures;

//查看内容
select * from USER_SOURCE where type = 'PROCEDURE' and name = 'GET_SEND_CODE'; 


```

### 存储过程分析 

oracle: 生成8位寄件码

```
procedure GET_SEND_CODE
(
  notuse in varchar2,
   send_code out varchar2
)
as
  v_code varchar2(20);
  v_count number;
begin
  while send_code is null LOOP
    begin
     select to_char(lpad(trunc(dbms_random.value(1, 100000000)), 8, '0')) into v_code from dual;
      select count(*) into v_count from xxx where SEND_CODE = v_code;
      if v_count = 0 then
      	insert into DEMO_SEND_CODE values (terminal_table_id.nextval, v_code, sysdate);
       send_code := v_code;
      end if;
    end;
  end LOOP;
  commit;
end GET_SEND_CODE;

```

核心片段： select to_char(lpad(trunc(dbms_random.value(1, 100000000)), 8, '0')) from dual;

* dbms_random.value(a,b) 生成[a,b]之间的随机数，包括浮点数
* trunc截断小数
* lpad(a, n, pat) 如果a不足n位，从左边补pat; 否则截断
* to_char 转成字符串


### mybatis实现存储过程调用

* mapper statement 的 statementType为CALLABLE
* 注意入参和出参模式，出参即返回结果

```
<parameterMap type="java.util.Map" id="getSendCodeMap">
 	<parameter property="notuse" mode="IN" jdbcType="INTEGER"/>
    <parameter property="sendCode" mode="OUT" jdbcType="VARCHAR"/>
</parameterMap>

<select id="getSendCode" parameterMap="getSendCodeMap" statementType="CALLABLE">
    CALL GET_SEND_CODE(?,?)
</select>


```
