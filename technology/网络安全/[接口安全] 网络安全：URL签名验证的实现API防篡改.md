# 网络安全：URL签名验证的实现API防篡改
[TOC]
我们在做APP开发的时候，APP的网络安全是极其重要，我们有必要对请求的API进行加密和防篡改。HTTPS是一个很好的传输加密的方式。如果APP的请求API地址和参数被泄露，我们还是可能会被恶意请求。

所以我们有必要实现 URL签名，对请求的参数进行校验，在客户端生成URL签名，在服务端对签名进行校验，如果客户端的URL签名算法保密做得好，就可以避免非法请求，签名算法的加密需要使用 C++编写，这里就不介绍，只介绍URL签名算法的实现。

## 原理

在 APP 端，对请求的参数+时间戳+密钥，进行MD5生成一个值，并和时间戳、参数一起传到服务器，在服务器也进行相同的方式生成一个MD5，对比两个MD5是否一致，如果一致就说明这个请求是从APP发起的，否则就是非法请求。

## 算法

算法的实现有很多方式，这里介绍一个比较通用的算法。
假设参与参数签名计算的请求参数分别是“k1”、“k2”、“k3”，它们的值分别是“v1”、“v2”、“v3”，则参数签名计算方法如下：

 1. 每一个请求都必须带有tm参数，长度为13的毫秒时间戳；
 2. 将请求参数格式化为“key=value”格式，即“K1=v1”、“K2=v2”、“K3=v3”，key全转为大写
 3. 将格式化好的参数键值对以字典序升序排列后，拼接在一起，即“k1=v1k2=v2k3=v3”；
 4. 在拼接好的字符串末尾追加上一个密钥（需要注意保密，保存在服务端和客户端）； 
 5. 对上述字符串进行MD5，即为签名的值v_sign
 6. 请求的时候加上参数sign=v_sign


## 服务器验证

 1. 对所有接收到的参数也以上面的方式拼接，并加上密钥生成MD5；
 2. 取出 sign 字段进行对比。如果一样就说明没有被篡改，如果不一样就返回错误；
 3. 对参数中的 tm 字段和服务器的时间进行对比，如果误差超过 5分钟就说明请求已经过期，就应该返回错误。

## python 实现

```python

import hashlib

SALT = 'abcdefg'

class urlSign(object):
    salt = SALT

    @classmethod
    def generate_sign(cls, params, salt=None):
        '''
        生成签名
        :param params:
        :param salt:
        :return:
        '''
        print(params)
        if not salt:
            salt = cls.salt

        #获取待签名的字符串
        kvs = []
        for k, v in params.items():
            kv = '='.join([k.upper(), str(v)])
            kvs.append(kv)

        kvs.sort()
        kvs.append(salt)
        sign_str = ''.join(kvs)
        print('sign_str: {0}'.format(sign_str))

        sign = hashlib.md5(sign_str).hexdigest()
        print('sign: {0}'.format(sign))
        return sign

#装饰器
def verification_sign(func):
    """
    验证签名
    """
    @wraps(func)
    def wrapper(*args, **kwargs):
        params = request.args.to_dict()

        #check 请求是否过期
        tm = params.get('tm', None)
        if not tm:
            return json_response({"data": {"error": "no param: " + 'tm'}})
        else:
            now = time.time()
            if now > int(tm) + 60:
                return json_response({"data": {"error": "invalid param: " + 'tm'}})

        #check sign
        sign = params.pop('sign', None)
        if not sign:
            return json_response({"data": {"error": "no param: " + 'sign'}})

        sign_new = urlSign.generate_sign(params)
        if sign != sign_new:
            return json_response({"data": {"error": "invalid param: " + 'sign'}})


        ret = func(*args, **kwargs)
        return ret

    return wrapper
```