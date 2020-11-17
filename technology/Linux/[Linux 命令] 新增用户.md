## 密码访问


```shell
userdel -r wpzhou
adduser  wpzhou
usermod -g admin  wpzhou
root@150501d113:~# usermod -g admin wpzhou1
usermod: group 'admin' does not exist
# 加sudo权限
root@150501d113:~# vi /etc/sudoers
```


## 密钥登录
```shell
su  wpzhou
cd ~
mkdir .ssh
vi authorized_keys
在文件中将用户的公钥添加进去，然后用户就可以用密钥登陆了
chmod 600 authorized_keys
```

```shell
userdel -r webadmin;
adduser  webadmin;
usermod -g webadmin  webadmin;
```

