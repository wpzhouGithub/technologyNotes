## 用户管理
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

