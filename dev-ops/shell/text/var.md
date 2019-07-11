## 变量

### 环境变量

环境变量是可以从父进程继承而来的共享变量

例如：

* PATH, 二进制文件的寻址路径
* HOME, 用户目录
* PWD, 当前路径

#### 应用

**1 查看进程的环境变量**

```
//直接查看环境变量
echo $HOME

//查看进程号
pgrep java

//换行打印所有环境变量
cat /proc/28503/environ | tr '\0' '\n' 

```
**输出：**

```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
PWD=/usr/local/aegis/aegis_quartz
LANG=en_US.UTF-8
SHLVL=3
LANGUAGE=en_US:
_=/usr/local/cloudmonitor/wrapper/bin/./wrapper
WRAPPER_INIT_DIR=/usr/local/aegis/aegis_quartz
WRAPPER_BIN_DIR=/usr/local/cloudmonitor/wrapper/bin
WRAPPER_WORKING_DIR=/usr/local/cloudmonitor/wrapper/bin
WRAPPER_CONF_DIR=/usr/local/cloudmonitor/wrapper/conf
WRAPPER_LANG=en
WRAPPER_PID=28496
WRAPPER_BITS=64
WRAPPER_ARCH=x86
WRAPPER_OS=linux
WRAPPER_HOSTNAME=iZwz9amxfnp7ntp8ayaf5lZ
WRAPPER_HOST_NAME=iZwz9amxfnp7ntp8ayaf5lZ
WRAPPER_FILE_SEPARATOR=/
WRAPPER_PATH_SEPARATOR=:
JAVA_HOME=../../jre

```

**2 搜索环境变量**

```
//从所有环境变量中搜索
env | grep USER

//根据=切割(var=value)并输出指定value
env | grep USER | cut -d "=" -f 2

```

结果：

```
[root@iZwz9amxfnp7ntp8ayaf5lZ ~]# env | grep USER | cut -d "=" -f 2
root

```


**3 添加环境变量**

* var = $var:new
* export var;

```

//为环境变量PATH，添加新的寻找路径
export PATH="$PATH:/home/user/bin"

```
