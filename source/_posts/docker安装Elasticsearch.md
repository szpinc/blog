---
title: docker安装Elasticsearch
date: 2019-05-07 11:10:02
tags:
    - Docker
    - Elasticsearch
categories:
    - 学习笔记
---

# 安装Elasticsearch

## 安装

``` bash
docker run -it --name elasticsearch -d -p 9200:9200 -p 9300:9300 -p 5601:5601 elasticsearch
```

> 官方的镜像的网络设置是允许外部访问的即network.host=0.0.0.0

> 如果要制定es配置可用通过-E{param=value}指定，或者通过-v "$PWD/config":/usr/share/elasticsearch/config 映射配置文件地址

> -p 5601:5601 是kibana的端口地址 (我这里kibana的container共用elasticsearch的网络，所以这样设置)

## 验证
``` xml
> curl http://localhost:9200
{
    "name": "OwHPNzY",
    "cluster_name": "elasticsearch",
    "cluster_uuid": "WeiDMjJARv2DHMcCrQgS6g",
    "version": {
    "number": "5.6.4",
    "build_hash": "8bbedf5",
    "build_date": "2017-10-31T18:55:38.105Z",
    "build_snapshot": false,
    "lucene_version": "6.6.1"
},
    "tagline": "You Know, for Search"
}
```

# 安装Kibana

## 安装

``` bash
docker run -it -d -e ELASTICSEARCH_URL=http://127.0.0.1:9200 --name kibana --network=container:elasticsearch kibana
```

> --network 指定容器共享elasticsearch容器的网络栈 (使用了--network 就不能使用-p 来暴露端口)

## 验证
访问 http://localhost:5601

![](https://ws1.sinaimg.cn/large/005LP3H3gy1g2sl90ikgxj31hc0o3tce.jpg)

# 安装Elasticsearch-head

> v5.x以后不支持plugin需要独立部署

``` nodejs
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install
npm run start
```
## 验证

访问 http://localhost:9100/

![](https://ws1.sinaimg.cn/large/005LP3H3gy1g2slf44hejj30r40d5mxo.jpg)

直接配置elasticsearch服务有跨域问题,所以加了nginx代理

# nginx 服务配置

``` json
server {
        listen       9201;
        server_name  localhost;
    location / {
        add_header Access-Control-Allow-Origin *;
                add_header Access-Control-Allow-Headers Origin,X-Requested-With,Content-Type,Accept;
                add_header Access-Control-Allow-Methods GET,POST,PUT,PATCH,OPTIONS,DELETE;
            add_header Cache-Control no-store;
        proxy_pass http://127.0.0.1:9200;
    }
}
```

# 安装中文分词器

``` bash
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.0.0/elasticsearch-analysis-ik-6.0.0.zip
```

# 集群部署

当然我们也可以使用`docker-compose`的方式进行集群部署
`docker-compose.yml`文件内容如下

``` yml
version: '3.7'
services:
  elasticsearch:
    image: elasticsearch
    container_name: elasticsearch
    hostname: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet
  elasticsearch2:
    image: elasticsearch
    container_name: elasticsearch2
    hostname: elasticsearch2
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=[elasticsearch,elasticsearch2,elasticsearch3]"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata2:/usr/share/elasticsearch/data
    networks:
      - esnet
  elasticsearch3:
    image: elasticsearch
    container_name: elasticsearch3
    hostname: elasticsearch3
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=[elasticsearch,elasticsearch2,elasticsearch3]"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata3:/usr/share/elasticsearch/data
    networks:
      - esnet    
  kibana:
    image: kibana
    container_name: kibana
    ports:
      - 5601:5601
    environment:
      - ELASTICSEARCH_URL=http://elasticsearch:9200
    networks:
      - esnet

volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local
  esdata3:
    driver: local  
networks:
  esnet:
```

执行`docker-compose up -d`启动

