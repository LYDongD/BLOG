## bzip

####  bzip/bunzip

* -k 压缩时保留源文件
* -1~9 指定压缩等级
* -c 输出到stdout

```
#压缩与解压
bzip2 test.txt
bunzip2 test.txt.bz2

#保留源文件
bzip2 test.txt -k


#接收stdin压缩输出到stdout, 最后重定向到文件
cat test.txt | gzip2 -c > test.tar.bz2

```

### 使用tar进行归档压缩

* -j 使用bzip2进行压缩或解压

```
#归档压缩
tar -czvf archive.tar.gz *.txt

#归档解压
tar -xzvf archive.tar.gz -C tmp/

```
