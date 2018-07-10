---
title: Docker容器技术：初学笔记
date: 2017-06-27 20:04:09
tags:
    - Docker
    - 云计算
    - 大数据

categories:
    - 学习笔记
---


好久没来了。艾维瑞巴迪。我胡汉三又回来了！
![](http://img.mukewang.com/5820943f00011ebf03630199.gif)
最近在学习Docker容器技术，顺带整理了一下。
[Docker官网](https://hub.docker.com/)
[DockerCloud](https://dashboard.daocloud.io/)
**1.使用docker需要系统root权限，否则提示权限不足**
![](http://img.mukewang.com/582092d7000197f310320154.png)
**2.常用的docker命令**
  1.查看docker信息（version、info）
``` bash
# 查看docker版本
docker version
# 显示docker系统的信息
docker info
```
2.账号以及对image的操作（search、pull、push、images、rmi、history）
``` bash
#登录Docker Hub，push时需要登录
docker login
#退出账号
docker logout
#从远程仓库搜索镜像
docker search java
#从远程仓库拉取镜像
docker pull imagePath
#推送本地镜像到远程仓库
docker push imagePath
#查看所有镜像
docker images
#删除镜像(务必先停掉该镜像下的所有容器)
docker rmi [options] imaId

root@yadong-virtual-machine:/home/yadong# docker rmi 527
Error response from daemon: Conflict, cannot delete 527fbbf3458d because the container 52174c817c43 is using it, use -f to force
Error: failed to remove images: [527]

# 显示一个镜像的历史
docker history image_name
```

3.容器操作（run、create、start、stop、restart、kill、rm、ps、）
``` bash
#启动一个容器（create-start）
docker run -it -p [port] conId conShell
#创建一个容器
docker create (用法同run，只不过create只创建并不会启动)
#启动/停止/重启容器
docker start/stop/restart
#杀掉一个容器
docker kill conId
#删除容器（删除处于停止状态的容器）
docker rm conId
#在运行的容器中执行命令
docker exec
#列出正在运行容器
docker ps [OPTIONS]
#列出所有容器
docker ps -a
#列出最近创建的5个容器
docker ps -n 5
OPTIONS说明:
    -a :显示所有的容器，包括未运行的。
    -f :根据条件过滤显示的内容。
    --format :指定返回值的模板文件。
    -l :显示最近创建的容器。
    -n :列出最近创建的n个容器。
    --no-trunc :不截断输出。
    -q :静默模式，只显示容器编号。
    -s :显示总的文件大小。
#提交本地容器为新的镜像（保存对容器的修改）
docker commit [-m "xx"] conId imagePath
#容器中的进程管理器
docker top conId
#链接容器
docker attach conId
#获取某个容器的日志
docker logs [options] conId
OPTIONS说明：
    -f : 跟踪日志输出
    --since :显示某个开始时间的所有日志
    -t : 显示时间戳
    --tail :仅列出最新N条容器日志
    #查看指定容器的端口映射
    docker port conId
    #获取容器/镜像的元数据
    docker inspect conId
```

```bash
#-v挂载多个目录
docker run -it -v /home/yadong/yadongDir:/yadongTest -v /home/yadong/yadonga:/yadongaa e07 /bin/bash
```

Docker常用命令关系图
![](http://img.mukewang.com/582093910001fed218700970.png)
容器状态图
![](http://img.mukewang.com/582093ba0001d21411700620.png)

最后推荐一些学习Docker不错的网站：

1. [DockOne.io，最专业的Docker交流平台](http://dockone.io/)
2. [Docker —— 从入门到实践](http://udn.yyuap.com/doc/docker_practice/index.html)
3. [菜鸟学习：Docker篇](http://www.runoob.com/docker/docker-architecture.html)


