---
layout:     post
title:      "Win10 WSL部署"
subtitle:   "在win10上拥有一个ubuntu系统"
date:       2021-11-27 12:00:00
author:     "Xinbo"
header-img: "img/post-bg-rwd.jpg"
tags:
    - 工具部署
    - WSL
---

### 准备

安装 windows powershell 和 Terminal

应用商店 -> 搜索powershell 或者 Terminal -> 安装

### 启用功能

<img src="https://i.loli.net/2021/11/27/biwFqRVymHfea97.png" alt="image-20211123220941674" style="zoom:50%;" />



> 或者通过命令行
>
> admin 模式（管理员权限）打开powershell

``` shell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

### 下载wsl

点此下载内核更新包 [X86](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

> 安装完后可能需要重启

### 安装ubuntu

> 管理员权限

``` shell
 wsl --set-default-version 2
 wsl --install -d Ubuntu-20.04  # 本次选择2004版本
```

> 或者打开应用商店搜索 “Ubuntu 20.04”并安装

安装完成后启动一次，设置用户名和密码

### 迁移wsl到非系统盘

> 为了防止系统盘过大，建议迁移wsl至非系统盘

将WSL关闭

```shell
wsl --shutdown
```

导出到F盘（可根据情况自己选择）

> 需要WSL这个目录存在

```shell
wsl --export Ubuntu-20.04 F:\WSL\Ubuntu-20.04.tar
```

注销ubuntu2004

```shell
 wsl --unregister Ubuntu-20.04
```

导出

```shell
wsl --import Ubuntu-20.04 F:\WSL\Ubuntu-20.04 F:\WSL\Ubuntu-20.04.tar --version 2
```

设置默认用户

```shell
ubuntu2004 config --default-user xinbo
```

删除tar文件

```shell
rm F:\WSL\Ubuntu-20.04.tar
```

### 配置阿里云仓库

用下面内容替换掉 /etc/apt/sources.list

```
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
   
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
   
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
   
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
   
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

更新缓存 升级

```shell
sudo apt-get update
sudo apt-get upgrade
```

### 限制WSL内存使用

先使用管理员权限 powershell 命令关闭 wsl： `wsl --shutdown`

接着在用户文件夹下新建一个 .wslconfig 文件 `C:\Users\<yourUserName>\.wslconfig`，内容如下：`

```
[wsl2]
memory=3GB
swap=3GB
localhostForwarding=true
```

