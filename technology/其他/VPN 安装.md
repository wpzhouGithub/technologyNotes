服务器安装

```shell
# 系统环境：ubuntu18
apt-get  update
apt-get install  python3-pip
pip3 install  shadowsocks

# 编辑配置文件
vi /etc/shadowsocks.json
{
"server":"0.0.0.0",
"local_address":"127.0.0.1",
"local_port":1080,
"port_password":{
	"9000":"wpzhou",
	"9001":"wpzhou",
	"9002":"wpzhou"
},
"timeout":300,
"method":"aes-256-cfb",
"fast_open": false
}

# 开放端口
ufw allow 9000
ufw allow 9001
ufw allow 9002

#启动服务
ssserver  -c /etc/shadowsocks.json  -d start

#报错
AttributeError: /usr/lib/x86_64-linux-gnu/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup
# 报错解决，所有EVP_CIPHER_CTX_cleanup 替换为EVP_CIPHER_CTX_reset
grep EVP_CIPHER_CTX_cleanup  /usr/local/lib/python3.6/dist-packages/shadowsocks/crypto/openssl.py  -c 
vi  /usr/local/lib/python3.6/dist-packages/shadowsocks/crypto/openssl.py  
EVP_CIPHER_CTX_cleanup 替换为EVP_CIPHER_CTX_reset

ssserver  -c /etc/shadowsocks.json  -d start

# 启动ufw
ufw enable
ufw status
```

