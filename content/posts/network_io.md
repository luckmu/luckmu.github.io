---
title: "Network I/O"
date: 2023-01-13T16:00:16+08:00
draft: true
---

5 network I/O models & epoll details.

<!--more-->

Glos:

- unp: unix network programming
- tlpi: the linux programming interface
- fd: file descriptor

\<unp\> introduced 5 network I/O models: **blocking**, **nonblocking**, **multiplexing**, **signal-driven**, **asynchronous**

\<tlpi\> talked about: multiplexing and signal-driven.

De facto standard: **epoll**, kqueue, ... <u>(multiplexing + signal-driven)</u>.

While select and poll are multiplexing.

Benefits?

1. multiplexing compare to blocking, nonblocking?
2. multiplexing + signal-driven compare to multiplexing?

+ **Answer1**: multi-threads + blocking? = multiplexing; <u>multi-threads uses much more resources than multiplexing</u>.
+ **Answer2**: Each call to `select()` or `poll()`, they will scan all fds to find ready fds while <u>`epoll()` just need to traverse ready_fds</u>;

So multiplexing is good at **monitoring multiple fds**, signal-driven is good at **finding ready fds among monitored fds**.

```cpp
// 创建 epoll instance
int epoll_create1(int __flags);

int epoll_ctl(int __epfd, int __op, int __fd,
                struct epoll_event *__event);

- epoll_ctl: add fd to epoll instance
  - EPOLL_CTL_ADD, EPOLL_CTL_DEL, EPOLL_CTL_MOD // 

int epoll_wait (int __epfd, struct epoll_event *__events, 
              int __maxevents, int __timeout);

```
