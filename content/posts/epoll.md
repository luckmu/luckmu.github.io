---
title: "Epoll"
date: 2023-01-12T11:37:21+08:00
draft: true
---

**epoll** in detail.

<!--more-->

What: <u>Monitoring multiple file descriptors to see if I/O is possible on any of them;</u>

Why: <u>select & poll & epoll;</u>

为什么 epoll 比 select & poll 好?

epoll 本质是 I/O 多路复用 + 信号驱动 I/O; 而 select & poll 都是 I/O 多路复用;

如果非信号驱动, 就需要检查 list 中所有 fd 是否 ready; 而信号驱动 (如 epoll) 维护 rdy_list, 其中所有 fd 都是 ready 的;

```c
// Returns number of ready file descriptors, 0 on timeout, or -1 on error
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```
I/O Multiplexing:

fds, fds_proxy;

select & poll 的几个问题:

1. 每次 select() 或 poll() 调用都需要检查所有 fds, 即 O(n)
2. 

> On each call to select() or poll(), the kernel must check all of the specified file descriptors to see if they are ready.

> In each call to select() or poll(), the program must pass a data structure to the
kernel describing all of the file descriptors to be monitored, and, after checking
the descriptors, the kernel returns a modified version of this data structure to
the program. 

> After the call to select() or poll(), the program must inspect every element of the
returned data structure to see which file descriptors are ready.




2 个需求:

- 检查对 fd 能否 I/O, 并且在不能 I/O 时不阻塞
- monitor 多个 fd, 查看在任意 fd 上能否 I/O

watch 若干 fd, check 其中能 I/O 的事件;

select, poll 都是 I/O 多路复用;<br>
而 epoll 是 I/O 多路复用 + <u>信号驱动 I/O</u>;



How:

主要是 lt 和 et 使用上的区别

```cpp
// 0 is stdin
// epoll_create1: create epoll instance
int epoll_fd = epoll_create1(0);
struct epoll_event event;
// interested in EPOLLIN, 
event.events = EPOLLIN;
event.data.fd = 0;

epoll_ctl(epoll_fd, EPOLL_CTL_ADD, 0, &event);



```

## Refer

- [epoll in 3 easy steps](https://suchprogramming.com/epoll-in-3-easy-steps/)
- [epoll LT/ET 深入剖析](https://blog.csdn.net/dongfuye/article/details/50880251)
- [epoll 源码](https://github.com/torvalds/linux/blob/master/fs/eventpoll.c)
- [epoll 源码解析](https://github.com/xuyuuu/epoll-sourcecode-analysis)
- [man epoll](https://man7.org/linux/man-pages/man7/epoll.7.html)
- The Linux Programming Interface chapter63: Alternative I/O Models
