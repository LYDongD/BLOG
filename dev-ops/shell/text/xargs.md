## xargs

### 功能

将标准输入(stdin)转换为命令行参数，是单行命令的重要组件

### 用法

1 文本单行和多行之间的转换

```

//多行转单行
cat test.txt | xargs

//单行转多行，指定每行的字符数量
cat test.txt | xargs -n 3

//-d使用特定字符进行分割并指定每行的数量
cat "aXbXcXdXeXfXhX" | xargs -d X -n 3

```

2 结合find执行

```

//查找md文件并统计行数
find ./ -name '*.md' -print0| xargs -0 wc -l

```
