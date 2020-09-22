@[TOC](Ubuntu 安装包的两种形式：)
# 1. 源码安装
 - ./configure 生成Makefile
 - make
 - make install
 - 不能用service启动，当然如果把执行文件拷贝到/etc/init...下是可以的
 - 本地编译文件，安装实际上是将编译好的文件复制到指定的安装目录
![安装目录](https://img-blog.csdnimg.cn/20190820170420699.png)
# 2. apt-get安装

 - 安装到默认的目录，可以用service 包名 start
![安装目录](https://img-blog.csdnimg.cn/20190820170254153.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAwODAyMzU=,size_16,color_FFFFFF,t_70)
# 3. apt命令的使用
 - apt-cache search packagename: 搜索要安装的包
 - dpkg -l | grep ruby: 查看已经安装的包
 - apt-show-versions -a python3： 查看当前系统支持的所有版本
```
>>apt-show-versions -a python3
python3:amd64 3.5.1-3 install ok installed
python3:amd64 3.5.1-3 xenial ap-southeast-1.ec2.archive.ubuntu.com
python3:amd64/xenial 3.5.1-3 uptodate
```





