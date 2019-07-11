## 搜索

### grep

* 可进行单个或多个文件搜索
* 支持正则表达式
* 支持递归
* 支持静默输出


#### 举例

```

//正则表达式
//egrep 或 grep -E
egrep "[a-z]+" file
grep -E "[0-9]" file


//打印匹配行前后指定行
seq 10 | grep 5 -C 3
seq 10 | grep 5 -A 3 //后面
seq 10 | grep 5 -B 3 //前面

//判断某个搜索单词是否在文件中
grep -q "word" file
echo $? //如果存在，返回0

//递归和文件过滤
grep "word" ./ -r
grep "word" ./ -r -exclude-dir next //排除文件夹
grep "word" ./ -r -include *.txt //通配包含文件

//其他
//-i : 忽略大小写
//-e : 匹配多个样式
//-c : 统计匹配行数
//-o : 按单词匹配，匹配结果换行显示(仅显示pattern)

grep -i "WORD" file
grep -e "word" -e "hello" file
grep -c "word" file
echo "1 2 3 4\nhello\n5 6" | egrep -o "[0-9]" | wc -l //统计数字的个数

```

统计词频

```

#!/bin/bash
  
if [ $# -ne 1 ]; then
    echo "Usage: $0 filename"
    exit -1
fi

filename=$1

#use array to save each word's count
#awk 'pattern{action}', naturally iterate each line

egrep -o "\b[a-zA-z]+\b" $filename | \
awk '{count[$0]++}
END{printf("%-14s%s\n", "Word", "Count")
for(ind in count)
{
    printf("%-14s%d\n", ind, count[ind]);
}
}'


```

显示搜索词颜色

```

使用grep --color, 可设置别名：

vim ~/.source_rc
alias grep=‘grep  - -color’
source ~/.source_rc

```


