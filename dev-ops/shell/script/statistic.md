## 分组统计脚本

### 方法执行时间耗时统计

> 模式：

2019-08-13 10:26:54.446 [pool-15-thread-18] INFO com.fcbox.service.function.BoxUsedRateExecutor.doBus (BoxUsedRateExecutor.java:181) [TxId :  , SpanId : ] doBus:edcode=FC5750840,count=183,cost=313

> 涉及语法

* 关联数组(字典)
* sed 模式提取
* grep 过滤
* 父子shell
* 循环判断

> 脚本

```

#!/bin/bash

LOG_FILE=/app/applogs/edmssvr/svrs.log
SYNC_FILE=/tmp/tmpSync.log

#fork sub shell to grep data
(grep "doBus:edcode" $LOG_FILE > $SYNC_FILE)

declare -A cost_dic
count=0
max=0
min=100000

#遍历行，提取耗时并分组统计，保存到字典
while read line
do
    line=${line}
    cost=$(echo ${line} | sed "s/.*cost=\([0-9]*\).*/\1/g" | bc)
    if test $[cost] -gt 2000; then
        let cost_dic[">2000ms"]+=1
    elif test $[cost] -gt 1000; then
        let cost_dic["1000-2000ms"]+=1
    elif test $[cost] -gt 500; then
        let cost_dic["500-1000ms"]+=1
    elif test $[cost] -gt 400; then
        let cost_dic["400-500ms"]+=1
    elif test $[cost] -gt 300; then
        let cost_dic["300-400ms"]+=1
    elif test $[cost] -gt 200; then
        let cost_dic["200-300ms"]+=1
    elif test $[cost] -gt 100; then
        let cost_dic["100-200ms"]+=1
    elif test $[cost] -gt 0; then
        let cost_dic["<100ms"]+=1
    fi

    if test $[cost] -gt $[max]; then
        let max=$cost
    elif test $[cost] -lt $[min]; then
        let min=$cost
    fi

    let ++count
done < $SYNC_FILE

#打印字典
echo 总数：$count 最大耗时: $max ms 最小耗时: $min ms
echo 分布 数量 占比
for key in ${!cost_dic[*]}; do
    percent=$(echo "scale=2;${cost_dic[$key]}*100/$count" | bc)
    echo $key ${cost_dic[$key]} $percent%
done

#删除临时文件
rm $SYNC_FILE

```

### 单词分组统计排序

文本：

```
the day is sunny the the
the sunny is is

``` 

分组排序：

```
cat words.txt | tr -s ' ' '\n' | sort | uniq -c | sort -r -k 1 | awk '{ print $2, $1 }'

```

* tr -s A B 将A替换成B， 这里将空格替换成回车
* sort 根据单词排序，这样把重复的单词排在了一起
* uniq -c 去重并计算重复的数量
* sort -r -k 1 按照第一列数量逆序排序
* awk '{print $2 $1}' 格式化打印第二列和第一列

