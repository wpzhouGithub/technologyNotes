@[TOC](linux 软件、硬链)

# 1. 命令语法

```
 ln -s 源文件 目标文件
 -s 表示软链， 默认是硬链
```

链接： 会保持每一处链接文件的同步性，也就是说，不论你改动了哪一处，其它的文件都会发生相同的变化

# 2. 软链、硬链

软链接：ln –s ** **，它只会在你选定的位置上生成一个文件的镜像，不会占用磁盘空间**

​	删除软链目标文件时，源文件不受影响；

删除源文件时，cat 目标文件报错： No such file or directory

修改软链的源文件和目标文件，对应的源文件、目标文件都会同步变化

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190820185733550.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAwODAyMzU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190820185754228.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAwODAyMzU=,size_16,color_FFFFFF,t_70)
硬链接： ln ** **，没有参数-s， 它会在你选定的位置上生成一个和源文件大小相同的文件

删除目标文件或者源文件，另外的文件不受影响；

更新其中一个文件，另外的文件同步变化

件不受影响；

更新其中一个文件，另外的文件同步变化

