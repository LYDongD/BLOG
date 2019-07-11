## 归档 cpid

* -o 从stdin读取文件并输出，需要通过 > 重定向归档到指定文件
* -i 从stdout读入
* -t 列出文件列表
* -d 解归档
* -v 输出信息

```
#归档
ls file* | cpio -ov > archive.cpio 

#查看
cpio -it < archive.cpid

#解归档
cpio -id < archive.cpid

```
