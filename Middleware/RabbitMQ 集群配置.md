# RabbitMQ 集群配置

## 1、修改 hostname

```
sudo hostnamectl set-hostname rabbit01
sudo hostnamectl set-hostname rabbit02

# 本机 hostname 修改后 /etc/hosts 里 127.0.0.1 那一行加上新改的 hostname，
# 否则使用 hostname 访问不到本地（mq 命令创建队列等操作会通过 hostname 寻找 mq ip）
sudo vi /etc/hosts

# 重启服务器
reboot
```

## 2、安装 MQ

```
# 使用安装包进行安装
```

### 可能出现问题

1、修改了 hostname 为 rabbit01，但 /etc/hosts 中 127.0.0.1 没有添加新的 rabbit01，
执行 `rabbitmqadmin` 命令时提示连接不上 rabbit01。

```
rabbit@host01:
  * unable to connect to epmd (port 4369) on host01: nxdomain (non-existing domain)


Current node details:
 * node name: 'rabbitmqcli-3167-rabbit@host01'
 * effective user's home directory: /var/lib/rabbi
```

解决：先执行命令 `sudo rabbitmq-server -detached`

## 3、配置 mq

### 3.1、先关闭从节点 rabbit02 的 mq

```
rabbitmqctl stop_app
```

### 3.2、同步两台机器的erlang.cookie

```
# rabbit01
cd /var/lib/rabbitmq
sudo chmod 777 .erlang*
scp .erlang.cookie  root@ip:/var/lib/rabbitmq

# rabbit02
cd /var/lib/rabbitmq
sudo chown rabbitmq:rabbitmq .er*
sudo chmod 400 .er*
```

### 3.3、集群搭建

```
####################################### rabbit01  ########################
#修改节点名称，添加配置内容
sudo vi /usr/lib/rabbitmq/bin/rabbitmq-env

<!--添加内容-->
<!--RABBITMQ_NODENAME=rabbit@host01-->

<!--rabbitmq-env 文件部分内容-->
<!--##  basis, WITHOUT WARRANTY OF ANY KIND, either express or implied. See-->
<!--##  the License for the specific language governing rights and-->
<!--##  limitations under the License.-->
<!--##-->
<!--##  The Original Code is RabbitMQ.-->
<!--##-->
<!--##  The Initial Developer of the Original Code is GoPivotal, Inc.-->
<!--##  Copyright (c) 2007-2015 Pivotal Software, Inc.  All rights reserved.-->
<!--##-->

<!--RABBITMQ_NODENAME=rabbit@host01-->

<!--if [ "$RABBITMQ_ENV_LOADED" = 1 ]; then-->
    <!--return 0;-->
<!--fi-->

<!--if [ -z "$RABBITMQ_SCRIPTS_DIR" ]; then-->
    <!--# We set +e here since since our test for "readlink -f" below needs to-->
    <!--# be able to fail.-->
    <!--set +e-->
    <!--# Determine wher-->
    
sudo rabbitmqctl set_cluster_name rabbit@host01 rabbit_cluster



################### rabbit02 #######################
sudo rabbitmq-server -detached
sudo rabbitmqctl stop_app
pkill –f rabbitmq # 这个命令好像报错了
sudo rabbitmqctl join_cluster --ram rabbit@rabbit01 # 注意防火墙，或者端口是否能通4369

```