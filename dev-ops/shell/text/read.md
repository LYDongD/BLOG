## read

### 定义

从stdin读取数据


### 用法

从终端读取密码并禁止回显

**方法1 使用stty**

```

//用stty控制echo回显机制
stty -echo
read -p 'Eneter Password:' pwd
stty echo
echo
echo 'done'

```

**方法2 使用read参数**

```

read -p 'Enter Password' -s pwd
echo
echo 'done'
echo $pwd

```
