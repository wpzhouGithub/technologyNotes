### 修改系统时区

```shell
# 查看系统时区
date -R   

# 1. 运行tzselect，然后依次选择4>9>1 (亚洲、中国、北京时间)
tzselect
# 2. 复制文件到指定目录
cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
```



### 海外机器更新系统时间，同步互联网时间

```shell
#同步时间
ntpdate time.nist.gov
#时间写入硬件
hwclock --systohc
```



### 迁移新机器改了时区， syslog crontab 时间与系统时间不一致

```shell
service rsyslog restart
service  cron restart
```

