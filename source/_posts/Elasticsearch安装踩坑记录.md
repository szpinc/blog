---
title: Elasticsearch安装踩坑记录
date: 2019-05-08 13:17:20
tags:
    - Docker
    - Elasticsearch
categories:
    - 学习笔记
---

# 下载ES
官网安装教程：[](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/docker.html)

Docker官方提供了Elasticsearch的镜像，拉取镜像，版本是`5.6.12`:

> docker pull elasticsearch:5.6.9

# 安装ES

## 创建外挂目录
``` bash
sudo mkdir /data/docker/db/elasticsearch/{config,data}
sudo chmod -R 777 /data/docker/db/elasticsearch/*
```
## 创建并编辑elasticsearch.yml配置文件
``` bash
vim /data/docker/db/elasticsearch/config/elasticsearch.yml
```
编辑yml
``` yml
# ---------------------------------- Cluster -----------------------------------
#集群名称
cluster.name: elasticsearch
#
# ------------------------------------ Node ------------------------------------
#节点名
node.name: node-1
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
#path.data: /path/to/data
#
# Path to log files:
#
#path.logs: /path/to/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# 允许所有主机访问
#
network.host: 0.0.0.0
# 对外真实ip
network.publish_host: 192.168.31.100
#
# http访问端口
http.port: 9200
# 允许跨域
http.cors.enabled: true
# 允许跨域的主机(*代表允许所有主机访问)
http.cors.allow-origin: "*"
# tcp访问端口
transport.tcp.port: 9300
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.zen.ping.unicast.hosts: ["host1", "host2"]
#
# Prevent the "split brain" by configuring the majority of nodes (total number of master-eligible nodes / 2 + 1):
#
#discovery.zen.minimum_master_nodes: 3
#
# For more information, consult the zen discovery module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
#gateway.recover_after_nodes: 3
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true
```



## 坑一 JVM内存不够
我这里使用的是虚拟机，内存只有1G，然而es从5.0开始默认指定jvm内存为2g，我们需要修改下参数。
查看docker镜像中心的介绍[](https://hub.docker.com/r/itzg/elasticsearch/):
![](https://ws1.sinaimg.cn/large/005LP3H3gy1g2tuh4m83zj30wl06twer.jpg)

需要在运行命令中加上
```
ES_JAVA_OPTS="-Xms512m -Xmx512m"
```

## 坑2 Java使用
