# Mysql

[* 安装配置](#安装配置)

[* 启动服务](#启动服务)

[* 命令行执行 sql 等](#命令行执行sql)

[* 导出 mysql 数据](#导出mysql数据)

## <h2 id="安装配置">安装配置</h2>

```
# 查看 mysql 安装随机生成的密码
grep "temporary password" /var/log/mysqld.log

########################################################################
############################ 修改  root 初始密码  ########################
########################################################################
# 方式一
set password for root@localhost=password('123456')
# 方式 2
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
# MYSQL 8.8 版本好像不支持 alter 创建用户。
CREATE USER 'root'@'localhost' IDENTIFIED BY '123456';

# 设置密码提示密码简单，不能设置成功：
# Your password does not satisfy the current policy requirements
set global validate_password_policy=LOW;
set global validate_password_length=6;

# 设置远程访问
# 注： 'root'@'localhost'、 'root'@'%' 在数据库中属于不同用户。
grant all privileges on *.* to 'root'@'%' IDENTIFIED BY '8888';
# mysql 8
grant all privileges on *.* to 'root'@'%';
# 更新配置信息
flush privileges;

```

## <h2 id="启动服务">启动服务</h2>

```
systemctl start mysqld
```

## <h2 id="命令行执行sql">命令行执行sql</h2>

```
mysql --user='root' --password='**' -e 'sql'
```



