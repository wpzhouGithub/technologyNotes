# Ubuntu查看进程的子进程、线程的运行情况

### 1. 查看进程

```shell
ps -ef | grep run.py
root     14809 14431  0 10:09 pts/0    00:00:00 python run.py
root     14810 14809  0 10:09 pts/0    00:00:13 python run.py
root     14811 14809  0 10:09 pts/0    00:00:15 python run.py

```

可以看到， 进程14809有两个子进程

### 2. 查看进程的子线程

注：*lwp      LWP    lwp (light weight process, or thread) ID of the lwp being reported. (alias spid, tid).*

就是说lwp spid tid都是指线程ID.

```shell
ps -eLf | grep run.py
UID        PID  PPID   LWP  C NLWP STIME TTY          TIME CMD
root     14809 14431 14809  0    1 10:09 pts/0    00:00:00 python run.py
root     14810 14809 14810  0    3 10:09 pts/0    00:00:07 python run.py
root     14810 14809 14818  0    3 10:09 pts/0    00:00:03 python run.py
root     14810 14809 14819  0    3 10:09 pts/0    00:00:03 python run.py
root     14811 14809 14811  0    3 10:09 pts/0    00:00:09 python run.py
root     14811 14809 14816  0    3 10:09 pts/0    00:00:03 python run.py
root     14811 14809 14817  0    3 10:09 pts/0    00:00:03 python run.py
```

可以看到14810、14811分别有两个子线程

### 3. 树状图显示子进程、子线程

```shell
pstree -p 14809
python(14809)─┬─python(14810)─┬─{python}(14818)
              │               └─{python}(14819)
              └─python(14811)─┬─{python}(14816)
                              └─{python}(14817)                              
```
树状图直接显示了子进程及子进程对应的子线程

### 4. 查看子进程、子线程的资源消耗情况，如cpu，内存

```shell
# 查看单独进程
top -H -p 14810
top - 14:56:00 up 41 days, 15:43,  3 users,  load average: 0.06, 0.02, 0.00
Threads:   3 total,   0 running,   3 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.2 us,  0.2 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16432624 total, 12267932 free,   743776 used,  3420916 buff/cache
KiB Swap:  1046524 total,   995912 free,    50612 used. 15131036 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                          
14810 root      20   0  200672  27092   5488 S  0.0  0.2   0:07.27 python                                                                                           
14818 root      20   0  200672  27092   5488 S  0.0  0.2   0:03.07 python                                                                                          
14819 root      20   0  200672  27092   5488 S  0.0  0.2   0:03.13 python

# 查看多个进程
top -H -p 14809, 14810,14811
top - 15:11:58 up 41 days, 15:59,  3 users,  load average: 0.00, 0.00, 0.00
Threads:  10 total,   0 running,  10 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.4 us,  0.0 sy,  0.0 ni, 99.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.4 us,  0.0 sy,  0.0 ni, 99.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16432624 total, 12261664 free,   747096 used,  3423864 buff/cache
KiB Swap:  1046524 total,   995912 free,    50612 used. 15127744 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                          
14809 root      20   0   43660  24868  10344 S  0.0  0.2   0:00.22 python                                                                                           
14810 root      20   0  200928  27252   5624 S  0.0  0.2   0:07.69 python                                                                                           
14818 root      20   0  200928  27252   5624 S  0.0  0.2   0:03.20 python                                                                                           
14819 root      20   0  200928  27252   5624 S  0.0  0.2   0:03.26 python                                                                                           
14811 root      20   0  425016  28736   6376 S  0.0  0.2   0:09.80 python                                                                                           
14816 root      20   0  425016  28736   6376 S  0.0  0.2   0:03.16 python                                                                                           
14817 root      20   0  425016  28736   6376 S  0.0  0.2   0:03.15 python                                                                                           
14934 root      20   0  425016  28736   6376 S  0.0  0.2   0:00.00 python                                                                                           
14935 root      20   0  425016  28736   6376 S  0.0  0.2   0:00.12 python                                                                                           
14938 root      20   0  425016  28736   6376 S  0.0  0.2   0:00.00 python

```


