Python 3最重要的新特性之一是对字符串和二进制数据流做了明确的区分。Python 3不会以任意隐式的方式混用str和bytes，你不能拼接字符串和字节流，也无法在字节流里搜索字符串（反之亦然），也不能将字符串传入参数为字节流的函数（反之亦然）。

## unicode、str、bytes的关系

硬盘中一般编码都是uft-8，而在内存中采用unicode编码方式。

python中的str其实显示的就是读取unicode,str的内存格式就是unicode，所以理解为str就是unicode,unicode就是str。

bytes是对Unicode的encode， 如字符串的encode方法就可以将str转为bytes类型；
在bytes和str的互相转换过程中，实际就是编码解码的过程，必须显式地指定编码格式。

## str bytes相互转换
### 1. str bytes方法相互转换
```python
>>> s = "中文"
>>> s
'中文'
>>> type(s)
<class 'str'>
>>> b = bytes(s, encoding='utf-8')
>>> b
b'\xe4\xb8\xad\xe6\x96\x87'   #开头的b表示这是一个bytes类型。\x 代表是十六进制
>>> type(b)
<class 'bytes'>

>>> s1 = str(b)
>>> s1
"b'\\xe4\\xb8\\xad\\xe6\\x96\\x87'"   #变成这样子的串了。
>>> type(s1)
<class 'str'>
>>> s1 = str(b, encoding='utf-8')   #这样是对的
>>> s1
'中文'
>>> type(s1)
<class 'str'>
```
总结：
从str到bytes，编码；从bytes到str，解码

从str到bytes，是编码,   比特流=str(串,encoding='utf-8')
从bytes到str，是解码，串=bytes(比特流,encoding='utf-8')
### 2. encode decode方法相互转换
```python
'''中文编码'''
>>> a='中文'
>>> type(a)
str
>>> b=a.encode(encoding='utf-8')
>>> b
b'\xe4\xb8\xad\xe6\x96\x87'
 
'''中文解码'''
>>> c=b.decode(encoding='utf-8')
>>> c
'中文'
 
'''可以省略解码格式, python3默认utf-8'''
>>> d=b.decode()
>>> d
'中文'
 
'''英文编码'''
>>> a='hello'
>>> b=a.encode(encoding='utf-8')
>>> b
b'hello'
 
'''英文解码'''
>>> c=b.decode('utf-8')
>>> c
'hello'
```