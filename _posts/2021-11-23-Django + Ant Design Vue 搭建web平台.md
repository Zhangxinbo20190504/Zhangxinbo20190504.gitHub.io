---
layout:     post
title:      "Django + Ant Design Vue 搭建web平台"
subtitle:   "前后端整合"
date:       2021-11-23 17:00:00
author:     "Xinbo"
header-img: "img/post-bg-universe.jpg"
tags:
    - Django
    - Vue
---

# Django + Ant Design Vue 搭建web平台——前后端整合

* 系统 ubuntu2004
* Docker内搭建（可选）

## Django安装与初始化

### 本地安装

安装django

> 自行安装python3 及 python3-pip，本环境python版本3.8.10，建议版本3.6及以上

```bash
pip3 install django
```

检查django版本

``` bash
Python 3.8.10 (default, Sep 28 2021, 16:10:42)
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import django
>>> django.get_version()
'3.2.8'
```

新建项目

``` bash
python3 -m django startproject mysite
```

初始化结构如下

> README.md不是该命令生成，自己添加

``` 

├── README.md
├── manage.py
└── mysite
    ├── __init__.py
    ├── asgi.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

启动

> * 可以不使用 <your_id>:<your_port> ，直接python3 manage.py runserver，直接访问 http://127.0.0.1:8000/
> * 如果指定ip，需要修改 settings.py ，在 ALLOWED_HOSTS 列表中添加指定的ip

``` bash
python3 manage.py runserver <your_id>:<your_port>
```

启动成功后访问 http://<your_id>:<your_port>/，正常情况下可以看到一个小火箭

###  docker内安装 

> 基础镜像为docker hub获取的ubuntu2004

运行docker

``` bash
docker run -it -v <code_path>:<code_path> -p 80:8001 <images> /bin/bash
```

>  通过-p映射docker内端口
>
> docker内安装django方法同本地

在docker内运行

``` bash
cd <code_path>
python3 manage.py runserver 0.0.0.0:8001
```

在浏览器访问 <宿主机ip>:80 ，效果与本地运行一致



> 默认接下来的操作均在docker 中进行

## Ant Design Vue环境搭建

### 安装构建工具npm/cnpm

安装nodejs

> 安装过程中可能会选地区，Asia Shanghai

``` bash
apt install nodejs
```

检查node版本

``` bash
node --version
v10.19.0
```

安装npm

``` bash
apt install npm
```

检查npm版本

``` bash
npm --version
6.14.4
```

使用国内淘宝仓库

``` bash
npm config set registry https://registry.npm.taobao.org
```

node升级

> 当出现npm不支持node版本的情况下

``` bash
npm --version
npm WARN npm npm does not support Node.js v10.19.0
```

执行

``` bash
npm install n -g
n latest
```

检查版本

``` bash
node -v && npm -v
v16.11.0
8.0.0
```

### 前端代码构建

克隆 ant design vue pro的代码

``` bash
git clone --depth=1 https://github.com/vueComponent/ant-design-vue-pro.git vue-frontend
```

npm升级

``` bash
npm install -g npm
```

执行install

> 执行完之后可以看到文件夹下新增了node_modules文件夹及package-lock.json文件
>
> node_modules文件夹里是安装的所有依赖库，比较大，如果前端是用git管理建议在.gitignore文件中过滤

``` bash
npm install
```

启动

``` 
npm run serve
```

访问端口可以看到ant design vue pro的界面，环境搭建成功

## 前后台整合

将 vue-frontend 放到与 mysite 同级，结构如下

> 如果后台使用git管理，可在.gitignore中将vue-frontend过滤

``` bash
./
├── README.md
├── db.sqlite3
├── manage.py
├── mysite
└── vue-fronted  # 前端文件
```

vue-fronted/vue.config.js修改

> 默认 npm run build会将css和js文件夹生成在dist目录下，后续部署静态资源全部从static下获取，为了统一，增加如下配置，让css和js生成在 dist/static 下

``` javascript
// vue.config.js
const vueConfig = {
  // npm run build dist dir config
  runtimeCompiler: true,
  publicPath: '/',
  outputDir: 'dist',
  assetsDir: 'static',
  indexPath: 'index.html',
  filenameHashing: true,
  configureWebpack: {
    ...
  },
```

执行

``` bash
npm update
npm run build
```

生成静态文件夹dist

mysite/settings.py 添加template配置

```python
TEMPLATES = [
    {
        ...
        # 增加static路径及前端index.html路径
        'DIRS': [os.path.join(BASE_DIR, 'static'), os.path.join(BASE_DIR, 'vue-fronted/dist')],
        ...
    }
]
```



mysite/settings.py 添加静态文件路由

``` python
import os

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'static/'),
    os.path.join(BASE_DIR, "vue-fronted/dist/static/"),
)
```

mysite/urls.py 添加vue工程入口

> ant design vue pro入口为生成的dist/index.html

``` python
from django.urls import include
from django.conf.urls import url
from django.conf import settings
from django.conf.urls.static import static
from django.views.generic.base import TemplateView

urlpatterns = [
    # 后台 API Here
    # vue入口
    url(r'^', TemplateView.as_view(template_name="index.html")),
] + static(settings.STATIC_URL, document_root=settings.STATICFILES_DIRS)
```

启动 Django

``` bash
python3 manage.py runserver 0.0.0.0:9997
```

通过访问 9997 端口看到 ant design vue pro的界面，则前后台整合成功



## 官方文档/参考资料

* Django官方文档：https://docs.djangoproject.com/zh-hans/3.2/
* Ant Design Vue Pro：https://pro.antdv.com/docs/getting-started
* Ant Design Vue：https://1x.antdv.com/docs/vue/introduce-cn/





