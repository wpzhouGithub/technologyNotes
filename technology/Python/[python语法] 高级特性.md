# [python语法] python高级特性

[TOC]

## 一、切片

- 切片的正常用法不再赘述，除了正常的切片功能，还可以在切片的基础上每个N个元素取一个；


- 以下是每隔2个元素取一个


```python
l = range(10)
for i in l[::2]:
    print(i)
    
0
2
4
6
8
```



## 二、迭代

用**for遍历**list、tuple等可遍历对象，这种遍历就叫做**迭代**

在Python中，迭代是通过`for ... in`来完成的，而很多语言比如C语言，迭代list是通过下标完成的，比如Java代码：

```Java
for (i=0; i<list.length; i++) {
    n = list[i];
}
```

- python中只要是可迭代对象，都可以通过for来遍历；通过collections模块的Iterable类型判断是否可迭代

```python
>>> from collections import Iterable
>>> isinstance('abc', Iterable) # str是否可迭代
True
>>> isinstance([1,2,3], Iterable) # list是否可迭代
True
>>> isinstance(123, Iterable) # 整数是否可迭代
False
```

- Python内置的`enumerate`函数可以把一个list变成索引-元素对，这样可以同时获取到index和value

```python
>>> for i, value in enumerate(['A', 'B', 'C']):
...     print(i, value)
...
0 A
1 B
2 C
```

## 三、列表生成式

简单的用法就不说了，记录一些特殊的用法

- 多层循环

```python
>>> [m + n for m in 'ABC' for n in 'XYZ']
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```

- ```if ... else```的使用

  1. **在for之前时，必须包含else**。

     因为**`for`前面的部分是一个表达式，它必须根据`x`计算出一个结果**。而表达式：`x if x % 2 == 0`，它无法根据`x`计算出结果，因为缺少`else`，必须加上`else`：
  
  不加`else`报错：

```python
>>> [x if x % 2 == 0 for x in range(1, 11)]
  File "<stdin>", line 1
    [x if x % 2 == 0 for x in range(1, 11)]
                       ^
SyntaxError: invalid syntax
```

```python
>>> [x if x % 2 == 0 else -x for x in range(1, 11)]
[-1, 2, -3, 4, -5, 6, -7, 8, -9, 10]
```

  2. **for之后不用`else`**

     因为**for后面是筛选条件，用来判断当前的x要不要返回**，所以不用else
     
     ```python
     >>> [x for x in range(1, 11) if x % 2 == 0]
     [2, 4, 6, 8, 10]
     ```
     
     

## 四、生成器

- 生成器的典型特点：按照某种算法推算出来，那循环的过程中不断推算出后续的元素；不必创建完整的list，从而节省大量的空间；

- 生成器的创建

  要创建一个generator，**只要把一个列表生成式的`[]`改成`()`**，就创建了一个generator：

  ```python
  >>> L = [x * x for x in range(10)]
  >>> L
  [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
  >>> g = (x * x for x in range(10))
  >>> g
  <generator object <genexpr> at 0x1022ef630>
  ```

  **使用()在列表长度很大的时候，可以节约大量的内存，提高性能**

- 可以直接用for遍历生成器，不一定非要用next； next还要捕获最后一个空元素的异常

  ```python
  >>> g = (x * x for x in range(4))
  >>> for n in g:
  ...     print(n)
  ... 
  0
  1
  4
  9
  ```

- 比如，著名的斐波拉契数列（Fibonacci），除第一个和第二个数外，任意一个数都可由前两个数相加得到：

  1, 1, 2, 3, 5, 8, 13, 21, 34, ...

  斐波拉契数列用列表生成式写不出来，但是，用函数把它打印出来却很容易：

  ```
  def fib(max):
      n, a, b = 0, 0, 1
      while n < max:
          print(b)
          a, b = b, a + b
          n = n + 1
      return 'done'
  ```

  生成器的方式的实现，要把`fib`函数变成generator，只需要把`print(b)`改为`yield b`就可以了：

  ```python
  def fib(max):
      n, a, b = 0, 0, 1
      while n < max:
          yield b
          a, b = b, a + b
          n = n + 1
      return 'done'
  ```

  如果一个函数定义中包含`yield`关键字，那么这个函数就不再是一个普通函数，而是一个generator：

  ```
  >>> f = fib(6)
  >>> f
  <generator object fib at 0x104feaaa0>
  ```

  这里，最难理解的就是generator和函数的执行流程不一样。函数是顺序执行，遇到`return`语句或者最后一行函数语句就返回。**而变成generator的函数，在每次调用`next()`的时候执行，遇到`yield`语句返回，再次执行时从上次返回的`yield`语句处继续执行**。

  但是用`for`循环调用generator时，发现拿不到generator的`return`语句的返回值。**如果想要拿到返回值，必须捕获`StopIteration`错误，返回值包含在`StopIteration`的`value`中**：
  
  ```python
  >>> g = fib(6)
  >>> while True:
  ...     try:
  ...         x = next(g)
  ...         print('g:', x)
  ...     except StopIteration as e:
  ...         print('Generator return value:', e.value)
  ...         break
  ...
  g: 1
  g: 1
  g: 2
  g: 3
  g: 5
  g: 8
  Generator return value: done
  ```

## 五、迭代器

凡是可作用于`for`循环的对象都是`Iterable`（可迭代的）类型；

凡是可作用于`next()`函数的对象都是`Iterator`（迭代器）类型，它们表示一个**惰性计算**的序列；

集合数据类型如`list`、`dict`、`str`等是`Iterable`但不是`Iterator`，不过可以**通过`iter()`函数获得一个`Iterator`对象**。