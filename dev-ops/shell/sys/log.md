##日志管理

#### logrotate

结合crontab，周期性执行logrotate，读取日志轮替配置，对日志进行轮替归档处理

```
#配置轮替
vim /etc/logrotate.d/test

#配置如下
/tmp/log/test.log {
    missingok
    daily
    notifempty
    size 1k
    rotate 5
    create 0600 root root
}

#写数据
dd if=/dev/zero of=/tmp/log/test.log bs=100k count=1000

#强制执行一次
logrotate --debug --verbose --force /etc/logrotate.d/test


```

查看/etc/cron.daily/logrotate, 可以看到logrotate执行的脚本

```
/usr/sbin/logrotate -s /var/lib/logrotate/logrotate.status /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0

```

#### syslog

syslogd 根据日志标记将日志记录在/var/log/xx.log 指定文件下

* 配置在/etc/rsyslog.conf 

```
#使用logger打印日志
logger "test"

#日志被记录在/var/log/message
tail -n 1 /var/log/message

```

查看进程被杀记录

```
#递归查找所有日志，根据关键字过滤
grep -i -r 'killed process' /var/log

```

解析/var/log/secure, 统计入侵者行为

```
#default log path
auth_log=/var/log/secure

#get path from param
if [[ -n $1 ]]; then
    auth_log=$1
fi


#invalid line
#Invalid user admin from 27.34.2.159 port 56141
#input_userauth_request: invalid user pony [preauth]
#Failed password for invalid user admin from 182.160.124.26 port 50405 ssh2

#use log to save valid user record
log=/tmp/valid.$$log
grep -v "invalid" $auth_log > $log

#Jan 22 16:18:16 izwz9amxfnp7ntp8ayaf5lz sshd[6776]: Failed password for liam from 61.141.158.189 port 58042 ssh2
# Failed password for liam from 61.141.158.189 port 58042 ssh2
# get users, $NF is last column num, -5 move to user column
users=$(grep -i "failed password" $log | awk '{print $(NF-5)}' | sort | uniq)

#prinf title line, 6 titles from order to time range
printf "%-5s|%-10s|%-10s|%-33s|%s\n" "Sr#" "User" "Attempts" "IP address" "Time Range"

ucont=0

#get iplist
ip_list="$(egrep -o "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" $log | sort | uniq)"

#use tmp file to calculate: ip -> user -> time -> limit -> print
for ip in $ip_list;
do
    #for specific ip, get all lines and save
    grep $ip $log > /tmp/temp.$$.log

    #for specific user, get all lines in this ip and save
    for user in $users;
    do
        grep $user /tmp/temp.$$.log > /tmp/$$.log
        #get time for this user
        cut -c -16 /tmp/$$.log > $$.time

        #get start and end time
        tstart=$(head -1 $$.time)
        start=$(date -d "$tstart" "+%s")
        tend=$(tail -1 $$.time)
        end=$(date -d "$tend" "+%s")

        #limit time is 5s, if try multi times, even exced limited time, count it
        limit=$(($end-$start))
        if [ $limit -gt 5 ]; then
            let ucount++
            ip=$(egrep -o "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" /tmp/$$.log | head -1)
            time_range="$tstart --> $tend"
            attempts=$(cat /tmp/$$.log | wc -l)
            printf "%-5s|%-10s|%-10s|%-33s|%s\n" "$ucount" "$user" "$attempts" "$ip" "$time_range"
        fi
    done
done


#delete tmp file
rm /tmp/valid.$$.log /tmp/$$.log /tmp/temp.$$.log $$.time 2> /dev/null

```

#### 统计不同用户的累计登陆时间


* 将指定格式的时间转成时间戳: date -d "xxx" +%s
* 删除文本的指定字符: tr -d 'xxxx'
* 计算 $(echo "xxxx" | bc) 或 let a=xxx

```
#!/bin/bash
log=/var/log/wtmp

if [[ -n $1 ]]; then
    log=$1
fi

printf "%-4s %-10s %-10s %-6s %-8s\n" "Rank" "User" "Start" "Logins" "Usage Hours"

last -f $log | head -n -2 > /tmp/ulog.$$

#以空格分隔，提取第一列
cat /tmp/ulog.$$ | cut -d ' ' -f1 | sort | uniq >> /tmp/users.$$

(while read user;
do
    grep ^$user /tmp/ulog.$$ > /tmp/user.$$
    cat /tmp/user.$$ | awk '{print $NF}' | tr -d ')(' > /tmp/userTime.$$
    seconds=0
    while read t;
    do
        s=$(date -d $t +%s 2>/dev/null)
        let seconds=seconds+s
    done < /tmp/userTime.$$
    firstlog=$(tail -n 1 /tmp/user.$$ | awk '{print $5,$6}')
    nlogins=$(cat /tmp/user.$$ | wc -l)
    baseSeconds=$(date -d "00:00" +%s)
    let relSeconds=seconds-baseSeconds
    hours=$(echo "$relSeconds/3600.0" | bc)
    printf "%-10s %-10s %-6s %-8s\n" $user $firstlog $nlogins $hours
done < /tmp/users.$$) | sort -nrk4 | awk '{printf("%-4s %s\n",NR,$0)}'

rm /tmp/user*.$$ /tmp/ulog.$$

```
