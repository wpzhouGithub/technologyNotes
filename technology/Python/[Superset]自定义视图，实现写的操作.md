# Superset 自定义视图，实现写

## 一、背景

- superset数据展现很友好，所以最初选择了这个框架

- 后期需要实现内容配置，即控制台实现写库的操作

综上，需要自定义视图，集成superset实现写的操作

## 二、Version

- 系统：Ubuntu
- python版本：3.6
- superset版本： apache-superset==0.36.0
- conda 虚拟环境

## 三、Installation

### conda 安装

```shell
# 安装miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh 
# source 环境变量
source  ~/.bashrc 
# 默认不启动
conda config --set auto_activate_base false
# 安装python3.6 superset独立环境
conda create -n superset python=3.6
# 激活superset环境
conda activate superset
```

### Superset安装

```shell
pip install apache-superset==0.36.0
```

### Superset升级

```shell
pip install apache-superset --upgrade
superset db upgrade
superset init
```

如果已有数据，可以备份数据之后直接升级

## 四、实现

1. 尽量不修改superset的源码；但是FAB的update_perms被默认为False，这个必须要改一下，不然新增加的视图无法显示

```python
        # appbuilder.update_perms = False
        appbuilder.update_perms = True
```

2. 新增superset_config.py文件，作为配置文件；与原有的默认配置config.py分开； 

```python
#config.py
"""All configuration in this file can be overridden by providing a superset_config
in your PYTHONPATH as there is a ``from superset_config import *``
at the end of this file.
"""
#在os环境变量里面配置路径
if CONFIG_PATH_ENV_VAR in os.environ:
    # Explicitly import config module that is not necessarily in pythonpath; useful
    # for case where app is being executed via pex.
    try:
        cfg_path = os.environ[CONFIG_PATH_ENV_VAR]
        module = sys.modules[__name__]
        override_conf = imp.load_source("superset_config", cfg_path)
        for key in dir(override_conf):
            if key.isupper():
                setattr(module, key, getattr(override_conf, key))

        print(f"Loaded your LOCAL configuration at [{cfg_path}]")
    except Exception:
        logger.exception(
            f"Failed to import config for {CONFIG_PATH_ENV_VAR}={cfg_path}"
        )
        raise
#让文件位于python path，可以直接import
elif importlib.util.find_spec("superset_config"):
    try:
        from superset_config import *  # pylint: disable=import-error,wildcard-import,unused-wildcard-import
        import superset_config  # pylint: disable=import-error

        print(f"Loaded your LOCAL configuration at [{superset_config.__file__}]")
    except Exception:
        logger.exception("Found but failed to import local superset_config")
        raise
```
   - 满足以上2个条件之一，superset会自动去获取superset_config的配置，覆盖原有的config

```python
#superset_config.py
"""All configuration in this file overwrite config.py
"""
from data_report_admin import CustomAppInitializer

# 默认数据源
SQLALCHEMY_DATABASE_URI = "sqlite:////data/.superset_new/superset.db"
# 用于配置多个数据源
SQLALCHEMY_BINDS = {
    'mysql_datareport': 'mysql://user:password@localhost/data_report',
}
# app初始化类，重写了post_init方法；用来添加自定义view
APP_INITIALIZER = CustomAppInitializer
SUPERSET_WEBSERVER_PORT = 8089
# 控制台显示对应的视图，0.36.0以后默认不显示了
FAB_ADD_SECURITY_PERMISSION_VIEW = True
FAB_ADD_SECURITY_VIEW_MENU_VIEW = True
FAB_ADD_SECURITY_PERMISSION_VIEWS_VIEW = True
```

3. 自定义视图
   - 初始化
```python
#data_report_admin.__init__.py
import logging

from flask import Flask
from superset.extensions import appbuilder
from superset.app import SupersetAppInitializer

from .views.crawler_settings import AdmobAdIdsView


#后期新增视图，直接添加即可；不需要再修改superset相关的设置；
def init_views():
    """
    init datareport admin views
    :param appbuilder:
    :return:
    """
    logging.info('init_view in!')
    appbuilder.add_view(AdmobAdIdsView, name='admob广告位ID配置', category='爬虫配置')


class CustomAppInitializer(SupersetAppInitializer):
    def __init__(self, app: Flask):
        super().__init__(app)

    def post_init(self) -> None:
        """
        Called after any other init tasks
        """
        with self.flask_app.app_context():
            init_views()
```

   - view
```python
#data_report_admin.views.crawler_settings.py
from flask_appbuilder.models.sqla.interface import SQLAInterface
from flask_appbuilder import ModelView

from ..models.crawler_settings import AdmobAdIds


class AdmobAdIdsView(ModelView):
    datamodel = SQLAInterface(AdmobAdIds)
    label_columns = {'app': '应用名称', 'ad_unit_id': '广告位id'}
    list_columns = add_columns = edit_columns = ['app', 'ad_unit_id']
    search_columns = ['app']

```
   - model
```python
#data_report_admin.models.crawler_settings.py
from sqlalchemy import Column, String
from flask_appbuilder import Model


class AdmobAdIds(Model):
    __tablename__ = 'app_ad_ids_as'
    #有多个源的时候，非默认的需要明确指定数据源
    __bind_key__ = 'mysql_datareport'
    app = Column(String, primary_key=True)
    ad_unit_id = Column(String, primary_key=True)

    def __repr__(self):
        return self.app

```

## 五、填的坑

1. python3.6 默认安装apache-superset==0.35.2， 该版本不是最新的；初始化的源码结构没有0.36.0好；
2. 所读源码跟线上不是统一版本，将线上升级到0.36.0；升级方法，上文有提
3. 根据flask appbuild的文档，开发好视图；调用superset的appbuilder.add_view， 视图并没有显示；需要设置appbuilder.update_perms = True
   - 手动拼链接，被重定向到login页面；因为是登陆状态，进而又被重定向到welcome页面； 这样看来，推测是权限问题导致的；
   - 追溯appbuild的模板，发现在渲染模板的时候，有判断权限；无权限，页面将不显示对应的视图；将权限check 注释之后，页面显示了新增视图；打开的时候任然提醒权限； 所以，就是权限问题导致的
   - flask appbuild官网说增加视图之后，会自动加到权限控制，失去了方向；
   - 追溯add_view代码，发现有一个判断，决定是否要增加视图到权限控制;  是的，就是update_perms 参数惹的祸，它决定了要不要更新权限；

```python
    def _add_permission(self, baseview, update_perms=False):
        if self.update_perms or update_perms:
            try:
                self.sm.add_permissions_view(
                    baseview.base_permissions, baseview.class_permission_name
                )
            except Exception as e:
                log.exception(e)
                log.error(LOGMSG_ERR_FAB_ADD_PERMISSION_VIEW.format(str(e)))
```

综上，把坑填了，自定义视图写操作还是比较容易的，哈哈！