---
title: traceroute的原理
date: 2016-05-23 21:49:09
tags:
- IP
categories:
- 网络
---

###### 1.什么是traceroute?
 traceroute， Linux系统称为tracepath，Windows系统称为tracert，是一种计算机网络工具。它可显示数据包在IP网络经过的路由器的IP地址。通过traceroute我们可以知道信息从你的计算机到互联网另一端的主机是走的什么路径。
<!-- more -->
traceroute有不同的实现版本：常规的traceroute(基于UDP和ICMP)和tcptraceroute(基于TCP)

###### 2.traceroute原理
常规的traceroute和tcptraceroute具有相同的工作原理：
>1.  发送一个TTL(Time-To-Live)相当小的包，TTL经过每一跳时会递减。当它减为0时，数据包就被丢弃。
2. 当TTL失效后，看哪个路由器返回一个带有表明的ICMP “time exceeded”
3. 如果返回的路由器就是最终的目的地，停止trace
4. 否则，TTL加1并返回到步骤1

两者的不同点：
>- 常规的traceroute使用UDP包或ICMP “Echo”包，这两种包都可能会被防火墙拦截。
- tcptraceroute使用TCP “SYN”包。发送带SYN标志位的数据段是TCP建立连接时进行“三次握手”的第一次握手，只要目标地址是被允许访问的，通常这种包不会被防火墙拦截。但是防火墙会拦截其他的不是用于建立连接的TCP包。
- 基于TCP的traceroute拥有更高的访问权限。以amazon.com为例。基于UDP的traceroute停在205.251.248.5，这个地址很可能是某种防火墙。基于TCP的traceroute访问80端口，这是amazon.com的默认端口，然后进入下一步，最终停在72.21.194.212
