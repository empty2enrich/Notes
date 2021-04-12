# Mysql

[* 安装配置](#安装配置)

[* 启动服务](#启动服务)

[* 命令行执行 sql 等](#命令行执行sql)

[* 导出 mysql 数据](#导出mysql数据)

[* MySql 处理 json 串](#MySql处理json串)

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

## <h2 id="MySql处理json串">MySql 处理 json 串</h2>

* JSON_EXTRACT

* 一些常用的方法

## JSON_EXTRACT

```
# field_name 是 table 中的字段， json_key 是 field_name 字段值里 json 串的 key
# 使用了不存在的 key, sql 会执行报错。
select JSON_EXTRACT(field_name, '$.json_key') from table;

# 同时 JSON_EXTRACT 返回的值是带有“"” 的，使用的的时候可以使用 replace 替换
elect REPLACE(JSON_EXTRACT(field_name, '$.json_key'), '"', '') from table;    
```

## 一些常用的方法

### 创建 json

* json_array 创建 json 数组

* json_object 创建 json 对象

* json_quote 将 json 转成 json 字符串类型

### 查询 json

* json_contains

* json_contains_path 判断某个路径下是否包含 json 值

* json_extract 获取json 串中某个 key 的值

* column->path json_extract 的缩写

* column->>path json_unquote(column->path) 缩写

* json_keys 获取 json 串的 keys，返回值是 json 数组

* json_search 按给定字符串关键字搜索 json，返回匹配路径

### 修改 json

* json_append 已废弃

* json_array_append 

* json_array_insert

* json_insert 插入新值

* json_merge 合并 json 数组或对象

* json_remove 删除 json 数据

* json_replace 替换值 （只替换已存在的旧值）

* json_set 设置值（替换旧值或插入新值）

* json_unquote 去除 json 串的引号

### 返回 json 属性

* json_depth 返回 json 串的最大深度

* json_length 返回 json 长度

* json_type 返回 json 值类型

* json_valid 盘是否为合法 json 串