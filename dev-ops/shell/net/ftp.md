## 文件传输

### ftp服务器

> 1 安装ftp服务器，ftp服务监听21端口

```
yum -y install vsftpd
lsof -i:21

```

> 2 修改配置，为特定用户分配权限

```
cd /etc/vsftpd
vim ftpusers //取消用户限制
vim vsftpd.conf //set userlist_deny=NO

```

> 3 设置ftp默认上传目录

* vim /etc/vsftpd.conf

```
local_root=/home/ftp/pub
chroot_local_user=YES
anon_root=/home/ftp/pub
allow_writeable_chroot=YES

```

* mkdir /home/ftp/pub
* chmod 755 /home/ftp/pub

> 4 启动服务

```
service vsftpd start/restart

```

> 5 使用lftp/ftp交互模式下连接服务并实现文件传输

* lftp username@host 登录
* cd 切换远程主机目录
* lcd 切换本地主机目录
* get file 下载
* put file 上传

> 6 使用ftp脚本实现文件上传下载

* -i 关闭交互模式
* << EOF data EOF 即时文件重定向输入

```
#!/bin/bash
host='120.78.142.82'
user='root'
passwd='Pss123er'

ftp -i -n $host << EOF

#设置ftp用户名和密码
user ${user} ${passwd}
#ftp文件传输类型
binary

#切换远程主机目录
cd /tmp
#切换本地主机目录
lcd ./
#切换交互提示
prompt
#上传文件
put test2.txt

#退出
quit

EOF

```


### sftp

利用sftp模拟ftp文件传输，该命令依赖ssh，默认端口22

* -oPort 指定ssh端口
* get file 下载
* put file 上传


### rsync

* 参考rsyn(../compress/rsync.md)


### scp

* scp source username@host:/tmp/test
* scp username@host:/tmp/test ./
