1. 需要使用的类赋值给变量，不要直接在代码里面调用；这样方便后期维护修改

```python
class SecurityManager(BaseSecurityManager):
    """
        Responsible for authentication, registering security views,
        role and permission auto management

        If you want to change anything just inherit and override, then
        pass your own security manager to AppBuilder.
    """

    user_model = User
    """ Override to set your own User Model """
    role_model = Role
    """ Override to set your own Role Model """
    permission_model = Permission
    viewmenu_model = ViewMenu
    permissionview_model = PermissionView
    registeruser_model = RegisterUser
```

2. 使用三方类比较多的，可基于三方类自定义一个base类； 优点：
   - 后期如果三方类的使用方法有变或者想换一个其他的三方类使用，只要修改自定义的类即可，不用每个调用的地方都改；
   - 后期想要重写三方类的某个方法
   - 

```
add_permission_view_menu
add_permissions_menu
add_permissions_view
```