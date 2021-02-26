# [MySQL] log文件不输出 log文件都为空

## 问题表象

- MySQL启动一段时间之后，去分析`error.log` `mysql-slow.log` 都为空
- `error.log.1`   `mysql-slow.log.1`的更新时间新于`error.log`   `mysql-slow.log`
- 重启MySQL能恢复

## 问题解决

1. 查看MySQL打开的文件句柄

   ```shell
   # 获取pid
   ps  -ef | grep mysql
   mysql    15850     1  0 Feb02 ?        00:53:59 /usr/sbin/mysqld --daemonize --pid-file=/run/mysqld/mysqld.pid
   root     30242 27799  0 15:36 pts/0    00:00:00 grep --color=auto mysql
   
   # 查看对应进程打开的文件
   cd /proc/15850
   ll fd | grep log
   l-wx------ 1 mysql mysql 64 Feb  7 14:43 1 -> /var/log/mysql/error.log.1 (deleted)
   lrwx------ 1 mysql mysql 64 Feb  7 14:43 124 -> /mnt/mysql_data/mysql/superset/logs.ibd
   l-wx------ 1 mysql mysql 64 Feb  7 14:43 17 -> /var/log/mysql/mysql-slow.log.1 (deleted)
   l-wx------ 1 mysql mysql 64 Feb  7 14:43 2 -> /var/log/mysql/error.log.1 (deleted)
   lrwx------ 1 mysql mysql 64 Feb  7 14:43 3 -> /mnt/mysql_data/mysql/ib_logfile0
   lrwx------ 1 mysql mysql 64 Feb  7 14:43 9 -> /mnt/mysql_data/mysql/ib_logfile1
   
   ```

   - 文件句柄显示，MySQL打开的是*log.1

2. *log.1 是log rotate在log切割的时候产生的文件，去看log rotate MySQL的配置

   ```shell
   ## /etc/logrotate.d/mysql-server
   /var/log/mysql.log /var/log/mysql/*log {
   	daily
   	rotate 7
   	missingok
   	create 640 mysql adm
   	su mysql adm
   	compress
   	sharedscripts
   	postrotate
   		test -x /usr/bin/mysqladmin || exit 0
   		# If this fails, check debian.conf! 
   		MYADMIN="/usr/bin/mysqladmin --defaults-file=/etc/mysql/debian.cnf"
   		if [ -z "`$MYADMIN ping 2>/dev/null`" ]; then
   		  # Really no mysqld or rather a missing debian-sys-maint user?
   		  # If this occurs and is not a error please report a bug.
   		  #if ps cax | grep -q mysqld; then
   		  if killall -q -s0 -umysql mysqld; then
    		    exit 1
   		  fi 
   		else
   		  $MYADMIN flush-logs
   		fi
   	endscript
   }
   ```

   - 切割日志需要注意的一个关键点就是日志切割之后，程序任然读写原来的文件；

   - 这里MySQL写的文件是*.log.1 应该就是log rotate配置的问题了

   - 对MySQL来说有两种方式
     - `flush-logs`
     
     - `copytruncate` （会丢失一定的数据）
     
       ```tex
       问题：如何告诉应用程序重新打开日志文件？
       以Nginx为例，是通过postrotate指令发送USR1信号来通知Nginx重新打开日志文件的。但是其他的应用程序不一定遵循这样的约定，比如说MySQL是通过flush-logs来重新打开日志文件的。更有甚者，有些应用程序就压根没有提供类似的方法，此时如果想重新打开日志文件，就必须重启服务，但为了高可用性，这往往不能接受。还好Logrotate提供了一个名为copytruncate的指令，此方法采用的是先拷贝再清空的方式，整个过程中日志文件的操作句柄没有发生改变，所以不需要通知应用程序重新打开日志文件，但是需要注意的是，在拷贝和清空之间有一个时间差，所以可能会丢失部分日志数据。
       ```
     
   - 上面MySQL使用了`flush-logs`； 

     ```shell
     # 手动执行postrotate endscript代码块失败
     $MYADMIN ping 2
     mysqladmin: connect to server at 'localhost' failed
     error: 'Can't connect to local MySQL server through socket '/mnt/mysql_data/mysql/mysqld.sock' (2)'
     Check that mysqld is running and that the socket: '/mnt/mysql_data/mysql/mysqld.sock' exists!
     
     $MYADMIN flush-logs
     mysqladmin: connect to server at 'localhost' failed
     error: 'Can't connect to local MySQL server through socket '/mnt/mysql_data/mysql/mysqld.sock' (2)'
     Check that mysqld is running and that the socket: '/mnt/mysql_data/mysql/mysqld.sock' exists!
     
     # check mysql的服务
     ps -ef | grep mysql
     mysql    30532     1  0 15:54 ?        00:00:17 /usr/sbin/mysqld --daemonize --pid-file=/run/mysqld/mysqld.pid
     root     31056 27799  0 16:35 pts/0    00:00:00 grep --color=auto mysql
     
     ```

     - 手动执行命令报错，发现log rotate配置的MySQL的命令跟当前正在使用的不一样；修改配置如下：

     ```shell
     ## /etc/logrotate.d/mysql-server
     /var/log/mysql.log /var/log/mysql/*.log {
     	daily
     	rotate 7
     	missingok
     	create 640 mysql adm
     	su mysql adm
     	compress
     	sharedscripts
     	postrotate
     		test -x /usr/sbin/mysqld || exit 0
     		# If this fails, check debian.conf! 
     		MYADMIN="/usr/sbin/mysqld "
     		if [ -z "`$MYADMIN ping 2>/dev/null`" ]; then
     		  # Really no mysqld or rather a missing debian-sys-maint user?
     		  # If this occurs and is not a error please report a bug.
     		  #if ps cax | grep -q mysqld; then
     		  if killall -q -s0 -umysql mysqld; then
      		    exit 1
     		  fi 
     		else
     		  $MYADMIN flush-logs
     		fi
     	endscript
     }
     
     ```

     ```shell
     logrotate /etc/logrotate.d/mysql-server
     ll /var/log/mysql/ -t
     total 318360
     -rw-r-----  1 mysql adm        25724 Feb  7 15:39 error.log
     drwxrwxr-x 13 root  syslog      4096 Feb  7 14:55 ../
     drwxr-x---  2 mysql adm         4096 Feb  7 06:25 ./
     -rw-r-----  1 mysql adm            0 Feb  7 06:25 mysql-slow.log
     -rw-r-----  1 mysql adm            0 Feb  6 06:25 error.log.1
     -rw-r-----  1 mysql adm            0 Feb  6 06:25 mysql-slow.log.1
     ```

     - `error.log`已生效，但是`mysql-slow.log`还是不行；`flush-logs` 对 mysql-slow.log不生效；修改配置如下

     ```shell
     ## /etc/logrotate.d/mysql-server
     /var/log/mysql.log /var/log/mysql/error.log {
     	daily
     	rotate 7
     	missingok
     	create 640 mysql adm
     	su mysql adm
     	compress
     	sharedscripts
     	postrotate
     		test -x /usr/sbin/mysqld || exit 0
     		# If this fails, check debian.conf! 
     		MYADMIN="/usr/sbin/mysqld "
     		if [ -z "`$MYADMIN ping 2>/dev/null`" ]; then
     		  # Really no mysqld or rather a missing debian-sys-maint user?
     		  # If this occurs and is not a error please report a bug.
     		  #if ps cax | grep -q mysqld; then
     		  if killall -q -s0 -umysql mysqld; then
      		    exit 1
     		  fi 
     		else
     		  $MYADMIN flush-logs
     		fi
     	endscript
     }
     
     /var/log/mysql/mysql-slow.log {
             daily
             rotate 7
     	create 640 mysql adm
     	su mysql adm
             missingok     #如果日志文件不存在,继续处理下一个文件而不产生报错信息
             delaycompress #推迟要压缩的文件，直到下一轮询周期再执行压缩
             copytruncate  # 先拷贝再清空的方式，整个过程中日志文件的操作句柄没有发生改变
     }
     
     ```

     ```shell
     logrotate /etc/logrotate.d/mysql-server 
     ll /var/log/mysql/ -t
     total 318360
     -rw-r-----  1 mysql adm        25724 Feb  7 15:39 error.log
     drwxrwxr-x 13 root  syslog      4096 Feb  7 14:55 ../
     drwxr-x---  2 mysql adm         4096 Feb  7 06:25 ./
     -rw-r-----  1 mysql adm            0 Feb  7 06:25 mysql-slow.log
     
     ```

     - 仿佛没有生效，重启MySQL `service  mysql  restart` 
     - [ ] ` mysql-slow.log`有输出了，但是这里不一定是logrotate修改配置生效了，后期需要持续观察