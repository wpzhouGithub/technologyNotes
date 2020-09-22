**查看/proc/uptime文件计算系统启动时间**
cat /proc/uptime
输出: 5113396.94 575949.85
第一数字即是系统已运行的时间5113396.94 秒，运用系统工具date即可算出系统启动时间

- 代码: 

  `date -d "$(awk -F. '{print $1}' /proc/uptime) second ago" +"%Y-%m-%d %H:%M:%S"`


输出: 2008-11-09 11:50:31



查看系统版本

```shell
head /etc/issue
Ubuntu 16.04.5 LTS \n \l

```



