# Linux 查看硬件资源使用

## 内存使用查看

* free -h (查看当前机器的内存使用状况）

* 查看每个用户内存使用

```
#!/bin/bash

stats=””
echo "%   user"
echo "============"

# collect the data
for user in `ps aux | grep -v COMMAND | awk '{print $1}' | sort -u`
do
  stats="$stats\n`ps aux | egrep ^$user | awk 'BEGIN{total=0}; \
    {total += $4};END{print total,$1}'`"
done

# sort data numerically (largest first)
echo -e $stats | grep -v ^$ | sort -rn | head
```

* 根据 pid 查看内存使用

```
# 使用 top 命令
top -p pid

# 查看进程状态, VmRSS 是内存信息
cat /proc/pid/status

# cat 命令输出
<!--[huairuo@host02 ~]$ cat /proc/51362/status-->
<!--Name:	fdfs_storaged-->
<!--Umask:	0000-->
<!--State:	S (sleeping)-->
<!--Tgid:	51362-->
<!--Ngid:	51362-->
<!--Pid:	51362-->
<!--PPid:	1-->
<!--TracerPid:	0-->
<!--Uid:	0	0	0	0-->
<!--Gid:	0	0	0	0-->
<!--FDSize:	64-->
<!--Groups:	0 48 -->
<!--VmPeak:	   84308 kB-->
<!--VmSize:	   84308 kB-->
<!--VmLck:	       0 kB-->
<!--VmPin:	       0 kB-->
<!--VmHWM:	   67144 kB-->
<!--VmRSS:	   67144 kB-->
<!--RssAnon:	   66148 kB-->
<!--RssFile:	     996 kB-->
<!--RssShmem:	       0 kB-->
<!--VmData:	   70236 kB-->
<!--VmStk:	     132 kB-->
<!--VmExe:	     320 kB-->
<!--VmLib:	    3316 kB-->
<!--VmPTE:	     180 kB-->
<!--VmSwap:	       0 kB-->
<!--Threads:	9-->
<!--SigQ:	0/1030542-->
<!--SigPnd:	0000000000000000-->
<!--ShdPnd:	0000000000000000-->
<!--SigBlk:	0000000000000000-->
<!--SigIgn:	0000000000001000-->
<!--SigCgt:	0000000180004a07-->
<!--CapInh:	0000000000000000-->
<!--CapPrm:	0000001fffffffff-->
<!--CapEff:	0000001fffffffff-->
<!--CapBnd:	0000001fffffffff-->
<!--CapAmb:	0000000000000000-->
<!--NoNewPrivs:	0-->
<!--Seccomp:	0-->
<!--Speculation_Store_Bypass:	thread vulnerable-->
<!--Cpus_allowed:	ffffff,ffffffff-->
<!--Cpus_allowed_list:	0-55-->
<!--Mems_allowed:	00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000003-->
<!--Mems_allowed_list:	0-1-->
<!--voluntary_ctxt_switches:	14902-->
<!--nonvoluntary_ctxt_switches:	12-->


```
 
 
## 查看磁盘 I/O

iotop, 安装命令 `yum install iotop`

```
iotop
<!--Total DISK READ :	0.00 B/s | Total DISK WRITE :      17.65 K/s-->
<!--Actual DISK READ:	0.00 B/s | Actual DISK WRITE:       0.00 B/s-->
  <!--TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND                                                                     -->
<!--28928 be/4 ll          0.00 B/s   17.65 K/s  0.00 %  0.00 % python -u /home/ll/.pycharm_helpers/pyd~12 --file /home/ll/Github/QANet/main.py-->
    <!--1 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % systemd --switched-root --system --deserialize 22-->
    <!--2 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kthreadd]-->
    <!--4 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kworker/0:0H]-->
    <!--5 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kworker/u128:0]-->
    <!--6 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ksoftirqd/0]-->
    <!--7 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [migration/0]-->
    <!--8 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [rcu_bh]-->
    <!--9 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [rcu_sched]-->
   <!--10 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [lru-add-drain]-->
   <!--11 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [watchdog/0]-->
   <!--12 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [watchdog/1]-->
   <!--13 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [migration/1]-->
   <!--14 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ksoftirqd/1]-->
   <!--16 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kworker/1:0H]-->
   <!--18 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [watchdog/2]-->
   <!--19 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [migration/2]-->
   <!--20 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ksoftirqd/2]-->
   <!--22 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kworker/2:0H]-->
   <!--23 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [watchdog/3]-->
   <!--24 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [migration/3]-->
   <!--25 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ksoftirqd/3]-->
   <!--27 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kworker/3:0H]-->
   <!--28 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [watchdog/4]-->
   <!--29 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [migration/4]-->
   <!--30 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ksoftirqd/4]-->
<!--32799 be/4 huairuo     0.00 B/s    0.00 B/s  0.00 %  0.00 % java -Xms2048m -Xmx2048m -Xmn512m -XX:+~rver-1.0-SNAPSHOT.jar [C2 CompilerThre]-->
   <!--32 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kworker/4:0H]-->
   <!--33 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [watchdog/5]-->
<!--32802 be/4 huairuo     0.00 B/s    0.00 B/s  0.00 %  0.00 % java -Xms2048m -Xmx2048m -Xmn512m -XX:+~rver-1.0-SNAPSHOT.jar [C2 CompilerThre]-->
   <!--35 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ksoftirqd/5]-->
```