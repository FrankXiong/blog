---
title: CentOS 7.0 配置防火墙
date: 2016-05-30 23:23:00
tags:
- CentOS
categories:
- 后端
---
> 前一阵子被一个蜜汁 bug 困扰：Node.js 代码能在服务器上跑起来，但从浏览器却无法访问服务器 80 端口。于是在本地玩了一周。今天突然想到可能是防火墙的配置问题。

之前用的 iptables 来管理的防火墙，后来发现 CentOS 7.0 中已经用 firewalld 取代
 iptables 了，于是与时俱进，停用了
 iptables。
```
systemctl stop iptables.service
```
然后来启动 firewalld 吧
```
systemctl start firewalld.service
```
给我报了这个错
>Failed to start firewalld.service: Unit firewalld.service is masked.

查了很久没找到解决办法，于是试着输入了下面这行命令，解决了。
```
systemctl unmask firewalld.service
```
启动 firewalld.service
```
systemctl start firewalld.service
```
把 80 端口添加到防火墙开放端口中
```
firewall-cmd --permanent --zone=public --add-port=80/tcp
```
重启一遍 firewalld 服务使其生效
```
systemctl restart firewalld.service
```
检查更改是否生效
```
firewall-cmd --zone=public --query-port=80/tcp
```
-------
参考：http://www.linuxidc.com/Linux/2016-05/131158.htm