[TOC]

# git每次都输入用户名密码

## 背景

使用GitHub每次pull等操作的时候都要输入密码，很费时间；

## 解决方法

1. 直接使用密钥提交，此处不详细讲

2. 使用git的认证信息存储

   ```bash
   # 下面命令会将下次弹框的账号和密码保存起来，永久使用
   git config --global credential.helper store
   
   # 报错
   Logon failed, use ctrl+c to cancel basic credential prompt
   # 需要更新git
   windows: git update-git-for-windows
   Linux/Unix: git update
   
   # 报错
   fatal: unable to access 'https://github.com/wpGithub/gitStudy.git/': OpenSSL SSL_read: Connection was reset, errno 10054
   # 产生原因：一般是这是因为服务器的SSL证书没有经过第三方机构的签署，所以才报错
   # 参考网上解决办法：解除ssl验证后，再次git即可
   git config --global http.sslVerify false
   
   ```

   