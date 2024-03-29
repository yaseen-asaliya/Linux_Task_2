# Linux_Task_2
* Performance Statistics
> Shell script `system_performance_statistics.sh` to performance some statistics and stored them in file based on item type and date collected
```
#!/bin/bash

CURRENT_TIME=$(date "+%Y.%m.%d-%H.%M.%S")

echo "$(df -h)" | awk '{print $0}' > /root/Task_two/disk_usage_files/"Disk_usage--$CURRENT_TIME".txt
echo "$(free -m)" | awk '{print $0}' > /root/Task_two/memory_usage_files/"Memory_usage--$CURRENT_TIME".txt
echo "$(mpstat)" | awk '{print $0}' > /root/Task_two/CPU_utlization_files/"CPU_utlization--$CURRENT_TIME".txt
```
* Calculate Statistics Avarage
> Shell script `calculate_avgs.sh` to compute avgs usage for each of cpu,disks,and memory and stored the result in files
```
#!/bin/bash

# Cacluate avg for used and free disks

cat disk_usage_files/*.txt > all_disk_usage_data.txt

awk '{
        sum_used[$1]+=$3;
        sum_free[$1]+=$4
        count[$1]+=1
} END {
        for(user in sum_used)
                print user,"\t",sum_used[user]/count[user],"\t",sum_free[user]/count[user]

}' all_disk_usage_data.txt | awk '/\/dev\/*/ {print $0}' > disk_usage.txt

rm all_disk_usage_data.txt


# Calculate avg for used and free memory

cat memory_usage_files/*.txt > all_memory_usage_data.txt

awk '{
        sum_used[$1]+=$3
        sum_free[$1]+=$4
        count[$1]+=1
} END {
        for(user in sum_used)
                print user,"\t",sum_used[user]/count[user],"\t",sum_free[user]/count[user]

}' all_memory_usage_data.txt | awk '!/total*/  {print $0}' > memory_usage.txt

rm all_memory_usage_data.txt


# Calculate avg for CPU usage

cat CPU_utlization_files/*.txt > all_CPU_utlization_data.txt

# To filter requier rows only
awk '{if($3=="all") print $0}' all_CPU_utlization_data.txt | awk '{print $0}' > filterd_CPU_utlization_data.txt


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
}' filterd_CPU_utlization_data.txt | awk '{print $0}' > CPU_usage.txt

rm all_CPU_utlization_data.txt
rm filterd_CPU_utlization_data.txt
```
* Install and enable apache server
```
# systemctl enable httpd
# systemctl start httpd
# systemctl status httpd      # to check the status of apache
```
* Set data collected and avarages in web page
> shell script `set_data_in_html.sh` to set all data collected and avags in separated html pages
```
#!/bin/bash


set_avgs_in_html(){
   echo "$(cat $1)" | awk -F'\t' -v title=$2 '
     BEGIN{
        printf "<table>"
        print title
     }
     {
        print "<tr>"
        for(i=1;i<=NF;i++)
            print "<td>" $i "</td>"
         print "</tr>"
     }
     END{
        print "</table>"
     }
'
}




set_all_data_collected_in_html(){
   echo "$(cat $1)" | grep -v "^$2" | grep "^$3" | awk -v time=$4 '
   {
     print "<tr>"
     printf "<td>"
     printf time
     printf "</td>"
     for(i=1;i<=NF;i++)
       print "<td>" $i "</td>"
     print "</tr>"
   }
'
}





create_sub_header(){
   arr=("$@")

   for str in ${arr[@]}; do
     echo -n "<th>$str</th>"
   done

}





########################## Disk Usage ######################################
echo "<html><body>" > /var/www/html/htmlfiles/disk.html
echo "<h3>Disks And Partitations Avarage Usage</h3>" >> /var/www/html/htmlfiles/disk.html

DISK_HEADER_DATA=("Disk" "Used" "Free")
DISK_HEADER="$(create_sub_header "${DISK_HEADER_DATA[@]}")"

set_avgs_in_html "disk_usage.txt" $DISK_HEADER >> /var/www/html/htmlfiles/disk.html

echo "<h3>Files contains data collected each hour</h3><table>" >> /var/www/html/htmlfiles/disk.html

DISKS_HEADER_DATA=("Timestamp" "Filesystem" "Size" "Used" "Avail" "Use%" "Mounted-on")
DISKS_HEADER="$(create_sub_header "${DISKS_HEADER_DATA[@]}")"
echo $DISKS_HEADER >> /var/www/html/htmlfiles/disk.html

for dir in disk_usage_files/*
do
    TEMP="${dir##*--}"
    TIME=$(echo $TEMP | rev | cut -c8- | rev)
    set_all_data_collected_in_html $dir "File*" "/dev/" $TIME >> /var/www/html/htmlfiles/disk.html
done

echo "</table></body></html>" >> /var/www/html/htmlfiles/disk.html

##############################################################################


########################## Memory Usage ###############################
echo "<html><body>" > /var/www/html/htmlfiles/memory.html
echo "<h3>Memory Avarage Usage</h3>" >> /var/www/html/htmlfiles/memory.html

MEMORY_HEADER_DATA=("Memory" "Used" "Free")
MEMORY_HEADER="$(create_sub_header "${DISK_HEADER_DATA[@]}")"


set_avgs_in_html "memory_usage.txt" $MEMORY_HEADER >> /var/www/html/htmlfiles/memory.html

echo "<h3>Files contains data collected each hour</h3><table>" >> /var/www/html/htmlfiles/memory.html

DATA_HEADER=("Timestamp" "Type" "Total(M)" "Used(M)" "Free(M)" "Shared(M)" "Buff/cache(M)" "Available(M)")
MEMORY_HEADER="$(create_sub_header "${DATA_HEADER[@]}")"
echo $MEMORY_HEADER >> /var/www/html/htmlfiles/memory.html


for dir in memory_usage_files/*
do
    TEMP="${dir##*--}"
    TIME=$(echo $TEMP | rev | cut -c8- | rev)
    set_all_data_collected_in_html $dir " " "" $TIME >> /var/www/html/htmlfiles/memory.html
done

echo "</body></html>" >> /var/www/html/htmlfiles/memory.html
#######################################################################


########################## CPU Utlization ###############################
echo "<html><body>" > /var/www/html/htmlfiles/cpu.html
echo "<h3>CPU Utlization Avarage</h3>" >> /var/www/html/htmlfiles/cpu.html

CPU_HEADER_DATA=("CPU" "%usr" "%nice" "%sys" "%iowait" "%irq" "%soft" "%steal" "%guest" "%gnice" "%idle")
CPU_HEADER="$(create_sub_header "${CPU_HEADER_DATA[@]}")"

set_avgs_in_html "CPU_usage.txt" $CPU_HEADER >> /var/www/html/htmlfiles/cpu.html

echo "<h3>Files contains data collected each hour</h3><table>" >> /var/www/html/htmlfiles/cpu.html
CPU_HEADER_DATA=("Timestamp" "CPU" "%usr" "%nice" "%sys" "%iowait" "%irq" "%soft" "%steal" "%guest" "%gnice" "%idle")
CPU_HEADER="$(create_sub_header "${CPU_HEADER_DATA[@]}")"
echo $CPU_HEADER >> /var/www/html/htmlfiles/cpu.html

for dir in CPU_utlization_files/*
do
    TEMP="${dir##*--}"
    TIME=$(echo $TEMP | rev | cut -c5- | rev)

    echo $(cat $dir | grep -v "^Linux*" | awk 'END{print}' |awk '{for (i = 3; i <= 13;i++) print $i}') > tmp

    set_all_data_collected_in_html "tmp" $tem "Linux*" "" $TIME >> /var/www/html/htmlfiles/cpu.html
done
echo "$(rm tmp)"

echo "</body></html>" >> /var/www/html/htmlfiles/cpu.html
#######################################################################
```

* Create Shell to run task
> shell script `execution.sh` to run all previous shell commands
```
#!/bin/bash

chmod a+x system_performance_statistics.sh
./system_performance_statistics.sh

chmod a+x calculate_avgs.sh
./calculate_avgs.sh

chmod a+x set_data_in_html.sh
./set_data_in_html.sh

```
* Set cron job
> simply we set cronjob to execute `execution.sh` each hour per day by use `crontab -e` command
```
0 * * * * /root/Task_two/execution.sh
```
