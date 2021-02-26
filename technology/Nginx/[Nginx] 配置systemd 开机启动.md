# [Nginx] 配置systemd 开机启动

- 配置service文件

```shell
## /lib/systemd/system/nginx.service
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target
 
[Service]
Type=forking
# 跟nginx的pid文件保持一致
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre= /usr/bin/nginx -t
ExecStart=/usr/bin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
 
[Install]
WantedBy=multi-user.target
```

```shell
systemctl daemon-reload
# 开启开机启动
systemctl enable nginx.service
# 验证设置是否成功
systemctl --all | grep nginx
# 重启
systemctl restart  nginx
```

