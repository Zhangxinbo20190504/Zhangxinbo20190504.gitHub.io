---
layout:     post
title:      "服务器监控大屏搭建"
subtitle:   "docker部署 promethus grafana"
date:       2021-12-16 17:00:00
author:     "Xinbo"
header-img: "img/post-bg-rwd.jpg"
tags:
    - 工具部署
    - Prometheus
    - Grafana
---

服务器的监控在devops价值流中显得尤为重要，准确的监控对判断业务风险有极大的价值，利用 promethus 和 grafana 可以让我们短时间迅速掌握服务器监控，拥有炫酷的监控大屏

docker摆脱了监控搭建步骤的繁琐，能够统一环境，学习成本与运维成本低，是很好的搭建选择

> 基于ubuntu2004 ，且已安装docker

### 运行node-exporter

在需要监控的服务器上下载 node-exporter

```shell
docker pull prom/node-exporter
```

运行 node-exporter

```shell
docker run -d -p 9100:9100 -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro" --net="host" prom/node-exporter
```

查看 9100 端口是否监听

```shell
netstat -anpt | grep 9100
>>>
tcp6       0      0 :::9100                 :::*                    LISTEN      2659039/node_export
```

在 浏览器运行 http://<your_ip>:9100/metrics，可看到类似如下输出

![image-20211216162732311](https://s2.loli.net/2021/12/16/GRbSTDoasUYw4qc.png)

### 部署promethus

```shell
docker pull prom/prometheus
```

新建 prometheus 目录，并编辑配置文件 prometheus.yml

```shell
mkdir /opt/prometheus
cd /opt/prometheus/
vim prometheus.yml
```

```yaml
global:
  scrape_interval:     60s
  evaluation_interval: 60s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
        labels:
          instance: prometheus

  - job_name: linux
    static_configs:
      - targets: ['<your server ip>:9100']   # 需要监控的ip
        labels:
          instance: ubuntu
```

启动 promethus

```shell
docker run -d -p 9090:9090 -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

访问 http://<your_ip>:9090/targets 可以看到如下效果

 ![image-20211216165430834](https://s2.loli.net/2021/12/16/6aRuEIcVg7bxdCZ.png)

promethus 运行成功

### 部署grafana

为了更好的展示promethus的数据，我们选用 grafaana 作为数据展示的看板

下载grafana

```shell
docker pull grafana/grafana
```

新建空文件夹存放grafana的数据

```
mkdir /opt/grafana-storage
```

设置权限

> grafana会使用 grafana 用户去写数据，所以直接设置成777

```javascript
mkdir /opt/grafana-storage
```

启动

```shell
docker run -d -p 3000:3000 --name=grafana -v /opt/grafana-storage:/var/lib/grafana grafana/grafana
```

访问 http://<your_ip>:3000  可以看到grafana的登录界面

> 默认用户名密码 admin / admin，登录会要求重设密码，可以跳过

![image-20211216170625860](https://s2.loli.net/2021/12/16/k63JRCfX8r5yiQx.png)

进入 grafana 后 设置 promethus 数据源

![image-20211216171141588](https://s2.loli.net/2021/12/16/UQLivtaAVp56ePX.png)

接下来 import 一个模板，通过id 和 json导入均可，数据源选择 刚刚设置的promethus

> https://grafana.com/dashboards/  可查看各种模板及id

![image-20211216171501379](https://s2.loli.net/2021/12/16/bJELlDqKkzZVWdo.png)

然后就可以根据需要调整看板显示数据了