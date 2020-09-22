# python 编码ASCII、Unicode、utf-8

## 1.  python2中我们看到的字符串
```python
#当前系统的默认编码为utf-8
In [39]: locale.getdefaultlocale()
Out[39]: ('en_US', 'UTF-8')
    
In [25]: teststr = '联通'

#在内存中是以utf-8编码存在的
In [26]: teststr
Out[26]: '\xe8\x81\x94\xe9\x80\x9a'

#可以看到字符串为指定编码时，使用的默认编码utf-8
In [27]: chardet.detect(teststr)
Out[27]: {'confidence': 0.7525, 'encoding': 'utf-8', 'language': ''}

In [28]: teststr.encode('utf-8')
Out[28]: '\xe8\x81\x94\xe9\x80\x9a'

#指定为Unicode编码之后，在内存中是以Unicode编码存在的
In [29]: teststr = u'联通'
In [31]: teststr
Out[31]: u'\u8054\u901a'
    
In [41]: print(teststr)
联通

In [32]: teststr.encode('utf-8')
Out[32]: '\xe8\x81\x94\xe9\x80\x9a'
```

综上，平时python代码里面的字符串，在内存中都是以指定的编码存储的； 不指定时，以当前环境的默认编码存储的。



## 2. Unicode编码

 起初由于计算机在美国发明，自然大家考虑的是英语如何表示，英语字母总共26个，加上特殊字符，128个字符，7位既一个byte即可表示出来。这个就是大家所熟知的**ascill编码**。对应关系很简单，一个字符对应一一个byte。

  但很快发现，其他非英语国家的文字远远超过ascill码，这时候大家当然想统一字符编码，不同国家出了自己不同的编码方式，中国的gb2312就是自 己做出来的编码方式，这样下去每个国家都有自己的编码方式，来回转换太麻烦了。这时候出现了新的编码方式，unicode编码方式，想将编码统一，所以规 定了每个字符对应的unicode码。


   Unicode（统一码，万国码）是基于通用字符集（Universal Character Set）的标准发展。它为每种语言中的每个字符设定了统一并且唯一的二进制编码，以满足语言、跨平台进行文本转换、处理的要求。

​      Unicode用数字0到0X10FFFF来映射这些数字，最多容纳1114112个字符。1114112是怎么计算出来的？将0X10FFFF分成0X10和0XFFFF两部分。我们知道0XFFFF是65535，那么 [0,65535] 左右闭区间，总共是65536个。同理，0X10用10进制表示为16，那么 [ 0,16 ] 左右闭区间，总共是17个。所以17乘以65536=1114112.

Unicode编码是一个统一的编码，其他编码类型之间相互转换，都需要先转成Unicode

如UTF8——>Unicode ——>GBK、GB2312



## 3. **Unicode 的问题**

需要注意的是，Unicode 只是一个符号集，它只规定了符号的二进制代码，却没有规定这个二进制代码应该如何存储。

比如，汉字`严`的 Unicode 是十六进制数`4E25`，转换成二进制数足足有15位（`100111000100101`），也就是说，这个符号的表示至少需要2个字节。表示其他更大的符号，可能需要3个字节或者4个字节，甚至更多。

这里就有两个严重的问题，第一个问题是，如何才能区别 Unicode 和 ASCII ？**计算机怎么知道三个字节表示一个符号，而不是分别表示三个符号呢？**第二个问题是，我们已经知道，英文字母只用一个字节表示就够了，如果 Unicode 统一规定，每个符号用三个或四个字节表示，那么每个英文字母前都必然有二到三个字节是`0`，这对于存储来说是极大的浪费，文本文件的大小会因此大出二三倍，这是无法接受的。

它们造成的结果是：1）出现了 Unicode 的多种存储方式，也就是说有许多种不同的二进制格式，可以用来表示 Unicode。2）Unicode 在很长一段时间内无法推广，直到互联网的出现。

**所以出现了utf-8、gbk 等编码方式。**

**如将Unicode以编码方式utf8编码之后存储；在读取的时候，用utf8解码为Unicode显示；**



## 4. 文件的读写

**在计算机内存中，统一使用Unicode编码，当需要保存到硬盘或者需要传输的时候，就转换为UTF-8编码。**

用记事本编辑的时候，从文件读取的UTF-8字符被转换为Unicode字符到内存里，编辑完成后，保存的时候再把Unicode转换为UTF-8保存到文件：

![rw-file-utf-8](I:\笔记\pictures\编码file-utf8.png)



## 5. 为什么会乱码

- 解码方式跟编码方式不一致就会导致乱码， 不涉及到编码、解码就不会报错
  - 写数据库时，如果写如的数据编码跟数据库编码不一样就会报错
  - python的print 在打印字符时，以当前环境默认编码解码
  - 比如从数据库获取的数据是gbk编码，但是代码的默认编码为utf-8，print时会乱码； 最好的办法是入程序时统一转成Unicode
- 平时写程序的时候，好像没有特别注意编码，但是出现乱码很少； 
  - 处理逻辑时，如果不decode\encode就不会报错；当前程序很少decode\encode， 都是读取数据格式化就数据返回了； 读取mongo返回的数据是Unicode

## 6. python2 中的乱码

- python2 中默认的编码是ASCII，需要在首行指定编码方式，未指定的话，很容易出现乱码

## 7. python3 中默认为utf-8

- 确保读入的数据都是utf8编码的即可正确解码

## 8. 数据库的读写

- MySQL

  - charset : utf-8
  - 写数据的时候，数据应用被encode('utf-8')
  - 读取出来的数据，任然是utf-8编码的，要正常显示，需要将终端的编码设置为utf-8
  - 程序处理时，读的数据也是utf-8编码的

- mongodb pymongo

  ```markdown
  MongoDB stores data in BSON format. BSON strings are UTF-8 encoded so PyMongo must ensure that any strings it stores contain only valid UTF-8 data. Regular strings (<type ‘str’>) are validated and stored unaltered. Unicode strings (<type ‘unicode’>) are encoded UTF-8 first. The reason our example string is represented in the Python shell as u’Mike’ instead of ‘Mike’ is that PyMongo decodes each BSON string to a Python unicode string, not a regular str.
  ```

  - mongo 以utf8格式存储数据

  - 写数据时，字符串将被直接写入； Unicode编码的，将被首先转换为utf8

  - 读数据时，pymongo将数据转为Unicode编码； 程序内部取数据时候并没有转码，给请求返回的是Unicode的编码

    ```python
    In [58]: data = db.wallpaper_category.find_one({'_id': 17})
    
    In [59]: type(data)
    Out[59]: dict
    
    In [60]: data.keys()
    Out[60]: [u'name', u'tag', u'_id', u'w_type', u'order', u'desc']
    
    In [61]: data.items()
    Out[61]: 
    [(u'name', u'keyboard \u5206\u7c7b1'),
     (u'tag', u'keyboard \u5206\u7c7b1'),
     (u'_id', 17),
     (u'w_type', u'keyboard'),
     (u'order', 1),
     (u'desc', u'keyboard \u5206\u7c7b1')]
    ```

    