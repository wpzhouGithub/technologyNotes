# shell 解析json 中的字段

- log
```shell
{ "@timestamp": "2019-12-20T08:30:04+00:00", "http_host": "api.u-launcher.com", "remote_addr": "41.188.67.242", "request_method": "GET", "url": "/client/v2/resource/config.json", "request": "GET /client/v2/resource/config.json?language=ar&country=AE&packageName=com.cyou.privacysecurity&sdkVersion=22&versionCode=20340000 HTTP/1.1", "status": 200, "up_status": "200", "up_cache_status": "-", "request_time": 0.004, "up_resp_time": "0.004", "up_addr": "10.0.0.104:8201", "bytes": 711, "x_forwarded": "41.188.67.242", "agent": "-" }
{ "@timestamp": "2019-12-20T08:30:04+00:00", "http_host": "api.u-launcher.com", "remote_addr": "223.188.174.49", "request_method": "GET", "url": "/client/v2/resource/config.json", "request": "GET /client/v2/resource/config.json?language=en&country=GB&packageName=com.cyou.privacysecurity&sdkVersion=22&versionCode=20340000 HTTP/1.1", "status": 200, "up_status": "200", "up_cache_status": "-", "request_time": 0.008, "up_resp_time": "0.008", "up_addr": "10.0.0.104:8201", "bytes": 711, "x_forwarded": "223.188.174.49", "agent": "-" }
{ "@timestamp": "2019-12-20T08:30:04+00:00", "http_host": "api.u-launcher.com", "remote_addr": "41.188.67.245", "request_method": "GET", "url": "/client/v2/resource/config.json", "request": "GET /client/v2/resource/config.json?country=MR&language=ar&sdkVersion=27&packageName=com.cyou.privacysecurity&versionCode=20340000 HTTP/1.1", "status": 200, "up_status": "200", "up_cache_status": "-", "request_time": 0.002, "up_resp_time": "0.002", "up_addr": "10.0.0.103:8201", "bytes": 711, "x_forwarded": "41.188.67.245", "agent": "-" }
{ "@timestamp": "2019-12-20T08:30:04+00:00", "http_host": "api.u-launcher.com", "remote_addr": "156.38.50.251", "request_method": "GET", "url": "/client/v2/resource/config.json", "request": "GET /client/v2/resource/config.json?versionCode=20340000&language=fr&packageName=com.cyou.privacysecurity&sdkVersion=19&country=FR HTTP/1.1", "status": 200, "up_status": "200", "up_cache_status": "-", "request_time": 0.002, "up_resp_time": "0.002", "up_addr": "10.0.0.104:8201", "bytes": 711, "x_forwarded": "156.38.50.251", "agent": "-" }
{ "@timestamp": "2019-12-20T08:30:04+00:00", "http_host": "api.u-launcher.com", "remote_addr": "149.255.210.55", "request_method": "GET", "url": "/client/locker/update/config.do", "request": "GET /client/locker/update/config.do?language=ar&packageName=com.cyou.privacysecurity HTTP/1.1", "status": 200, "up_status": "200", "up_cache_status": "-", "request_time": 0.003, "up_resp_time": "0.003", "up_addr": "10.0.1.148:8080", "bytes": 943, "x_forwarded": "149.255.210.55", "agent": "-" }

```

- 命令
```shell
#解析json 中的request的链接
awk -F "[,:}]" '/client\/v2\//{for(i=1;i<=NF;i++){if($i~/request\042/){split($(i+1), r, " "); sum[r[2]]+=1;}}; } END {for (a in sum) print sum[a], a}'

```

