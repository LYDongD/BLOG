## 权限控制

### chmod 修改文件对用户的权限

1 权限

* r -> 读 -> 4
* w -> 写 -> 2
* x -> 可执行 -> 1
* t -> 文件夹可读写但不可删除

```

chmod 641 test.txt

//赋予其他用户对文件夹test读写但不可删除的权限
chmod a+t test

```

2 递归

* -R

```

chmod ./ 777 -R

```

---

### chown 修改文件所有者

* pattern: user.group

```

//修改文件用户为liam，组为root
chown liam.root test.txt

```

