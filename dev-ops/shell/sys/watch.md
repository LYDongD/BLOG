## watch 监视

* -n 时间间隔s
* -d 高亮变化的区域

```
#每隔2s输出一次uptime并高亮线上变化
watch -d -n 2 uptime

```

### inotifywait监视目录或文件操作

在指定路径path下的操作都会被监视，并将监视记录保存到特点文件

```
#下载inotify-tool
yum install -y inotify-tools.x86_64

#监视脚本
path=$1
inotifywait -m -r -e create,move,delete $path -q > watch.log

#后台方式启动脚本
sh watchdir.sh /tmp &

```

### 监视磁盘使用情况

* 运算符
    * -e 文件运算符，判断文件是否存在
    * -n 字符串运算符，判断字符串是否为非空
* 时间
    * date %+D 按照指定格式输出
    * date %-s 输出秒
* 临时文件
    * $$, 进程号
* 循环读取文件行
    * while read line; do ... don < xxx.log
* 将一段输出文本重定向
    * (xxx) >> xx.log
* 格式化打印
    * printf 和 printf()

```
log_file="diskusage.log"

if [[ -n $1 ]]; then
    log_file=$1
fi

if [ ! -e $log_file ]; then
    printf "%-14s%-14s%-12s%-12s%-12s%-12s%-12s%-12s\n" "Date" "IP Address" "Device" "Capacity" "Used" "Free" "Percent" "Status" > $log_file
fi

ip_list="127.0.0.1"

(for ip in $ip_list;

do
    df -H | grep ^/dev/ > /tmp/$$.df
    while read line;
    do
        cur_date=$(date +%D)
        printf "%-14s%-14s" $cur_date $ip
        echo $line | awk '{printf("%-12s%-12s%-12s%-12s%-12s", $1, $2, $3, $4, $5)}'
        pusg=$(echo $line | awk '{print $5}' | sed 's/%//g') 
        if [ $pusg -lt 80 ] 
        then
            echo SAFE
        else 
            echo ALERT
        fi
    done < /tmp/$$.df
done
) >> $log_file

```
