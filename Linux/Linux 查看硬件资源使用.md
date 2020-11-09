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
 