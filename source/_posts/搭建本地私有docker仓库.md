---
title: 搭建本地私有docker仓库
date: 2017-06-27 20:50:14
tags:
    - Docker
    - 云计算
    - 大数据
categories:
    - 学习笔记
---

# 系统
Ubuntu
# 依赖环境
```bash
docker
Client: Version: 18.05.0-ce API version: 1.37 Go version: go1.9.5 Git commit: f150324 Built: Wed May 9 22:16:25 2017
OS/Arch: linux/amd64 Experimental: false Orchestrator: swarm
Server:
Engine:
Version: 18.05.0-ce
API version: 1.37 (minimum version 1.12)
Go version: go1.9.5
Git commit: f150324
OS/Arch: linux/amd64
Experimental: false
```
## 1、拉取registry镜像
```bash
docker pull registry
```
## 2、启动仓库
``` bash
docker run -d --name=my-docker-registry --restart=always -p 5000:5000 -v /opt/data/registry:/tmp/registry registry
#这里说明下： - `--name`是启动镜像后容器的名字 - `-p`是映射的端口 - `-v`是挂载主机目录 /opt/data/registry 到容器的 /tmp/registry ，用于存储 push 进去的镜像文件，这里前面是主机的目录，你可以随意修改的 启动完毕后，执行：
docker ps -a
#看到的是这样的输出说明就启动OK了
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
4478264183c6 registry "/entrypoint.sh /etc…" 14 seconds ago Up 9 seconds 0.0.0.0:5000->5000/tcp my-docker-registry
```

## 3、在宿主机本地测试仓库
``` bash
docker pull nginx
```

## 4、给上面镜像重新打上tag
``` bash
docker tag nginx localhost:5000/my-nginx:1.0
```
这里解释下镜像的名字： `localhost:5000`是镜像仓库的地址， `/`后面的接的是镜像的名字，之后的`:`后面接的是版本号。

## 5、上传到仓库
``` bash
docker push localhost:5000/my-nginx:1.0
```

## 6、拉取仓库里面的镜像
``` bash
docker pull localhost:5000/my-nginx:1.0
```



