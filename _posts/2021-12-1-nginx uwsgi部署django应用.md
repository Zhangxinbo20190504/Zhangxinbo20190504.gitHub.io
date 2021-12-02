---
layout:     post
title:      "Django应用部署"
subtitle:   "nginx + uwsgi"
date:       2021-12-01 19:00:00
author:     "Xinbo"
header-img: "img/post-bg-universe.jpg"
tags:
    - Django
---

### 安装uwsgi

```shell
sudo apt-get install uwsgi
```

创建 uwsgi.ini

```ini
[uwsgi]
chdir=/opt/website/ci_backend   # 项目路径
module=mysite.wsgi:application
# wsgi-file = /opt/website/ci_backend/mysite/wsgi.py
# env=DJANGO_SETTINGS_MODULE=mysite.settings
env=LANG=en_US.UTF-8
master=True
pidfile=/opt/pid/project-master.pid   # 要保证目录存在且用户有写权限
socket=172.16.20.200:8888    # socket 监听端口，以后nginx监听的端口
http=172.16.20.200:9999
processes=5
#uid=0
#gid=0
harakiri=1800
post-buffering = 65536
buffer-size = 65536
max-requests=5000
vacuum=True
daemonize=/opt/log/uwsgi/ci_backend.log  # log路径 要保证目录存在且用户有写权限
socket-timeout=1200
enable-threads=True
```

启动命令

``` shell
uwsgi --ini uwsgi.ini
```

查看端口监听

```shell
sudo lsof -i -P -n | grep LISTEN
```

* **问题：日志中出现 no request plugin is loaded, you will not be able to manage requests**

  安装插件 uwsgi-plugin-python3 或者 uwsgi-plugin-all

  ```shell
  sudo apt install uwsgi-plugin-python3
  ```

  如果还不生效可指定 plugin

  ```shell
  uwsgi --ini uwsgi.ini --plugin python3
  ```

### 安装nginx

安装nginx

```shell
sudo apt-get install nginx
```

在 /etc/nginx/conf.d 中新增文件 django.conf

```
server {
        listen       80;   # 访问的端口
        server_name  172.16.20.200;  # 访问的server 可以是IP或者域名
        charset UTF-8;
        add_header 'Access-Control-Allow-Origin' '*';
        # add_header Cache-Control no-cache;
        max_ranges 1;
        #charset koi8-r;
        access_log  /opt/log/uwsgi/access.log;  # log文件。必须有写权限且文件夹存在
        error_log  /opt/log/uwsgi/error.log;


        location / {
            # proxy_intercept_errors on;
            # recursive_error_pages on;
            #error_page 404  /rm4p$uri ;

            include     uwsgi_params;
            uwsgi_pass  172.16.20.200:8888;    # 监听的uwsgi端口，同uwsgi配置项socket一致
            uwsgi_connect_timeout 1200;
            uwsgi_read_timeout 1800;
            uwsgi_send_timeout 1200;
            proxy_read_timeout 1200;
            client_max_body_size 100m;
        }
}
```

保存后

```shell
service nginx restart
```

* **问题：Django静态文件找不到**

  在django全局settings.py中添加

  ```python
  import os
  ...
  STATICFILES_DIRS = (
      os.path.join(BASE_DIR, 'static/'),
  )
  
  STATIC_ROOT = '/opt/website/static'
  ```

  执行收集静态文件

  ```shell
  python3 manage.py collectstatic
  ```

  uwsgi中执行static

  ```shell
  uwsgi --ini uwsgi.ini --plugin python3 --static-map /static=/opt/website/static
  ```

### 一键启动脚本

```shell
cd /opt/website
# 杀掉之前的进程
sudo ps -ef | grep uwsgi | awk '{print $2}' | sudo xargs kill -9
uwsgi --ini uwsgi.ini --plugin python3 --static-map /static=/opt/website/static
```



