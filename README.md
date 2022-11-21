# Linux_Task_2
> system_performance_statistics.sh
```
#!/bin/bash

CURRENT_TIME=$(date "+%Y.%m.%d-%H.%M.%S")

echo "$(df -h)" | awk '{print $0}' > /root/Task_two/disk_usage_files/"Disk_usage--$CURRENT_TIME".txt
echo "$(free)" | awk '{print $0}' > /root/Task_two/memory_usage_files/"Memory_usage--$CURRENT_TIME".txt
echo "$(mpstat)" | awk '{print $0}' > /root/Task_two/CPU_utlization_files/"CPU_utlization--$CURRENT_TIME".txt

```

> calculate_avgs.sh
```
#!/bin/bash

# Cacluate avg for used and free disks

echo "$(cat disk_usage_files/*.txt)" > all_disk_usage_data.txt

awk '{
        sum_used[$1]+=$3;
        sum_free[$1]+=$4
        count[$1]+=1
} END {
        for(user in sum_used)
                print user,"\t",sum_used[user]/count[user],"\t",sum_free[user]/count[user]

}' all_disk_usage_data.txt | awk 'BEGIN{print "\tDisk\t\tused\tfree"} /\/dev\/*/ {print $0}' > disk_usage.txt

echo "$(rm all_disk_usage_data.txt)"






# Calculate avg for used and free memory

echo "$(cat memory_usage_files/*.txt)" > all_memory_usage_data.txt

awk '{
        sum_used[$1]+=$3
        sum_free[$1]+=$4
        count[$1]+=1
} END {
        for(user in sum_used)
                print user,"\t",sum_used[user]/count[user],"\t",sum_free[user]/count[user]

}' all_memory_usage_data.txt | awk 'BEGIN{print "Memory\tused\tfree"} !/total*/  {print $0}' > memory_usage.txt

echo "$(rm all_memory_usage_data.txt)"


# Calculate avg for CPU usage

echo "$(cat CPU_utlization_files/*.txt)" > all_CPU_utlization_data.txt
echo "$(tail -n 1 all_CPU_utlization_data.txt)"  > all_CPU_utlization_data.txt


awk '{
        sum_usr[$3]+=$4
        sum_nis[$3]+=$5
        sum_sys[$3]+=$6
        sum_iowait[$3]+=$7
        sum_irq[$3]+=$8
        sum_soft[$3]+=$9
        sum_steal[$3]+=$10
        sum_guest[$3]+=$11
        sum_gnic[$3]+=$12
        sum_idle[$3]+=$13
        count[$3]+=1
} END {
        for(user in sum_usr) {
                printf user "\t"
                printf sum_usr[user]/count[user] "\t"
                printf sum_nis[user]/count[user] "\t"
                printf sum_sys[user]/count[user] "\t"
                printf sum_iowait[user]/count[user] "\t"
                printf sum_irq[user]/count[user] "\t"
                printf sum_soft[user]/count[user] "\t"
                printf sum_steal[user]/count[user] "\t"
                printf sum_guest[user]/count[user] "\t"
                printf sum_gnic[user]/count[user] "\t"
                printf sum_idle[user]/count[user] "\t"


}
}' all_CPU_utlization_data.txt | awk 'BEGIN{print "CPU\t%usr\t%nice\t%sys\t%iowait\t%irq\t%soft\t%steal\t%guest\t%gnice\t%idle"} {print $0}' > CPU_usage.txt

echo "$(rm all_CPU_utlization_data.txt)"


```
