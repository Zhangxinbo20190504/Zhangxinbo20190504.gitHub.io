---
layout:     post
title:      "SonarQube部署"
subtitle:   "基于ubuntu2004"
date:       2021-11-23 15:00:00
author:     "Xinbo"
header-img: "img/post-bg-rwd.jpg"
tags:
    - 工具部署
---


# Ubuntu 2004 sonarqube 搭建

## 安装依赖

* 非root账户登录服务器

* 安装java11

  ``` bash
  sudo apt install openjdk-11-jdk
  ```

  

* 安装数据库 postgresql

  > 备注：SonarQube 7.9之后不再支持MySQL数据库

  ``` 
  sudo apt-get install postgresql
  ```

## 配置postgresql

> 安装完成后会自动创建用户 postgres

```bash
su - postgres
psql
create database sonar;
create user sonar;
alter user sonar with password 'sonar';
alter role sonar createdb;
alter role sonar superuser;
alter role sonar createrole;
alter database sonar owner to sonar；

```

## 下载sonarqube

安装路径（自定义）：/home/self-build/sonarqube-9.1.0.47736

下载地址 https://www.sonarqube.org/downloads/

解压

``` bash
unzip sonarqube-9.1.0.47736.zip
```

## 启动sonarqube

修改解压后的目录下/conf/sonar.properties文件

``` 
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost/sonar
sonar.web.host={your ip}
sonar.web.port=9000
sonar.web.context=/sonar
```

启动，解压目录 /bin/linux-x86-64

``` 
sh sonar.sh start
```

查询状态

``` bash
sh sonar.sh status
>>> SonarQube is running (1119558)
```

访问 http://{your_id}:9000/sonar  初始密码：admin admin，首次登录可能会要求修改admin默认密码

汉化：安装chinese插件，重启

### 可能遇到的问题

日志路径：解压目录 logs/ 下

* 问题1：sonarqube需要的最大虚拟机内存不足，文件大小限定过低

  > 需要root操作

  ``` 
  //编辑系统配置文件
  vim /etc/sysctl.conf
  //在文件尾部追加以下两行
  vm.max_map_count=262144
  fs.file-max=65536
  //刷新配置
  sysctl -p
   
  //编辑profile
  vim /etc/profile
  //在文件尾部追加下面一行
  ulimit -n 65536
  //保存使其生效
  source /etc/profile
  
  ```




