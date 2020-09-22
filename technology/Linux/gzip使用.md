# gzip 压缩、解压文件

**生成的压缩文件或者解压的文件，都输出在文件的对应目录的，跟当前目录没有关系**



```shell
语法
file为文件名，包括目录

压缩文件
gzip -f file

解压文件，-f 是覆盖已存在的文件
gzip -f -d file.gz
```



```shell
#压缩
root@XS16788180114:~# gzip /var/app/enabled/data_report/data_export/adsense1_20190923.json

root@XS16788180114:~# ll -h  /var/app/enabled/data_report/data_export/adsense1_20190923.json.gz 

-rw-r--r-- 1 root root 683K Sep 23 16:03 /var/app/enabled/data_report/data_export/adsense1_20190923.json.gz


#解压
root@XS16788180114:~# gzip -f -d /var/app/enabled/data_report/data_export/adsense1_20190923.json.gz

root@XS16788180114:~# ll /var/app/enabled/data_report/data_export/adsense1_20190923.json -h 

-rw-r--r-- 1 root root 12M Sep 23 16:03 /var/app/enabled/data_report/data_export/adsense1_20190923.json

```