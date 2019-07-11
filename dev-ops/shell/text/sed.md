## sed

sed 使用正则表达式匹配文本并进行替换删除等操作


### 查找满足条件的文件并替换模式

* -r 递归查找
* -l 仅打印满足条件的文件
* -i 替换后更新文件

```

#mac
grep "package main" -rl array/*.go | xargs sed -i '_bak' "s/package main/packgae array/g"

#linux
grep "package main" -rl array/*.go | xargs sed -i '_bak' "s/package main/packgae array/g"

```


### 全局匹配替换

sed -i 's/pattern/replace_string/g' file

* -i 替换后更新文件
* -g 替换所有满足条件的文本


### 移除空白行

sed '/^$/d' file

* /^$/ 匹配空白行
* d 删除

#### 子串匹配标记

* & 代表匹配到的子串

```
#匹配所有单词并添加方括号
echo this is an example | sed 's/\w\+/[&]/g'

```

* \1,\2 代表匹配到的第k组子串

#### 引用变量

* sed 表达式使用双引号可在表达式内引用变量

```
text=hello
echo hello world | sed "s/$text/HELLO/g"

```

#### 文件压缩

```
//压缩：替换空格，回车，制表符，注释等
cat sample.js | tr -d '\n\t' | tr -s ' '| sed 's:/\*.*\*/::g' | sed 's/\?\([{}();,:]\)\?/\1/g'

//解压：增加换行
cat obfuscated.txt | sed 's/;/;\n/g' | sed 's/{/{\n\n/g' | sed 's/}/\n\n}/g'

```

#### 模式提取

* -n 取消自动打印模式
* -e 以脚本方式执行
* s/../../p 替换并打印结果

```
#* master -> master, 其中\(.*\) -> \1 为模式提取
git branch | sed -n -e 's/^\*\(.*\)/\1/p'

```

#### 删除包含某个单词的句子

```
// [^.]* 匹配除了句号之外的所有字符, 整体是匹配一句话，包含mobile phone
sed "s/[^.]*mobile phone[^.]*\.//g" word.txt

```
