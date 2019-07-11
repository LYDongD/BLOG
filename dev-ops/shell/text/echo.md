## 打印

### 命令

* echo
* prinf

区别：echo默认换行，printf不换行，prinf用于格式化打印

---

### 举例


#### echo

**1 打印转义字符**

echo -e 

```
//必须添加参数-e
echo -e '2\t3'

// 打印特点颜色的字体
// \e[1;31m 以红色方式打印，31为红色色码
// \e[0m 恢复颜色
echo -e "\e[1;31m this is red text \e[0m"

```

**2 打印变量**

```
//打印变量不能加单引号
echo $PATH

```

#### printf

**1 格式化打印**

```
#!/bin/bash

#%-5s: 左对齐并占5个字符，不足填空格(默认右对齐)
#%-4.2f: 左对齐并占4个字符，保留2位小数点(四舍五入)

printf "%-5s %-10s %-4s\n" NO Name Mark
printf "%-5s %-10s %-4.2f\n" 1 liam 85.245

```

**结果：**

```
NO    Name       Mark
1     liam       85.25

```


