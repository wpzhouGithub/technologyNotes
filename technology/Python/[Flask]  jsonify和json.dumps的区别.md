

flask提供了jsonify函数供用户处理返回的序列化json数据，而python自带的json库中也有dumps方法可以序列化json对象，那么在flask的视图函数中return它们会有什么不同之处呢？想必开始很多人和我一样搞不清楚，只知道既然框架提供了方法就用，肯定不会错。但作为开发人员，我们需要弄清楚开发过程中各种实现方式的特点和区别，这样在我们面对不同的需求时才能做出相对合理的选择，而不是千篇一律地使用自己熟悉的。下面我就jsonify和json.dumps的区别这一问题简单探讨一下。



一、实验

python的flask框架为用户提供了直接返回包含json格式数据响应的方法，即jsonify，在开发中会经常用到。如下一段简单的flask后端代码，服务端视图函数根据请求参数返回json格式的数据到客户端。

```python
from flask import Flask
from flask import jsonify
from flask import Response
 
 
app = Flask(__name__)
 
 
@app.route('/hello/<name>/<words>',methods=['GET'])
def hello(name,words):
    return jsonify({'name':name,'words':words})#也可以传入key=value形式的参数，如jsonify(name=name,words=words)
 
 
if __name__ == '__main__':
    app.run()
```




用chrome浏览器访问得到的页面如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200309163636262.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200309163647627.png)






现在我们改为使用python自带的json库json.dumps作为视图函数的直接返回值，代码如下：
```python
from flask import Flask
from flask import jsonify
from flask import Response
 
 
app = Flask(__name__)
 
 
@app.route('/hello/<name>/<words>',methods=['GET'])
def hello(name,words):
    return json.dumps({'name':name,'words':words})
 
 
if __name__ == '__main__':
    app.run()
```
PS:直接返回json.dumps的结果是可行的，因为flask会判断并使用make_response方法自动构造出响应，只不过响应头各个字段是默认的。若要自定义响应字段，则可以使用make_response或Response自行构造响应。用chrome访问的响应页面如下图。
![在这里插入图片描述](https://img-blog.csdnimg.cn/202003091637268.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020030916373785.png)
二、分析

1.Content-Type有区别

jsonify的作用实际上就是将我们传入的json形式数据序列化成为json字符串，作为响应的body，并且设置响应的Content-Type为application/json，构造出响应返回至客户端。jsonify的部分源码如下：
```python
def jsonify(*args, **kwargs):
    if __debug__:
        _assert_have_json()
    return current_app.response_class(json.dumps(dict(*args, **kwargs),
        indent=None if request.is_xhr else 2), mimetype='application/json')
```
可以看出jsonify实际上也是使用了json.dumps来序列化json形式的数据，作为响应正文返回。indent表示json格式化的缩进，若是Ajax请求则不缩进（因为一般Ajax数据没必要直接展示），否则缩进2格。但想必从第一部分的实验结果我们已经看出来了，使用jsonify时响应的Content-Type字段值为application/json，而使用json.dumps时该字段值为text/html。Content-Type决定了接收数据的一方如何看待数据，如何处理数据，如果是application/json，则可以直接当做json对象处理，若是text/html，则还要将文本对象转化为json对象再做处理（个人理解，有误请指正）。


2.接受参数有区别

jsonify可以接受和python中的dict构造器同样的参数，如下图。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200309163831874.png)
而json.dumps比jsonify可以多接受list类型和一些其他类型的参数。但我试了一下，形式为key1=value1，[key2=value2,...]这样的参数是不行的，会报出“TypeError: dumps() takes exactly 1 argument (0 given)”这一错误，而jsonify不会报错并能正常返回数据。

最后，我们可以使用flask中的make_response方法或者直接通过Response类，通过设置mimetype参数来达到和使用jsonify差不多的效果，但少写点代码何乐而不为呢？况且简洁一点更不容易出错，参数越多调试和维护就越麻烦。当然，使用哪个并不是绝对的，必要时要根据前端的数据处理方式来决定。



更多关于jsonify的知识请参考官方文档：http://flask.pocoo.org/docs/0.12/api/#module-flask.json

更多关于json.dumps的知识参考官方文档：https://docs.python.org/2/library/json.html#module-json