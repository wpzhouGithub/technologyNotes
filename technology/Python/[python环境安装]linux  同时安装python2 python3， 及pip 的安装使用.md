# [python环境安装]linux  同时安装python2 python3， 及pip 的安装使用

[TOC]

# 1. python 的安装

 - 由于ubuntu系统自带python2.7（默认）和python3.4，所以不需要自己安装python。 
 - 可以使用python -V和python3 -V查看已安装python版本。



# 2. pip 的安装

 - 在不同版本的python中ubuntu默认没有安装pip，所以需要自己手动安装pip。 
 - 在不同版本中安装pip，可以使用一下命令：

```
sudo apt-get install python-pip
sudo apt-get install python3-pip
```

 - 安装完成后可以使用pip -V和pip3 -V查看看装的pip版本。


# 3. python包安装
 - 在使用pip安装其他库时，默认的python版本可以直接使用pip install XXXX
 - 另外的python版本可以使用python3 -m pip install XXXX 或pip3 install XXXX
