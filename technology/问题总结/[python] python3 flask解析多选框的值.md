# python3 flask解析多选框的值

## 结论

- ImmutableMultiDict 使用getlist 取多个相同key的值

```python
from werkzeug.datastructures import ImmutableMultiDict

d = ImmutableMultiDict([('comment', 'test'), ('wp_ids', '2233'), ('wp_ids', '2341')])
print(d['wp_ids'])
print(d.get('wp_ids'))
print(d.getlist('wp_ids'))

#'2233'
#'2233'
#['2233', '2341']
```



## 思路总结

request.form解析多选框值

1. debug时，发现form的wp_ids值有一个值
2. 浏览器 network check上传时，传了多个值，只是都传在一个key下
3. 推测框架测相同key的多个值，都只解析成dict的一个key的值，即最后的那个值
4. debug check生成request的逻辑，并没有额外的处理逻辑，只是将uwsgi的environ 赋值了
5. check了request可能的属性，没有发现有哪个数据包含多个值；
6. 为了确定不因为漏掉某个属性没有check，将整个request对象copy 成文本到本地，查找另外的那个wp_id; 结果居然找到了，而且还是在之前已经check过的属性里面；
7. 原因是check的属性是ImmutableMultiDict， 它格式化显示的时候，把相同的key合并update了；但是实际上是含有重复key的数据的；```ImmutableMultiDict([('comment', 'test'), ('wp_ids', '2233'), ('wp_ids', '2341')])```
8. 接下来就好办了，想办法取值就行了；直接读ImmutableMultiDict源码没有发现方法，Google了一下，有人提到getlist方法，尝试之后成功了，取出了list
9. 之前有遇到python2/3 取值不兼容的问题，2返回list，3返回单个值； 那现在就可以统一用getlist取出list，统一处理了