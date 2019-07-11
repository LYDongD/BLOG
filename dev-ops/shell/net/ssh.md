## ssh 秘钥登录

1 生成一对秘钥

```
ssh-keygen

```

2 将公钥保存到~/.ssh/authorized_keys

```
cd ~/.ssh
cat id_rsa.pub >> authorized_keys

```

3 设置文件权限

```
chmod 600 authorized_keys
chmod 700 ~/.ssh

```

4 修改sshd配置

/etc/ssh/sshd_config

```
RSAAuthentication yes
PubkeyAuthentication yes
PermitRootLogin yes

#禁用密码，可选
PasswordAuthentication no

```

5 重启sshd

```
service sshd restart

```

6 使用私钥登录

```
ssh -i id_rsa root@host

或

ssh-add id_rsa
ssh root@host

```


#### ssh执行远程命令

通过ssh可直接在本地执行远程命令而不必通过交互的方式

```
#将管道的stdin提交给远程的cat并重定向到test.txt文件
echo "text" | ssh root@120.78.142.82 'cat >> test.txt'

```

#### sshfs挂载远程目录

将远程目录挂载到本地

```
#mac下载
brew cask install osxfuse
brew install sshfs

#挂载远程目录到本地
sshfs root@120.78.142.82:/home/liam ~/mnt/liam

#取消挂载
umount ~/mnt/liam

```
