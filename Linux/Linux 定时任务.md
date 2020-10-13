# Crontab

## 启动、关闭、查看 crond 服务

```
# 查看定时任务状态
service crond status

# 启动定时任务
service crond start

# 关闭

```

## crontab 使用

```
# 添加当前用户定时任务
crontab -e 

# 通过文件添加定时任务 (file 里写有定时任务)
crontab file

# 显示定时任务
crontab -l

# 删除定时任务 (删除当前用户的所有定时任务)
crontab -r
```

## cron file 格式

```
  5 # For details see man 4 crontabs
  6 
  7 # Example of job definition:
  8 # .---------------- minute (0 - 59)
  9 # |  .------------- hour (0 - 23)
 10 # |  |  .---------- day of month (1 - 31)
 11 # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
 12 # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
 13 # |  |  |  |  |
 14 # *  *  *  *  * user-name  command to be executed

```
