### 历史

> 通过~/.bash_history查看历史命令

0 grep 过滤无效行
1 awk逐行处理，提取命令
2 用关联数组(hash)计数
3 sort根据第二列逆序排列
4 head输出前10

```

#打印前10的命令
cat ~/.bash_history | grep -v "#.*" | awk \
'{
list[$1]++;
}END{
for (i in list)
    printf("%s\t%d\n", i, list[i])
}' | sort -nrk 2 | head

``` 

> 设置历史输出格式并携带时间

vim ~/.bashrc
source ~/.bashrc

```
HISTTIMEFORMAT="%Y-%m-%d:%H-%M-%S:`whoami`:"
export HISTTIMEFORMAT

#其他
HISTFILESIZE=2000      #设置保存历史命令的文件大小
HISTSIZE=2000             #保存历史命令条数

```

> 查看当前用户，创建的终端等

* w 打印终端，终端tty会与一个设备文件相关连，通过tty可查看文件路径
* tyy 查看终端关联的文件路径，通常是/dev/xxx
* users 查看当前系统登录的所有用户
* last 查看历史登录用户


