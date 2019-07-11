## 归档

* -c 归档文件
* -f 指定归档目标文件名
* -v 输出详细信息
* -x 提取文件
* -t 查看归档文件列表
* -r 追加文件

```
#归档
tar -cvf test.tar *.txt

#提取, -C 指定提取到特定的目录
tar -xvf test.tar -C /tmp

#查看
tar -tvf test.tar

##合并, -A合并
tar -Avf test.tar test2.tar

#追加
tar -rvf test.tar test.txt

#删除, --delte
tar -f test.tar --delete file.txt

#排除, --exclude
tar -cf test.tar * --exclude *.txt

```
