## zip

zip/unzip一定会保留原文件

* -r 递归目录进行归档压缩
* -u 更新文件
* -d 删除文件
* -l 查看文件

```
#归档压缩
zip archive.zip test*.txt

#查看
zip -l archive.zip

##删除
zip archive.zip -d test2.txt

## 解压
unzip archive.zip

```
