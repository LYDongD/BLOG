## 设备相关

#### 向登录服务的用法发送消息

* /dev/pts 所有登录用户的字符设备，通过ls -l 可查看登录用户
* /dev/stdin 包含传递给当前进程的标准输入数据

```
user=$1
devices=`ls /dev/pts/* -l | awk '{print $3,$10}' | grep $USER | awk '{print $2}'`

for device in $devices;
do
    cat /dev/stdin > $device
done

```
