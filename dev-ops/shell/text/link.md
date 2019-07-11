## 链接

> 软链接

为源文件生成链接文件，链接文件软链至源文件

生成软链接后，修改源文件，可使target同步发生变化

ln -s source_file target_file

```

#为脚本文件生成软链接文件
ln -s /Users/lee/script/fclogin.sh /usr/local/bin/fclogin

```

查看target文件属性：

```
lrwxr-xr-x  1 lee  admin  28  5  9 09:51 /usr/local/bin/fclogin -> /Users/lee/script/fclogin.sh

```




