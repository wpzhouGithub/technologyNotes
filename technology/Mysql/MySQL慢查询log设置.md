# MySQL慢查询log设置

## 开启慢查询log

```shell
# 查找MySQL使用的具体配置文件
# 查找进程是否指定配置文件
ps -ef | grep mysql
# 进程未直接指定配置文件，使用命令查找默认的配置文件；多个文件的，优先级排在前面的优先
mysqld --verbose --help | grep 'my.cnf'
/etc/my.cnf /etc/mysql/my.cnf ~/.my.cnf 
# 依次查看各个文件，最后找到配置文件
/etc/mysql/mysql.conf.d/mysqld.cnf
# 配置文件中没有看到内存相关的异常配置

# 配置文件开启慢查询
slow_query_log		= 1
slow_query_log_file	= /var/log/mysql/mysql-slow.log
long_query_time = 2

# 命令开启慢查询log， 直接生效，不需要重启MySQL
set global slow_query_log_file = '/var/log/mysql/mysql-slow.log';
set global slow_query_log=ON;
mysql> show variables like 'slow_query%';
+---------------------+-------------------------------+
| Variable_name       | Value                         |
+---------------------+-------------------------------+
| slow_query_log      | ON                            |
| slow_query_log_file | /var/log/mysql/mysql-slow.log |
+---------------------+-------------------------------+
2 rows in set (0.00 sec)

mysql> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+

```

​    

## 设置毫秒级的慢查询

直接设置小数即可，如200毫秒设置为0.2 

```my
# 不使用global时，只对本地有效
mysql> set long_query_time = 0.1;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like 'long_query_time';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 0.100000 |
+-----------------+----------+
1 row in set (0.00 sec)

# 使用global之后，全局生效； 本地查看值不会有变化，需要exit 再进入，才能生效； 使用另外的账号登录，也生效了
mysql> set  global long_query_time = 0.200;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like 'long_query_time';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 0.100000 |
+-----------------+----------+
1 row in set (0.00 sec)

```



