# Mysql

## 安装配置

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

# 设置密码提示密码简单，不能设置成功：
# Your password does not satisfy the current policy requirements
set global validate_password_policy=LOW;
set global validate_password_length=6;

# 设置远程访问
grant all privileges on *.* to 'root'@'%' IDENTIFIED BY '8888';
# 更新配置信息
flush privileges;

```

## 启动服务

```
systemctl start mysqld
```