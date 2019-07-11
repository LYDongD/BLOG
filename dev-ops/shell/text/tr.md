## tr

## 功能

从标准输入中替换、缩减和/或删除字符，并将结果写到标准输出。

### 用法

#### 1 替换

```
//< 将stdin从键盘输入重定向到文件，即从文件中读取内容到stdin
//tr从stadin中替换内容并写入标准输出
tr 1 0 < test.txt

```

#### 2 删除

```

//管道连接stdout和stdin, tr接受stdin并删除0-3范文内的字符
cat test.txt | tr -d '0-3'

//删除tab和回车换行
cat test.txt | tr -d '\n\t

//删除空格
cat test.txt | tr -s ' '

```
