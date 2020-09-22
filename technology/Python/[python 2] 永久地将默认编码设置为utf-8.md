# python2.7 永久地将默认编码设置为utf-8

## 网上推荐了两种方法

- 第一个方法<不推荐>: 编辑site.py, 修改setencoding()函数, 强制设置为 utf-8
- 第二个方法<推荐>: 增加一个名为 sitecustomize.py, 推荐存放的路径为 site-packages 目录下
  sitecustomize.py 是在 site.py 被import 执行的, 因为 sys.setdefaultencoding() 是在 site.py 的结尾处被删除的, 所以, 可以在 sitecustomize.py 使用 sys.setdefaultencoding().
  \#file name:  sitecustomize.py
  import sys  
  sys.setdefaultencoding('utf-8')   

既然 sitecustomize.py 能被自动加载,  所以除了设置编码外, 也可以设置一些其他的东西.  

## 实践

- python2.7  sys.setdefaultencoding()报错，说不存在setdefaultencoding
- 但是这个问题大多数在python3 中抛出
- 把site.py 中的del setdefaultencoding的代码注释了任然无效
- 最后直接修改site.py 中setencoding()函数的encoding='utf-8'
- 重启程序，欧克，不再报编码问题

