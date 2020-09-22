# flask rbac学习调研

## 1. 可提供的主要功能

跟flask一起使用，通过装饰器控制接口的访问权限

1. 角色、父角色、用户的定义
2. 通过装饰器控制接口的访问权限（允许哪些角色使用）
3. 需要自己实现注册、登录、登录之后的token验证，从而对current_user赋值；可以使用flask-login
4. 未提供控制台的功能

## 2. 当前可以用来做什么
  1. 前后端分离
  2. 可以实现接口的分权限调用
  3. 可以通过接口直接render_template
## 3. 整体流程
  1. 使用ORM定个角色、父角色、用户，存数据库
  2. flask init_app时，会注册一个事件：app.before_request(self._authenticate) 
  3. _authenticate 是用来验证权限的
  4. 在写接口时，使用allow、deny等装饰器，对接口设置权限
  5. 用户登录时，会对current_user赋值
    6. 请求发生时，_authenticate 解析当前的用户，进行权限判断
