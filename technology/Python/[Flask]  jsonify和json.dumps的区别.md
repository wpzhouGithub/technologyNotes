# [Flask]  jsonify和json.dumps的区别

flask提供了jsonify函数供用户处理返回的序列化json数据，而python自带的json库中也有dumps方法可以序列化json对象，那么在flask的视图函数中return它们会有什么不同之处呢？

## 一、实验查看返回结果

- 请注意观察`jsonify()`和`json.dumps()`响应请求的`response headers`中的`content-type`;
- jsonify()的是application/json
- json.dumps()的是text/html

### 1. 看看jsonify()的返回

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
![在这里插入图片描述](../../pictures/jsonify.png)
![在这里插入图片描述](../../pictures/jsonify1.png)

### 2. json.dumps的返回

- 现在我们改为使用python自带的json库json.dumps作为视图函数的直接返回值，代码如下：

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
用chrome访问的响应页面如下图。
![在这里插入图片描述](../../pictures/jsondumps.png)
![在这里插入图片描述](../../pictures/jsondumps1.png)

可以看到content-type 为text/html， 为什么不是别的呢？

这是因为flask会判断并使用make_response方法自动构造出响应，只不过响应头各个字段是默认的，这里content-type默认为text/html。若要自定义响应字段，则可以使用make_response或Response自行构造响应

### 3. 使用json.dumps返回`content-type:application/json`的

```python
import json
from flask import make_response, Flask


app = Flask(__name__)

def json_response(obj):
    response = make_response(json.dumps(obj))
    response.headers['Content-Type'] = 'application/json; charset=utf-8'
    response.headers['Access-Control-Allow-Origin'] = '*'
    response.headers['Access-Control-Allow-Methods'] = 'OPTIONS,HEAD,GET,POST'
    response.headers['Access-Control-Allow-Headers'] = 'x-requested-with'
    return response

@app.route('/hello/<name>/<words>',methods=['GET'])
def hello(name,words):
    return json_response({'name':name,'words':words})
 
if __name__ == '__main__':
    app.run()
```

## Content-Type

jsonify的作用实际上就是将我们传入的json形式数据序列化成为json字符串，作为响应的body，并且设置响应的Content-Type为application/json，构造出响应返回至客户端。jsonify的部分源码如下：
```python
def jsonify(*args, **kwargs):
    if __debug__:
        _assert_have_json()
    return current_app.response_class(json.dumps(dict(*args, **kwargs),
        indent=None if request.is_xhr else 2), mimetype='application/json')
```
可以看出jsonify实际上也是使用了json.dumps来序列化json形式的数据，作为响应正文返回。indent表示json格式化的缩进，若是Ajax请求则不缩进（因为一般Ajax数据没必要直接展示），否则缩进2格。但想必从第一部分的实验结果我们已经看出来了，使用jsonify时响应的Content-Type字段值为application/json，而使用json.dumps时该字段值为text/html。Content-Type决定了接收数据的一方如何看待数据，如何处理数据，如果是application/json，则可以直接当做json对象处理，若是text/html，则还要将文本对象转化为json对象再做处理（个人理解，有误请指正）。

## 接受参数有区别

jsonify可以接受和python中的dict构造器同样的参数，如下图。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200309163831874.png)
而json.dumps比jsonify可以多接受list类型和一些其他类型的参数。但我试了一下，形式为key1=value1，[key2=value2,...]这样的参数是不行的，会报出“TypeError: dumps() takes exactly 1 argument (0 given)”这一错误，而jsonify不会报错并能正常返回数据。

