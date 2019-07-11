## rsync备份工具

* -a 以归档方式备份
* -z 备份时压缩处理
* --exclude 排除相关文件
* --delete 删除相关文件


```
#备份到远程主机
rsync -avz script root@120.78.142.82:/tmp/

```
