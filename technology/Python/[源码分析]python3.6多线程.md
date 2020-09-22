原文档： [https://he11olx.com/2018/08/04/1.CPython3.6%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/1.13.Python%20%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%9C%BA%E5%88%B6/](https://he11olx.com/2018/08/04/1.CPython3.6源码分析/1.13.Python 多线程机制/)



多线程实际上还是操作系统控制的，是C 调用系统的API实现线程的控制；

GIL 是通过在C 中通过设置变量，来控制锁，控制什么时候挂起线程、什么时候获取唤醒线程

python的threading 底层都是调用的C 的thread库





有一个发现，或许这个可以用来跟深入的看异步的原理

从源码中还能发现，遇到`SETUP_FINALLY、YIELD_FROM`是不会被调度的。



