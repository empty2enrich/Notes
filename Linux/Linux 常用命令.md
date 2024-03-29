# Linux 常用命令

## 目录
* [find](#find)
* [sed](#sed)
* [awk](#awk)
* [cut](#cut)
* [sudo](#sudo)
* [chown](#chown)
* [chmod](#chmod)
* [>](#>)
* [fuser](#fuser)
* [ps](#ps)
* [uname](#uname)
* [lsof](#lsof)


## <h2 id="find">find</h2>

* `find dir -name name` (根据文件名查询)
* `find -size [+-]N[bcwkMG]` (根据文件大小查询)
```
# b,c(byte),w,k,M,G 是表示文件大小的单位
# + 表示 size>N, - 表示 size <= N, 无符号表示 N-1 < size <= N  
```
*  (查询到文件并对这些文件执行 `ll -h` 命令，显示文件信息)

`find dir -name name -exec -ls -l {} \;`

`find dir -name name | xargs ls -lh`


## <h2 id="sed">sed</h2>

sed 替换规则里 "/" 可以使用正则, "#" 使用原始 str 直接替换，有特殊字符会转义。 
`-i` 表示把替换后的问题写入原文件, 没有 `-i` 把替换后的问题输出控制台（原文件不变）。

```
# replace str 与 target str 可以使用正则
sed -i "s/replace_str/target_str/g"
# replace str 与 target str 不能使用正则
sed -i "s#replace_str#target_str/g"
# 在每行末尾追加内容
sed -i '/$/a\\n' test.txt #文件的每行末尾添加一个回车
# 在文件末尾追加内容
sed -i '$a\eof' test.txt # 文件末尾添加 'eof'
```

## <h2 id="sudo">sudo</h2>

使用管理员权限执行命令

```
# 不用手动输入密码
echo "password" |sudo -S command

```

## <h2 id="chown">chown</h2>

修改文件所属用户组

chown 参数：

-c或——changes：效果类似“-v”参数，但仅回报更改的部分；

-f或--quite或——silent：不显示错误信息；

-h或--no-dereference：只对符号连接的文件作修改，而不更改其他任何相关文件；

-R或——recursive：递归处理，将指定目录下的所有文件及子目录一并处理；

-v或——version：显示指令执行过程；

--dereference：效果和“-h”参数相同；

--help：在线帮助；

--reference=<参考文件或目录>：把指定文件或目录的拥有者与所属群组全部设成和参考文件或目录的拥有者与所属群组相同；

--version：显示版本信息

```
# 把 dir_name 的所属用户用户组修改 test:test
chown -R test:test dir_name
```

## <h2 id="chmod">chmod</h2>

修改文件权限。

chmod [-cfvR] [--help] [--version] mode file...

mode：权限设定字符串，格式如下：

[ugoa...][[+-=][rwxX]...][,...]

参数说明：

u 表示该文件的拥有者，g 表示与该文件的拥有者属于同一个群体(group)者，o 表示其他以外的人，a 表示这三者皆是。
'+' 表示增加权限、'-' 表示取消权限、'=' 表示唯一设定权限。
r 表示可读取，w 表示可写入，x 表示可执行，X 表示只有当该文件是个子目录或者该文件已经被设定过为可执行

其他参数说明：

-c : 若该文件权限确实已经更改，才显示其更改动作
-f : 若该文件权限无法被更改也不要显示错误讯息
-v : 显示权限变更的详细资料
-R : 对目前目录下的所有文件与子目录进行相同的权限变更(即以递回的方式逐个变更)
--help : 显示辅助说明
--version : 显示版本

## <h2 id=">">></h2>

`&>`: 把所有输出内容写到文件内
 
`2>`: 把标准错误写到文件内

`1>`：把标准输出写到文件内

`>`：把标准输出写到文件内


## <h2 id="fuser">fuser</h2>

根据文件名，查看哪些进程在使用该文件、端口、系统(Show which processes use the 
named files, sockets, or filesystems.)。

```
[huairuo@rabbit02 ~]$ fuser -v /dev/nvidia*
                     USER        PID ACCESS COMMAND
/dev/nvidia0:        huairuo   20756 F...m python
/dev/nvidia1:        huairuo   20544 F...m python
/dev/nvidia2:        huairuo   20763 F...m python
/dev/nvidiactl:      huairuo   20544 F...m python
                     huairuo   20756 F...m python
                     huairuo   20763 F...m python
/dev/nvidia-uvm:     huairuo   20544 F.... python
                     huairuo   20756 F.... python
                     huairuo   20763 F.... python
```

## <h2 id="ps">ps</h2>

查看进程状态，指定输出信息
```
ps -eo user,stat,..,cmd 
<!--user          用户名 -->
<!--uid           用户号 -->
<!--pid           进程号 -->
<!--ppid          父进程号 -->
<!--size          内存大小, Kbytes字节. -->
<!--vsize         总虚拟内存大小, bytes字节(包含code+data+stack) -->
<!--share         总共享页数 -->
<!--nice          进程优先级(缺省为0, 最大为-20) -->
<!--priority(pri) 内核调度优先级 -->
<!--pmem          进程分享的物理内存数的百分比 -->
<!--trs           程序执行代码驻留大小 -->
<!--rss           进程使用的总物理内存数, Kbytes字节 -->
<!--time          进程执行起到现在总的CPU暂用时间 -->
<!--stat          进程状态 -->
<!--cmd(args)     执行命令的简单格式 -->

<!--进程状态-->
<!--D 不可中断 uninterruptible sleep (usually IO) -->
<!--R 运行 runnable (on run queue) -->
<!--S 中断 sleeping -->
<!--T 停止 traced or stopped -->
<!--Z 僵死 a defunct ("zombie") process -->
```

## <h2 id="uname">uname</h2>

* uname -m 查看系统架构

* uname -a 查看系统架构详细信息

## <h2 id="lsof">lsof</h2>

查看文件被哪些进程读写。

```
lsof |grep file_name
```
