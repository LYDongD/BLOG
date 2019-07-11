## 批量重命名

### 使用循环和mv命令

```
#!/bin/bash

count=1;
for old in *.txt
do

#文件切割语法，参考%和#的使用
new=rename-$count.${old##*.}

#错误输出fd=2 重定向到黑洞
mv "$old" "$new" 2> /dev/null

#通过返回值判断命令执行情况
if [ $? -eq 0 ];
then

echo "rename $old to $new"
let count++

fi

done


```
