---
title: 从此Github访问与clone不再慢了
date: 2018-08-08 20:48:01
thumbnail: https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=3470045779,3678888291&fm=27&gp=0.jpg
tags:
    - github
    - git
categories:
    - 笔记
---

在天朝，经常会出现访问Github 异常慢，尤其在clone远程代码的时候，几KiB/s的速度在遭遇几十上百M的代码后，让大家苦不堪言。

下面就介绍一种方法，绝对包治百病，亲测速度可以达到接近1M。

1.获取Github相关网站的ip

访问[https://www.ipaddress.com](https://www.ipaddress.com)，找到页面中下方的“IP Address Tools - Quick Links”，分别输入github.global.ssl.fastly.net和github.com，查询ip地址。

2.修改本地host文件

以Mac为例，命令行下输入：sudo vi /etc/host，然后输入电脑的密码，打开host文件。

3.增加host映射

参考如下，增加github.global.ssl.fastly.net和github.com的映射。

151.101.113.194 github.global.ssl.fastly.net192.30.253.112 github.com

4.更新DNS缓存

命令行输入：sudo dscacheutil -flushcache，使增加的映射生效。

5.大功告成

接下来就可以随意访问Github和clone代码了。
