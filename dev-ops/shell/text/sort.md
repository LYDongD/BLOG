## sort

###功能

对文件和stdin进行排序操作,并将输出写入stdout

###用法

1 对文件内容进行排序和去重

```
//对文件内容进行排序并将stadout重定向到文件sort.txt
sort file1.txt file2.txt file3.txt > sort.txt

//对stdin去重并将stdout重定向到文件uniq_sort.txt
cat sort.txt | uniq > uniq_sort.txt

//排序并去重
sort -u file.txt
sort file.txt | uniq 

```

2 个性化排序

```

//逆序
sort -r file.txt

//按列进行排序
sort -k 2 file2.txt 

```

3 sort + uniq

```

//统计行出现的次数
sort file.txt | uniq -c

//找出重复的行
sort file.txt | uniq -d


```

4 统计字符串中字符出现的次数

```
#!/bin/bash

input="ahebhaaa"

#按字符切割成行，排序后计数，合并行
output=`echo $input | sed 's/[^\n]/&\n/g'|sed '/^$/d'|sort|uniq -c|tr -d '\n'`
echo $output

```
