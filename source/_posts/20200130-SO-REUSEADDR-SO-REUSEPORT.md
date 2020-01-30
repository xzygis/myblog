---
title: SO_REUSEADDR & SO_REUSEPORT
date: 2020-01-30 12:50:51
tags: 
    - REUSEADDR
    - REUSEPORT
category: 后台开发
---

**SO_REUSEADDR**

1. 当有一个有相同本地地址和端口的socket1处于TIME_WAIT状态时，而你启动的程序的socket2要占用该地址和端口，你的程序就要用到该选项。
2. SO_REUSEADDR允许同一port上启动同一服务器的多个实例(多个进程)。但每个实例绑定的IP地址是不能相同的。在有多块网卡或用IP Alias技术的机器可
以测试这种情况。
3. SO_REUSEADDR允许单个进程绑定相同的端口到多个socket上，但每个socket绑定的ip地址不同。这和2很相似，区别请看UNPv1。
4. SO_REUSEADDR允许完全相同的地址和端口的重复绑定。但这只用于UDP的多播，不用于TCP。

<!-- more -->

**SO_REUSEPORT**
> The new socket option allows multiple sockets on the same host to bind to the same port, and is intended to improve the performance of multithreaded network server applications running on top of multicore systems.

linux kernel 3.9引入了最新的SO_REUSEPORT选项，使得多进程或者多线程可以创建多个绑定同一个ip:port的监听socket，提高服务器的接收连接的并发能力,程序的扩展性更好；此时需要设置SO_REUSEPORT（注意所有进程都要设置才生效）。

目的：每一个进程有一个独立的监听socket，并且bind相同的ip:port，独立的listen()和accept()；提高接收连接的能力。（例如nginx多进程同时监听同一个ip:port）

解决的问题：

- 避免了应用层多线程或者进程监听同一ip:port的“惊群效应”。
- 内核层面实现负载均衡，保证每个进程或者线程接收均衡的连接数。
- 只有effective-user-id相同的服务器进程才能监听同一ip:port （安全性考虑）

golang开源实现：https://github.com/kavu/go_reuseport

注意：SO_REUSEPORT只支持TCP和UDP。对unix domain socket不生效。