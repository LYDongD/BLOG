## shell相关

#### 子shell

* 一个子shell是一个子进程
* 子进程的变量无法被父进程读写

```

#利用()语法创建子shell
(a=1)
echo $a

#利用管道创建子shell
cat file.txt | while read line; do echo $line; done

```

