---
title: RabbitMQ教程（一）：安装RabbitMQ
date: 2019-01-05 20:57:43
tags:
    - 消息队列
categories:
    - 学习笔记
---

## CentOS7安装RabbitMQ
### 安装Erlang
由于RabbitMQ依赖Erlang， 所以需要先安装Erlang。
Erlang的安装方式大概有两种：
1. 从Erlang Solution安装(推荐)
``` bash
 # 添加erlang solutions源
 $ wget https://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
 $ sudo rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
 
 $ sudo yum install erlang
```
2. 从EPEL源安装(这种方式安装的Erlang版本可能不是最新的，有时候不能满足RabbitMQ需要的最低版本
``` bash
 # 启动EPEL源
 $ sudo yum install epel-release 
 # 安装erlang
 $ sudo yum install erlang  
```
**完成后安装RabbitMQ：**
先下载rpm：
``` bash
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.6/rabbitmq-server-3.6.6-1.el7.noarch.rpm
```
下载完成后安装：
``` bash
yum install rabbitmq-server-3.6.6-1.el7.noarch.rpm 
```
安装时如果遇到下面的依赖错误
``` bash
Error: Package: socat-1.7.2.3-1.el6.x86_64 (epel)
       Requires: libreadline.so.5()(64bit)
```
可以尝试先执行
```
$ sudo yum install socat
```
### 关于RabbitMQ的一些基本操作
```
$ sudo chkconfig rabbitmq-server on  # 添加开机启动RabbitMQ服务
$ sudo /sbin/service rabbitmq-server start # 启动服务
$ sudo /sbin/service rabbitmq-server status  # 查看服务状态
$ sudo /sbin/service rabbitmq-server stop   # 停止服务
 
# 查看当前所有用户
$ sudo rabbitmqctl list_users
 
# 查看默认guest用户的权限
$ sudo rabbitmqctl list_user_permissions guest
 
# 由于RabbitMQ默认的账号用户名和密码都是guest。为了安全起见, 先删掉默认用户
$ sudo rabbitmqctl delete_user guest
 
# 添加新用户
$ sudo rabbitmqctl add_user username password
 
# 设置用户tag
$ sudo rabbitmqctl set_user_tags username administrator
 
# 赋予用户默认vhost的全部操作权限
$ sudo rabbitmqctl set_permissions -p / username ".*" ".*" ".*"
 
# 查看用户的权限
$ sudo rabbitmqctl list_user_permissions username
```
更多关于rabbitmqctl的使用，可以参考[帮助手册](https://link.jianshu.com/?t=https://www.rabbitmq.com/man/rabbitmqctl.1.man.html)。

### 开启web管理接口
如果只从命令行操作RabbitMQ，多少有点不方便。幸好RabbitMQ自带了web管理界面，只需要启动插件便可以使用
```
$ sudo rabbitmq-plugins enable rabbitmq_management
```
然后通过浏览器访问
`http://ip:15672`
输入用户名和密码访问web管理界面了。

### 配置RabbitMQ
关于RabbitMQ的配置，可以下载RabbitMQ的[配置文件模板](https://link.jianshu.com/?t=https://raw.githubusercontent.com/rabbitmq/rabbitmq-server/stable/docs/rabbitmq.config.example)到/etc/rabbitmq/rabbitmq.config, 然后按照需求更改即可。
关于每个配置项的具体作用，可以参考[官方文档](https://link.jianshu.com/?t=https://www.rabbitmq.com/configure.html)。
更新配置后，别忘了重启服务哦！

### 开启用户远程访问
默认情况下，RabbitMQ的默认的guest用户只允许本机访问， 如果想让guest用户能够远程访问的话，只需要将配置文件中的loopback_users列表置为空即可，如下
```
{loopback_users, []}
```
另外关于新添加的用户，直接就可以从远程访问的，如果想让新添加的用户只能本地访问，可以将用户名添加到上面的列表, 如只允许admin用户本机访问
```
{loopback_users, ["admin"]}
```
更新配置后，别忘了重启服务哦！
```
 sudo /sbin/service rabbitmq-server status  # 查看服务状态
```
![](http://images2015.cnblogs.com/blog/321801/201611/321801-20161123155611737-770552575.png)
这里可以看到log文件的位置，转到文件位置，打开文件：
![](http://images2015.cnblogs.com/blog/321801/201611/321801-20161123155722050-140847138.png)
这里显示的是没有找到配置文件，我们可以自己创建这个文件
```
cd /etc/rabbitmq/
vi rabbitmq.config
```
编辑内容如下：
```
[{rabbit, [{loopback_users, []}]}].
```
这里的意思是开放使用，`rabbitmq`默认创建的用户`guest`，密码也是`guest`，这个用户默认只能是本机访问，`localhost`或者`127.0.0.1`，从外部访问需要添加上面的配置。
保存配置后重启服务：
```
service rabbitmq-server stop
service rabbitmq-server start
```
此时就可以从外部访问了，但此时再看log文件，发现内容还是原来的，还是显示没有找到配置文件，可以手动删除这个文件再重启服务，不过这不影响使用
```
rm rabbit\@mythsky.log 
service rabbitmq-server stop
service rabbitmq-server start
```
注意:记得要开放5672和15672端口
```
/sbin/iptables -I INPUT -p tcp --dport 5672 -j ACCEPT
/sbin/iptables -I INPUT -p tcp --dport 15672 -j ACCEPT
```

