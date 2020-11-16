# Linux 设置修改

[修改最大线程数](#修改最大线程数)
[修改最大打开文件数](#修改最大打开文件数)

## <h2 id="修改最大线程数">修改最大线程数</h2>

概述：当系统线程数到达设置的最大数，就不能新开线程，导致程序、shell（拒绝连接） 等
不能正常使用。

```
sudo vi /etc/security/limits.d/20-nproc.conf 

#######################################################
#################20-nproc.conf 文件内容如下#############################
#######################################################

# Default limit for number of user's processes to prevent
# accidental fork bombs.
# See rhbz #432903 for reasoning.

*          soft    nproc     65535
root       soft    nproc     unlimited

```


## <h2 id="修改最大打开文件数">修改最大打开文件数</h2>

```
sudo vi /etc/security/limit.conf

#######################################################
################# limit.conf 文件内容如下#############################
#######################################################

*               soft    noproc          65535
*               hard    noproc          65535
*               soft    nofile          65535
*               hard    nofile          65535


```
