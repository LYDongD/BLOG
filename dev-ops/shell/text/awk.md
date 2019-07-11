## awk

* 逐行处理文本，并可以提取指定字段
* 灵活强大的语法扩展


#### 基本模式

* 执行BEGIN语句块
* 遍历读取文件行/stadin,模板匹配执行语句块
* 执行END语句块

```
awk 'BEGIN{...}pattern{actiont}END{...}' file

```

#### 特殊变量

* NR：number of records 行号
* NF：number of fields 字段数
* $0: 当前行的文本内容
* $1: 第一个字段文本内容

#### pattern

* 行选择：NR < 5 或 NR==1, NR==4
* 正则表达式: /linux/, !/linux/

#### 范例

```
//print打印变量并拼接结果
echo | awk '{var1="v1"; var2="v2"; print var1"-"var2;}'

//累加
seq 5 | awk 'BEGIN{sum=0; print "SUMMARY:"} \
{print $1"+"; sum+=$1} END{print "=="; print sum}'

//打印行数
awk 'END{print NR}' file

//逆序打印
seq 5 | awk '{lifo[NR]=$0; lno=NR} END{ for(;lno>-1;lno--) {print lifo[lno];}}'

//接收外部变量
var1='v1' var2='v2'
echo | awk '{print v1, v2}' v1=$var1 v2=$var2

//主动读取行getline
seq 5 | awk 'BEGIN{getline; print"READ ahead:"$0}{print $0}'

//getline读取命令输出到cmdout变量
echo | awk '{ "grep root /etc/passwd" | getline cmdout; print cmdout}'

//-F指定定界符为:
awk -F: '{print $NF}' /etc/passwd

```

* 行范围打印

```
//打印5-10row
seq 100 | awk 'NR==5,NR==10'

//print pattern range row
cat section.txt | awk '/pa.*3/, /end/'

```
