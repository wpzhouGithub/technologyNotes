# flask  appbuild使用笔记

### 用户管理
```shell
#切换到目录 cd examples/quickhowto2
set FLASK_APP=app
#新建管理员账号
fabmanager create-admin
#重置密码
fabmanager reset-password

```

###  数据结构创建

- 表不存在时，调用`db.create_all()` 创建表结构

### MasterDetailView指定显示的列

```python
class GroupMasterView(MasterDetailView):
    datamodel = SQLAInterface(ContactGroup)
    #指定导航显示的列
    list_columns = ['name']
    related_views = [ContactGeneralView]
```

### 实现关联配置

flask app build 在实现关联的时候，view中配置的可编辑字段应该是关联model的字段；关联外键时，直接的ForeignKey应该是对应表.column，不是model的小写，如果表名跟model名不一样的时候，需要注意

```python
class TokenFbSpendModel(Model):
    """
    Facebook支出token配置
    """
    #有多个源的时候，非默认的需要明确指定数据源
    __bind_key__ = _DB_DATAREPORT
    __tablename__ = 'conf_fb_spend_token'
    machine_id = Column(Integer, ForeignKey('conf_machine.id'), nullable=False)
    machine = relationship("MachineModel")
    user_id = Column(String, primary_key=True)
    token = Column(String, nullable=False)
    bm_name = Column(String, nullable=False)
    is_delete = Column(Integer)
    
class TokenFbRevenueView(ModelView):
    """
    Facebook收入token配置
    每个应用配置
    """
    datamodel = SQLAInterface(TokenFbRevenueModel)

    label_columns = {
        'machine': '运行该爬虫的机器',
        'property_id': '资产ID',
        'app_id': '应用ID',
        'app_name': '应用缩写，如3DLW',
        'token': 'token',
        'is_delete': '是否弃用',
    }
    list_columns = add_columns = edit_columns = [
        'machine',
        'property_id',
        'app_id',
        'app_name',
        'token',
        'is_delete',
    ]
    search_columns = [
        'machine',
        'app_name',
    ]    
```

