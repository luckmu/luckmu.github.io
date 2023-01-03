---
title: "TIME_WAIT in TCP"
date: 2023-01-03T04:41:57+08:00
draft: false
categories:
  - defines
tags:
  - tcp
---

之前 k8s 集群里出现过多 `TIME_WAIT` 导致网络问题, 又面试遇到, 特记录。

主要问题是:

1. `TIME_WAIT` 的定义?
2. 为什么需要 `TIME_WAIT` 且持续 `2*MSL`?
3. `TIME_WAIT` 引起的问题? 怎么解决?

<!--more-->

## What is `TIME_WAIT`?

`TIME_WAIT` 是 TCP 的 1 个状态;

即在 4 次挥手过程中, 主动断开连接方收到被动方 `FIN` 后, 随即发送 `ACK`; 此时(发送 `ACK` 后)进入 `TIME_WAIT` 状态，并在等待 `2*MSL` 时间后变为 `CLOSED` 状态并断开连接

**MSL(Maximum Segment Lifetime)**: 报文最大生存时间; 如果报文在网络中活动 MSL 后没被接收方收到, 随即丢弃;

*RFC 793 建议为 2 min, 但取决于实现, Linux 为 30s, 即 `2*MSL` 为 60s, 硬编码于内核(~~除非重新编译内核, 无法修改此值~~)*

## Why `2*MSL`?

**将无 `TIME_WAIT` (`TIME_WAIT`=0) 阶段视为 `TIME_WAIT` < `2*MSL`;**

`TIME_WAIT` 短于 `2*MSL`, 客户端可以正常关闭连接, 而服务端则有两种情况:

+ 没有收到客户端 `ACK`, 进行 `FIN` 重传
  + `TIME_WAIT` < `2*MSL`, 此时服务端没有收到 `ACK`, 开始重传 `FIN`, 而客户端已经 `CLOSED`;<br>
    则服务端无法正常关闭 (收不到 `ACK`); 若此时客户端以相同的 4 元组 (src_ip, src_port, dst_ip, dst_port) 请求新的 TCP 连接, 会被服务端返回 RST 包并拒绝连接请求;
+ 收到 `ACK`, 并关闭连接
  + `TIME_WAIT` < `2*MSL`, 如果客户端以相同 4 元组建立新的 TCP 连接;<br>
    但网络中仍有旧连接的数据 (<u>e.g. 客户端在发送最后的 `ACK` 之前, 发送的包到达服务端 <`1*MSL`, 服务端应答并到达客户端 <`1*MSL`</u>), 这个旧的包可能刚好可以被客户端承认

## Too many `TIME_WAIT`s

`net.ipv4.tcp_tw_reuse`: 新的 timestamp 如果严格大于之前连接记录的最近的 timestamp, 1 个处于 `TIME_WAIT` 的 outgoing 连接可以被重用 (等待 1s 即可)。

+ 客户端 `ACK` 丢包, 服务端处于 `LAST_ACK` 状态;
+ 客户端准备建立连接, 发送 `SYN` 包;
+ 服务端收到 `SYN` 包, 返回 `FIN, ACK` 而不是 `RST` 重置连接;
+ 客户端收到 `FIN, ACK`, 发送 `RST` 重置连接;
+ 客户端发送 `SYN` 建立连接;

## Refer:

+ [Coping with the TCP TIME-WAIT state on busy Linux servers](https://vincent.bernat.ch/en/blog/2014-tcp-time-wait-state-linux)
+ [再叙 TIME_WAIT](https://huoding.com/2013/12/31/316)
+ [为什么 TCP 协议有 TIME_WAIT 状态](https://draveness.me/whys-the-design-tcp-time-wait/)
