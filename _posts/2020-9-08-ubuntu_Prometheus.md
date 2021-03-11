---
layout: post
title: '如何在ubuntu下安装Prometheus'
description: '如何如何在ubuntu下安装Prometheus'
categories: [Ubuntu]
image:  /assets/img/blog/Prometheus_Grafana.png

related_posts:
  - _posts/
  - _posts/
---

- Table of Contents
{:toc .large-only}

Prometheus是一个开源的服务监控系统，它通过HTTP协议从远程的机器收集数据并存储在本地的时序数据库上

### 安装Prometheus

我这里使用docker 存放 ``Prometheus`` 易于管理

首先下载 ``Prometheus`` 镜像

```powershell
docker pull prom/prometheus
```

然后在本地创建一个``yml``配置文件 

```powershell
mkdir -p /home/prometheus
cd /home/prometheus/
 
sudo vim /home/prometheus/prometheus.yml
```

将下面配置内容放进去

```powershell
global:
  scrape_interval: 15s  # 将刮擦间隔设置为每15秒一次。 默认值为每1分钟。
  scrape_timeout: 10s  # 每10秒评估一次规则。 默认值为每1分钟。
  evaluation_interval: 1m # 定期评估间隔

  external_labels:
     monitor: codelab-monitor

scrape_configs:

  - job_name: prometheus
    honor_timestamps: true # 配置 Prometheus 是否处理在已抓取的数据中存在的时间戳
    scrape_interval: 5s  # 抓取频率
    scrape_timeout: 5s   # 抓取超时时间
    metrics_path: '/metrics' # 从目标抓取指标的URL路径，默认是 /metrics
    scheme: 'http' # 请求的协议

    static_configs:
    - targets: ['localhost:9090']
```

然后启动容器

```powershell
docker run -d -p 9090:9090 --name prometheus \
--restart=always \
-v /home/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

查看镜像

```powershell
docker ps -a
```

打开浏览器输入 ``localhost:9090`` 就可以访问到Prometheus自带的界面了 

![自带页面](\assets\img\Prometheus\index.png)

###  通过exporter监控主机

下载exporter二进制文件安装

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v0.17.0/node_exporter-0.17.0.linux-amd64.tar.gz
tar -zxvf node_exporter-0.17.0.linux-amd64.tar.gz
mv node_exporter-0.17.0.linux-amd64 /usr/local/node_exporter
```

systemctl service常见存放在两个目录中：
/etc/systemd/system
/usr/lib/systemd/system
一般自己创建的service 直接放在 /etc/systemd/system 之中

```powershell
vim /usr/lib/systemd/system/node_exporter.service
```

将配置输入进去

```bash
[Unit]
Description=https://prometheus.io

[Service]
Restart=on-failure
ExecStart=/usr/local/node_exporter/node_exporter

[Install]
WantedBy = multi-user.target
```

启动服务

```powershell
systemctl start node_exporter.service 
```

之后node_exporter暴露一个9100端口

![exporter](\assets\img\Prometheus\exporter.png)



### 通过cAdvisor搜集容器资源使用信息

下载并启动cadvisor

```powershell
docker container run -d --name=cadvisor \
--volume=/:/rootfs:ro \
--volume=/var/run:/var/run:ro \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker/:/var/lib/docker:ro \
--volume=/dev/disk/:/dev/disk:ro \
--publish=8080:8080 \
--detach=true \
google/cadvisor:latest
```

然后访问8080 端口 可以看到自集成的页面

![cadvisor](\assets\img\Prometheus\cadvisor.png)



http://localhost:8080/**metrics** 暴露数据指标（接口），但cadvisor只负责采集，不负责存储，所以需要配合Prometheus

### 修改pometheus配置文件

修改pometheus.yml 配置 添加两个配置

```powershell
- job_name: docker
  static_configs:
  - targets:
    - 10.0.9.96:8080

- job_name: Linux
  static_configs:
  - targets :
    - 10.0.9.96:9100
```

之后重启 ``docker restart pometheus ``服务

等待配置验证一会 

访问``http://localhost:9090/targets``

![修改配置](\assets\img\Prometheus\change_setting.png)

ok 连接成功

![查询数据](\assets\img\Prometheus\query_data.png)

数据也有了

## Grafana 可视化展示

### 1. Grafana

Grafana的部署很简单，是一个纯静态的系统，拉取镜像暴露一个端口即可，

```bash
docker container run -d --name=grafand -p 3000:3000 grafana/grafana
```

自定义注册账号

![Grafana登录](\assets\img\Prometheus\Grafana_login.png)

然后添加数据源 选择prometheus

![添加数据源](\assets\img\Prometheus\add_data_source.png)

添加 url:``http://host:9090`` 点击保存

然后去社区找一个prometheus监控模版``https://grafana.com/grafana/dashboards/8919``

![添加模版](\assets\img\Prometheus\add_template.png)

回到页面导入模版

![导入模版](\assets\img\Prometheus\import_template.png)

修改配置

![导入配置](\assets\img\Prometheus\import_configuration.png)

导入成功后就可以看到监控组件

![监控组件](\assets\img\Prometheus\monitoring_component.png)

![监控组件2](\assets\img\Prometheus\monitoring_component2.png)

ok 基本跑起来了 那么我想监控一些组件怎么办呢

## 监控redis

下载redis和redis采集器

```powershell
docker pull redis
docker pull oliver006/redis_exporter:latest
```

  启动redis:

```
docker run -d --name redis -p 6379:6379 redis
```

启动redis 采集器

```powershell
docker run -d \
--name redis_exporter \
-p 9121:9121 \
oliver006/redis_exporter --redis.addr redis://47.117.126.142:6379
```

监控redis的地址必须是服务器公网ip

{:.note title="注意"}



修改pometheus.yml 添加redis数据源

```shell
  - job_name: redis
    scrape_interval: 5s
    static_configs:
    - targets: ['47.117.126.142:9121']
```

稍等一会访问 ``ip:9090/targets`` 就能看到redis数据已经有了

![redis_exporter](\assets\img\Prometheus\redis_exporter.png)

然后按上面步骤 导入一个 id ``11835``的模板

就可以监控redis的使用情况



