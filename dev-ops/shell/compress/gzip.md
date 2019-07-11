## gzip 压缩

#### gzip只能对一个文件进行压缩，对于多个文件需要先进行归档再压缩

* -l 查看压缩文件压缩比
* -1 ~ -9 指定压缩比
* --fast/--best 指定压缩特性

```
#压缩
gzip test.txt
#指定最大的压缩比
gzip -9 test.txt

#查看
gzip -l test.txt.gz

#解压
gunzip test.txt.gz

```

#### 使用tar归档快速进行压缩

* -z 归档后进行压缩
* -vv 打印详细信息
* -c 创建归档文件
* -x 提取文件列表
* -C 提取到指定目录

```
#归档并压缩
tar -czvvf test.tar.gz test*.txt

#复原
tar -xzvvf test.tar.gz -C tmp/


```
