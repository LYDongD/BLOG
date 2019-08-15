## 行转列


### 问题

给定一个文件，包含多行数据，每行是由以空格为分隔符的单词组成

要求按列打印（行转列)

例如：

```
name age
alice 21
ryan 30

```

脚本处理好，输出为：

```
name alice ryan
age 21 30

```


### 解法1

使用双层嵌套循环，外层按列遍历，内层按行遍历即可

遍历行时，拼接字符串，遍历完成后打印结果即为当前列结果

```
#!/bin/bash
col=$(head -n1 file.txt | awk '{print NF}')
for((i=1;i<=$col;i++))
do
    while read line; do
        tmp=$(echo $line | awk '{print $v1}' v1="$i")
        str=${str}" $tmp"
    done < file.txt

    echo $str
    str=""
done

```

### 解法2 

使用awk逐行处理，用数组保存每列的结果

第一行：初始化数组，每列占位一个元素

其他行: 根据列取出指定元素进行拼接

所有行处理完成后，每个元素都保存一个完整列，最后遍历数组打印即可

```
awk '
{
    for (i = 1; i <= NF; i++) {
        if(NR == 1) {
            s[i] = $i;
        } else {
            s[i] = s[i] " " $i;
        }
    }
}
END {
    for (i = 1; s[i] != ""; i++) {
        print s[i];
    }
}' file.txt


```
