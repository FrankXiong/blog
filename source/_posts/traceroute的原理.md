---
title: traceroute 的原理
date: 2016-05-23 21:49:09
tags:
- IP
categories:
- 网络
---

# 什么是 traceroute?
 traceroute， Linux 系统称为 tracepath，Windows 系统称为 tracert，是一种计算机网络工具。它可显示数据包在 IP 网络经过的路由器的 IP 地址。通过 traceroute 我们可以知道信息从你的计算机到互联网另一端的主机是走的什么路径。
<!-- more -->
traceroute 有不同的实现版本：常规的 traceroute(基于 UDP 和 ICMP)和 tcptraceroute(基于 TCP)

# traceroute 原理
常规的 traceroute 和 tcptraceroute 具有相同的工作原理：
>1. 发送一个 TTL(Time-To-Live) 相当小的包，TTL 经过每一跳时会递减。当它减为 0 时，数据包就被丢弃。
2. 当 TTL 失效后，看哪个路由器返回一个带有表明的 ICMP “time exceeded”
3. 如果返回的路由器就是最终的目的地，停止 trace
4. 否则，TTL 加 1 并返回到步骤 1

两者的不同点：
>- 常规的 traceroute 使用 UDP 包或 ICMP “Echo” 包，这两种包都可能会被防火墙拦截。
- tcptraceroute 使用 TCP “SYN” 包。发送带SYN标志位的数据段是 TCP 建立连接时进行“三次握手”的第一次握手，只要目标地址是被允许访问的，通常这种包不会被防火墙拦截。但是防火墙会拦截其他的不是用于建立连接的 TCP 包。
- 基于 TCP 的 traceroute 拥有更高的访问权限。以 amazon.com 为例。基于 UDP 的 traceroute 停在 205.251.248.5，这个地址很可能是某种防火墙。基于 TCP 的 traceroute 访问 80 端口，这是 amazon.com 的默认端口，然后进入下一步，最终停在 72.21.194.212
