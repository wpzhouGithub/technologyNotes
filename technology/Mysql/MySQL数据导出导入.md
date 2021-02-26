```shell
mysqldump -u$USER -p$PASSWD -h127.0.0.1 -P3306


mysqldump -u$USER -p$PASSWD -h127.0.0.1 -P3306 themes > themes.sql

mysql -u$USER -p$PASSWD -h127.0.0.1 -P3306 
source themes.sql

phpbb:$PASSWD@127.0.0.1/themes
```

