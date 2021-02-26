# ubuntu系统启用root用户远程登陆



- 新创建的ubuntu系统服务器，默认的登陆用户名为ubuntu，那很多用户都想使用超级管理员root用户来操作自己的服务器，但是root用户默认是被禁止登陆的，该如何启用root用户呢？


操作方法如下：

1. 使用ubuntu用户登陆到系统中；

2. 编辑/etc/ssh/sshd_config文件；

    - 设置配置参数：PermitRootLogin yes

4. 给root用户设置密码；

   `sudo passwd root`

   输入两遍密码；给root用户设置了密码后，就已经可以使用root用户从vnc登陆到系统中了

5. 重启ssh服务

    ```sudo  systemctl  restart  ssh ```

然后测试用root用户远程登陆即可；

