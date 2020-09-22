## 1. 环境安装
     - conda 安装虚拟环境
     - 在虚拟环境中运行官网提供的命令即可
```   
# Install superset
pip install apache-superset

# Initialize the database
superset db upgrade

# Create an admin user (you will be prompted to set a username, first and last name before setting a password)
$ export FLASK_APP=superset
flask fab create-admin

# Load some data to play with
superset load_examples

# Create default roles and permissions
superset init

# To start a development web server on port 8088, use -p to bind to another port
superset run -p 8088 --with-threads --reload --debugger
```
## 2. 权限控制
- 自带了5中权限
  
    ```
    1. 管理员：拥有所有可能的权限，包括授予或撤消其他用户的权限以及更改其他人的内容和仪表板。
    
    2. Alpha：有权访问所有数据源，但不能授予或撤消其他用户的访问权限。Alpha用户可以添加和更改数据源，但仅限于更改其拥有的对象。
    
    3. Gamma：访问权限受到限制。他们是内容的消费者。只能使用来自其他角色可以访问的数据源的数据及对应数据源制成的切片和仪表板。尽管他们可以创建切片和仪表板；
    
    4. sql_lab： 该角色授予对SQL Lab的访问权限。Admin 默认情况下用户可以访问所有数据库；但是Alpha和Gamma用户默认是没有访问权限的
    
    5. public: 可以允许注销的用户访问某些Superset功能。
      
    ```
    
- 
    某个数据的权限控制

```
    1. 利用gamma角色+其他的角色来控制；
    2. 增加一个其他的角色，该角色赋予访问某些表、数据库的访问权限
```

- 权限的种类
```
    1. model: 主要控制dashboard
    2. view: 主要控制各种功能，比如能否导出csv、能否使用SQL LAB
    3. data sourse：控制数据源的权限
    4. database：控制数据库、表的权限； 注意，如果赋予了某个表的权限，那么基于该表生成的chart、dashboard都将有权访问
```

综上，当前权限无法具体到某个投放用户的数据，因为不同的投放用户的数据是一张表；如果后期非要实现这块的控制，将会分表控制；

## 3. filter box使用
1. 可以使用多列组合过滤
2. table 需要选择可以使用filter
3. 在使用filter box过滤时，整个页面的多个chart都会只显示过滤之后的数据


## 4. 使用query生成chart
1. 保存query； 然后基于query生成chart；

2. 可以用这种方法，对同一table生成多个query；

3. 然后基于不同的query生成chart并控制权限
    所以或许可以通过区分query，达到权限控制不同的dashboard；

  具体步骤：

1. 新建query

2. query的result点击explore，可以看到创建chart的页面，创建chart即可

## 5. 指标的计算
在metrics那新增需要计算的列即可

![图片](https://uploader.shimo.im/f/V6pGaXoecWM3QRj5.png!thumbnail)

## 6. 日期的使用
  1. 标识时间的列在数据库里面设置成时间的类型
  2. 后期肯有用的点
  3. 显示数值时的格式，如1.4K，关注D3.format [https://github.com/d3/d3-format/blob/master/README.md#format](https://github.com/d3/d3-format/blob/master/README.md#format)

## 7. 修改filter的默认值

1. 编辑chart之后，修改json配置的url_params字段；
2. 在dashboard中，删除filter再重新添加才能生效

