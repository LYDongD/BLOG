## 调试

### 打印所有执行过程

```
//检查语法错误但不打印，类似于nginx -t
sh -n debug.sh

//打印每一行执行脚本
sh -x debug.sh

```

### 框选打印区域

set -x打开调试，+x关闭调试

```

//仅打印第3、4行
for i in {1..6}
set -x
echo $i
set +x
done

```

补充：set命令可打印所有本地变量

```
//查找PATH变量
set | grep PAT

```
 
