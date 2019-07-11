## 比较文件的异同

### comm - 逐行比较两个已排序的文件

假设比较两个文件a.txt和b.txt

* 第1列为a - b
* 第2列为b - a
* 第3列为a ∩ b

每次比较都会输出3列，可以通过参数-1,-2,-3屏蔽指定列

```

//输出两个文件相同的部分
comm -12 a.txt b.txt

//输出两个文件不同的部分
comm -3 a.txt b.txt

```

### diff - 比较文件差异

* 正常模式：输出不包含上下文
* 上下文模式: diff -c 输出包含上下文，相同部分不会合并
* 合并模式: diff -u 输出包含上下文且会合并相同部分

git diff 采用的是合并模式

参考[diff](http://www.ruanyifeng.com/blog/2012/08/how_to_read_diff.html)
