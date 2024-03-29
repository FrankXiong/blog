---
title: DDoS 原理与应对之道
date: 2016-03-23 01:46:48
tags:
- DDos
categories:
- 安全
---

# 原理
拒绝服务攻击的主要目的是使被攻击的网络或服务器不能提供正常的服务。web服务器在一定时间内只能处理一定量的通信，如果通信量超过了服务器承受的限度，服务器将会瘫痪。拒绝服务攻击就是向某台主机发送大量请求，使其无法响应正常的请求。比如每年春运时候的12306，大量用户涌入导致网站瘫痪，这就是一次现实版的DDoS攻击。

DDoS（分布式拒绝服务攻击）是在传统的DoS攻击基础上，利用更多的傀儡机来发起进攻。整个DDoS攻击体系可分为4部分：黑客、控制傀儡机、攻击傀儡机、受害者。黑客通过控制傀儡机向攻击傀儡机发送攻击命令，攻击傀儡机展开对目标主机的DoS攻击

# 典型的拒绝服务攻击手段
1. SYN湮没：这种攻击向服务器发送海量SYN信息，该信息中携带的源地址根本不可用，但是服务器会当真的SYN请求处理，当服务器尝试为每个请求分配连接来应答这些SYN请求时，就没有其他资源来处理用户真正合法的请求了。
2. Land攻击：Land攻击中，SYN数据包的源地址和目标地址都被设置成某一个服务器地址，这将导致接收到这个SYN包的服务器将向它自己发送SYN-ACK包，结果又自己返回ACK消息并建立一个空连接。每个这样的连接都将保持到超时。

# 应对之道
1. 首页静态化
2. 为服务器购买更大的带宽
3. 联系运营商进行流量清洗
4. 使用云主机
5. 封杀IP

------------
更多参见：[知乎：互联网创业公司如何防御 DDoS 攻击？](https://www.zhihu.com/question/19581905)