# 查看进程使用的cpu id

```shell
ps 命令可以告诉你每个进程/线程目前分配到的 （在“PSR”列）CPU ID。

$ ps -o pid,psr,comm -p <pid>
  PID PSR COMMAND
5357  10 prog

# 查看某进程id被分配的指定运行的cpu
taskset -c -p 1093
# 为pid 为1093的指定cpu 1-3
taskset -pc 1-3 1093
```


