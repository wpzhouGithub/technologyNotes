@[TOC](Ubuntu代理设置)

# 1. 代理设置

### 在.bashrc类似文件中设置：

```shell
http_proxy=http://proxy3.baina.com:25378
export http_proxy

https_proxy=http://proxy3.baina.com:25378
export https_proxy

ftp_proxy=ftp://proxy3.baina.com:25378
export ftp_proxy

sftp_proxy=socks://proxy3.baina.com:25378
export sftp_proxy
```



### ~/.ssh/config ， scp 使用代理的设置

```shell
Host *
    ProxyCommand nc -X connect  -x proxy3.baina.com:25378 %h %p
```



# 2. .bashrc类似文件的加载顺序

**从左到有依次加载**

**/etc/profile ->/etc/enviroment --> /etc/bash.bashrc --> $HOME/.profile-->$HOME/.env->$HOME/.bash* ** 



# 3. scp使用代理

**设置的代理，直接运行scp 命令可以用，但是使用python的paramiko.SFTPClient 没有生效**