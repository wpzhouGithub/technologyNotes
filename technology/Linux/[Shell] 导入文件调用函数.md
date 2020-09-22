- 公用公共的方法、变量；

common.sh

```shell
#!/usr/bin/env bash

LOG_PATH=/var/log/data_report
CODE_PATH=/var/app/enabled/data_report/data_report/
PYTHON=/root/miniconda3/envs/data_report_monitor/bin/python3

function kill_timeout_job() {
    project_name=$1
    seconds=$2
    sys_uptime=$(cat /proc/uptime | cut -d" " -f1);
    user_hz=$(getconf CLK_TCK) ;
    for pid in `ps -ef | grep $project_name | egrep -v "grep|tail" | awk '{print $2}'`;do
        pid_uptime=$(cat /proc/$pid/stat | cut -d" " -f22);
        last_time=$((${sys_uptime%.*}-$pid_uptime/$user_hz ));
        echo $last_time
        echo $pid "execute time of " $last_time "second";
        if [ $last_time -gt $seconds ];then
            echo $pid "$project_name execute time is over 600 seconds"
            ps -ef | grep $pid| grep -v "grep"
            kill -9 $pid
        fi
    done
}

function run_job() {
    project_name=$1
    parms=$2
    log=$3
    ProcNumber=`ps -ef |grep -w $project_name| egrep -v "grep|tail"|wc -l`
    if [ $ProcNumber -le 0 ];then
        echo "$project_name is not run"
        cd $CODE_PATH
        cmd="$PYTHON $parms -label $project_name >> $LOG_PATH/$log &"
        echo $cmd
        $cmd
    else
       echo "$project_name is  running.."
    fi
}


```

run.sh

```shell
#!/usr/bin/env bash

cd "$(dirname "$0")"
. ./common.sh

PROC_NAME=run_facebook_summary
SECONDS=900
LOG=run_facebook.log
PARMS="run_facebook.py summary"

date
kill_timeout_job $PROC_NAME $SECONDS
run_job "$PROC_NAME" "$PARMS" "$LOG"

```

