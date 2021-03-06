# python内存分配、id()函数的使用

[TOC]

## 1. id函数的使用

- **id()函数可返回对象的内存地址**

python中会为每个对象分配内存，哪怕他们的值完全相等。id(object)函数是返回对象object在其生命周期内位于内存中的地址，id函数的参数类型是一个对象。

```python
name1 = 2
name3 = 2
id(name1)
Out[31]: 140704132617280
id(name3)
Out[32]: 140704132617280
```

name1 == name3 比较的是name1和name3的值是否相等，name1 is name3 则比较的是name1和name3内存（或id)是否一样。

## 2. 为了提高内存利用效率对于一些简单的对象，如一些数值较小的int对象，字符串对象等，python采取重用对象内存的办法

- 如指向name1 =2，name2 =2时，由于2作为简单的int类型且数值小，python不会两次为其分配内存，而是只分配一次，然后将name1 与name2 同时指向已分配的对象。

- 如对于数值较大的int对象，python会为name1 和name2 分别申请一块内存，来存储20000000

```python
name1 = 200
name2 = 200
id(name1)
Out[37]: 140704132623616
id(name2)
Out[38]: 140704132623616
name1 = 20000000
name2 = 20000000
id(name1)
Out[41]: 2220351232464
id(name2)
Out[42]: 2220351233104
name1 == name2
Out[44]: True
name1 is name2
Out[45]: False
```

## 3. list类型浅拷贝时，内存地址一样；深拷贝时，内存地址发生了变化

```python
name1 = [1, 2, 3]
name3 = name1
id(name1)
Out[15]: 2220353242760
id(name3)
Out[17]: 2220353242760
name1
Out[18]: [1, 2, 3]
name3
Out[19]: [1, 2, 3]
name1.append(4)
name3
Out[21]: [1, 2, 3, 4]
name1
Out[22]: [1, 2, 3, 4]
id(name1)
Out[23]: 2220353242760
id(name3)
Out[24]: 2220353242760
name3.append(5)
id(name3)
Out[26]: 2220353242760
id(name1)
Out[27]: 2220353242760
id(name1.copy())
Out[28]: 2220377117384
```

## 4. 内存的分配

- 对于简单类型且值较小的值，如int、str， Python都会缓存这些对象，以便重复使用； 在赋值时：是先将变量值存储到内存单元，然后再将变量名称指向存储单元；多个相同值的变量名，会指向相同的地址；当值发生变化时，内存地址也会发生变化；

- 对于简单类型无论值大小，当值发生变化时，内存地址也会发生变化；

- 对于类、列表等符合类型对象： 即使值一样，变量指向的内存地址也不一样；并且，值发生变化时，内存地址不发生变化

  