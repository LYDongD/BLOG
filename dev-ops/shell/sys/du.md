## 磁盘使用情况

#### du

disk usage

* -h 自动添加单位
* -s summary统计
* -c total 统计
* -a 递归文件

```
#查看目录大小
du -sh dir

#查看文件大小
du -sh file

#递归查看目录下所有文件大小
du -ah dir

#查找当前目录下所有文件，计算大小后，按照第一列逆序排序
find . -type f | xargs du -k | sort -nrk 1 | head

#查看当前目录下各个目录的大小，并进行排序
ls -l | grep dr | awk '{print $9}' | xargs du -sh | sort -rk 1

```

#### df

disk free

```
#查看各个磁盘使用情况
df -h

```
