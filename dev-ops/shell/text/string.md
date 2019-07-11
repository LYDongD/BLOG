## 文本处理

### 替换

```

//{var/origin/replace
var="this is a line of text"
echo ${var/line/replaced}

//sed
echo $var | sed 's/line/replaced/g'

```

### 字符串切片

* ${str:startIndex:length}

```
echo ${str:0:4}

```
