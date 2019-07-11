## find 查找

### 功能

遍历文件并执行相应的操作

### 用法

#### 1 查看目录的内容

```
//查看当前目录下的文件(递归)
find .

//查看/shell目录下的文件(递归), 文件换行打印
find /shell 

```

#### 2 根据条件搜索

```

//根据文件名搜索
find ./ -name "*.md"

//根据路径搜索
find ./ -path "*shell*"

//根据文件类型搜索, d->目录， f->文件
find ./ -type d

//根据文件时间进行搜索
//找出过去十分钟内被修改的文件
find . -type f -mmin -10
```

//根据文件大小进行搜索
find ~/ -size +100M

//根据用户进行查找
find . -type f -user lee

```

#### 3 查找并执行

```

//-exec xxx {} \; 对查找得到的文件执行特定的命令
//这里也可以用管道和xargs实现
find ./ -type f -name "*.txt" -exec rm {} \;


```
