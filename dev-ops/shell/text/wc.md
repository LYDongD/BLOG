## 统计

### wc 

* wc -l 统计行
* wc -w 统计单词
* wc -c 统计字符

```
#统计代码行, 过滤空行
find . -type f -name “*.java” |xargs cat|grep -v ^$|wc -l

```
