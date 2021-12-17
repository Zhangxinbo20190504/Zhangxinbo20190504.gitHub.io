---
layout:     post
title:      "Django 实现Restful接口"
subtitle:   "django restframework + drf_yasg"
date:       2021-12-02 19:00:00
author:     "Xinbo"
header-img: "img/post-bg-universe.jpg"
tags:
    - Django
---

### 第三方库安装

安装 djangorestframework、drf_yasg

> drf_yasg主要为了能自动生成文档

```shell
pip3 install djangorestframework
pip3 install drf_yasg
```

在 settings.py 文件中将 rest_framework 加入 INSTALLED_APPS

```python
INSTALLED_APPS = [
    # my apps here
    ...
    'drf_yasg',
    'rest_framework',
    ...
]
```

### 新建模型

新建一个app

```shell
python3 manage.py startapp build_info
```

把新建的app加入 INSTALLED_APPS

```python
INSTALLED_APPS = [
    # my apps here
    ...
    'build_info',
    'rest_framework',
  	...
]
```

在 build_info/models.py中新建一个模型

> 因为我已经在数据库建了表，这里只是用Django ORM做映射，所以在meta信息里添加 managed = False ，添加后并不在数据库中实际生成表

```python
from django.db import models

# Create your models here.


class BuildJob(models.Model):
    ''' Jenkins构建任务记录 '''
    project = models.CharField(max_length=255)
    timestamp = models.CharField(max_length=255)
    branch = models.CharField(max_length=255)
    workspace = models.CharField(max_length=500)
    job_name = models.CharField(max_length=500)
    build_url = models.CharField(max_length=500)
    build_num = models.CharField(max_length=255)
    build_node = models.CharField(max_length=255)

    class Meta:
        db_table = 'build_job'
        managed = False
```

执行数据迁移

```shell
python3 manage.py makemigrations build_info
python3 manage.py migrate build_info
```

### 创建序列化类

在build_info文件夹下新建 serializers.py 文件，该文件用来存放序列化类，管理序列化/反序列化的json数据

在内置类Meta，我们定义了两个属性：

- model：用于序列化的模型
- fields：元组类型，需要序列化的字段名称

> id字段是Django自动生成，也需要包含在字段中

> 可以通过extra_kwargs对字段进行补充说明，显示在API文档中，更加友好

```python
from rest_framework import serializers 
from models import BuildJob

class BuildJobSerializer(serializers.ModelSerializer):

    class Meta:
        model = BuildJob
        fields = ('id', 'project', 'timestamp', 'branch', 'workspace', 'job_name', 'build_url', 'build_num', 'build_node')
        
```

### 配置API文档

在工程 url.py 下 新增

```python
from django.conf.urls import include, url

from rest_framework import permissions
from drf_yasg import openapi
from drf_yasg.views import get_schema_view

SchemaView = get_schema_view(
    openapi.Info(
        title="Your Title",
        default_version='v1.0',
        description="""
            Your Description
        """,
        contact=openapi.Contact(email="Youremail@email.com"),
        license=openapi.License(name="BSD License"),
    ),
    public=True,
    # schema view本身的权限类
    permission_classes=(permissions.AllowAny,),
)

urlpatterns = [
    url(r'^swagger(?P<format>.json|.yaml)$', SchemaView.without_ui(cache_timeout=0), name='schema-json'),  # 导出
    url(r'^redoc/$', SchemaView.with_ui('redoc', cache_timeout=0), name='schema-redoc'),  # redoc美化UI
    url(r'^docs/$', SchemaView.with_ui('swagger', cache_timeout=None), name='cschema-swagger-ui'),
]
```

在 build_info 文件夹下 新增 urls.py，并在工程urls.py中加入inclue

```python
urlpatterns = [
    url(r'^api/', include('build_info.urls')),
    url(r'^swagger(?P<format>.json|.yaml)$', SchemaView.without_ui(cache_timeout=0), name='schema-json'),  # 导出
    url(r'^redoc/$', SchemaView.with_ui('redoc', cache_timeout=0), name='schema-redoc'),  # redoc美化UI
    url(r'^docs/$', SchemaView.with_ui('swagger', cache_timeout=None), name='cschema-swagger-ui'),
]
```

在 build_info 中的 views.py 新增API视图

```python
from drf_yasg import openapi
from drf_yasg.utils import swagger_auto_schema
from rest_framework.viewsets import ViewSet

from .models import BuildJob
from .serializers import BuildJobSerializer

class BuildJobView(ViewSet):
    @swagger_auto_schema(
    	# API描述
        operation_description="list all build jobs",
        # tag便于分组 summary显示信息
        security=[], tags=tags, operation_summary='列出所有构建任务',
        # 定义response样式
        responses={
            200: openapi.Response(description='ok', examples={
                'application/json': {
                    'has_result': True,
                    'data': [
                        {
                            # 定义response
                        }
                    ]
                }
            })
        }
    )
    def get_all_build_jobs(self, request):
        jobs = BuildJob.objects.all()
        if jobs:
        	# 调用序列化类进行序列化
            jobs_serializer = BuildJobSerializer(jobs, many=True)
            return JsonResponse({"has_result": True, 'data': jobs_serializer.data}, safe=False)
        return JsonResponse({"has_result": False, 'data': ''})

```



使用 runserver 访问 ip:端口号/redoc 查看效果

### Q：403 CSRF cookie not set 问题

当启用 DRF 后，访问后台接口可能出现 403 CSRF cookie not set 的报错，即使将Django CSRF 中间件注释掉也无法解决，这是因为 restframework 默认采用的Session认证会强制校验 CSRF，需要在settings.py中做如下定义

```python
# *RestFramework采用token，避免CSRF强制验证
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.TokenAuthentication',
    )
}
```

即可正确请求API



### 资料

* 跨域支持请参考 [Django + Ant Design Vue 搭建web平台](https://zhangxinbo20190504.github.io/2021/11/23/Django-+-Ant-Design-Vue-%E6%90%AD%E5%BB%BAweb%E5%B9%B3%E5%8F%B0/)
* Django DRF [文档](https://drf-yasg.readthedocs.io/en/stable/)

