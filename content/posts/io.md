---
title: "Five Network I/O Models"
date: 2023-01-04T17:27:49+08:00
draft: false
author: luckmu
tags:
  - io
---

5 network I/O models: blocking, nonblocking, multiplexing, signal-driven, asynchronous

<!--more-->

## Blocking I/O Model

![blocking](/posts/io-1.png)

如果 no datagram ready, 系统阻塞 (blocking) 直到 datagram ready, 然后 copy datagarm

## Nonblocking I/O Model

![nonblocking](/posts/io-2.png)

**loop polling**:<br>
**if** no datagram ready: 立即返回 err (EWOULDBLOCK)<br>
**else if** datagram ready: 进行 recvfrom I/O, copy datagram 并返回 (blocking)

整个流程不阻塞 application

## I/O Multiplexing Model

![multiplexing](/posts/io-3.png)

可见, select 和 blocking 流程是相似的, 甚至于 select 需要 2 个 system call, 那么 select 是否不如 blocking 呢?

答案为**否**;

+ <u>select 可以同时等待多个 **fd** to be ready;</u>
  + ready 后 recvfrom I/O, copy data 并返回 (blocking)
+ 如 blocking 要达到相似的效果, 需要多个线程等待多个 **fd** (每个 thread 都调用 recvfrom 进行 1 个 blocking I/O);

## Signal-Driven I/O Model

![signal-driven](/posts/io-4.png)

系统调用 (system call) **sigaction** 得到 signal handler, 

datagram ready, handler 收到 **SIGIO**, recvfrom 并 copy datagram 到用户空间 (blocking)

## Asynchronous I/O Model

![asynchronous](/posts/io-5.png)

aio_read; 直到 datagram copy 完成 (data 复制到应用 buffer) 才返回, 也是和 signal-driven 的区别, 纯异步 (其他方法 recvfrom 是同步的, 即真实 I/O op 是同步的)

## Refer:

+ [5 Network I/O Models](https://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch06lev1sec2.html)
