## ps 进程

* ps -eo输出进程特定形象
* comm 输出命令
* pcpu 输出进程cpu占用比例

```

#1min内，每1s收集一次数据并统计
SEC=10
UNIT_TIME=1

STEPS=$(($SEC/$UNIT_TIME))

echo watching cpu usage...;

for((i=0;i<STEPS;i++))
do
    ps -eo comm,pcpu | tail -n +2 >> /tmp/cpu_usage.$$
    sleep $UNIT_TIME
done


cat /tmp/cpu_usage.$$ | \
awk '
{process[$1]+=$2;}
END{
for(i in process){
    printf("%-20s%s\n", i, process[i]);
}
}
' | sort -nrk 2 | head


rm /tmp/cpu_usage.$$

```

查看cpu/mem/包含线程数 排名前10的进程

```
ps -eo comm,pcpu --sort -pcpu | head

ps -eo comm,pid,pcpu,pmem --sort -pmem | head

#按线程数排序
ps -eLf --sort nlwp | head

```
