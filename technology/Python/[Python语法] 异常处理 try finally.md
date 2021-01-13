# 异常处理 try except else finally

- python中try/except/else/finally语句的完整格式如下所示：

```python
try:

   Normal execution block

except A:
   Exception A handle

except B:

   Exception B handle

except:

   Other exception handle

else:
   if no exception,get here

finally:
   print("finally")  
```

- 正常执行的程序在try下面的Normal execution block执行块中执行，在执行过程中如果发生了异常，则中断当前在Normal execution block中的执行跳转到对应的异常处理块中开始执行；

- python 从第一个except X处开始查找，如果找到了对应的exception类型则进入其提供的exception handle中进行处理，如果没有找到则直接进入except块处进行处理。except块是可选项，如果没有提供，该exception将会被提交给 python进行默认处理，处理方式则是终止应用程序并打印提示信息；

- **如果在Normal execution block执行块中执行过程中没有发生任何异常，则在执行完Normal execution block后会进入else执行块中（如果存在的话）执行**。

- 无论是否发生了异常，只要提供了finally语句，以上try/except/else/finally代码块执行的最后一步总是执行finally所对应的代码块。

  

- 需要注意的是：

1. 在上面所示的完整语句中try/except/else/finally所出现的顺序必须是try-->except X-->except-->else-->finally，即**所有的except必须在else和finally之前**，**else（如果 有的话）必须在finally之前，而except X必须在except之前**。否则会出现语法错误。
2. 对于上面所展示的**try/except完整格式而言，else和finally都是可选的，而不是必须的**，但是如果存在的话else必须在finally之前，finally（如果存在的话）必须在整个语句的最后位置。
3. 在上面的完整语句中，else语句的存在必须以except X或者except语句为前提，如果在没有except语句的try block中使用else语句会引发语法错误。也就是说else不能与try/finally配合使用。
4. except的使用要非常小心，慎用。