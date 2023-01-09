---
title: "Kernel-User Mode"
date: 2023-01-06T16:51:43+08:00
draft: false
categories:
  - system
tags:
  - linux
---

用户态和内核态是什么? 为什么要做这种区分? 如何实现?

<!--more-->

What: <u>限制应用 (application) 可执行的指令 (instructions) 以及可以访问的地址空间;</u>

Why: <u>避免用户进程导致系统崩溃; 取而代之, 应用崩溃只导致进程崩溃;</u>

> **Crashing "application" might crash the entire system -> Crashing "application" only crash the process;**

How: <u>表征进程的特权状态, 通过 mode bit 实现区分;</u>

+ x86 CPU 提供了 4 [protection rings](https://en.wikipedia.org/wiki/Protection_ring), 对应着执行指令的特权级别: 0, 1, 2 and 3; Linux 只用了 0 (kernel-mode) 和 3 (user-mode);
+ 控制寄存器中的模式位 (mode bit) 标识当前进程处于什么特权级别 (kernel or user mode);
  + mode bit set -> **kernel mode**: <u>可以执行任何指令, 访问任何内存地址;</u>
  + mode bit not set -> **user mode**: <u>不允许执行特权指令 (halt the processor, change the mode bit, initiate I/O operation), 不允许直接引用代码空间中内核区域的代码和数据;</u>

To kernel mode: 通过 exception (**interrupt**, **fault**, **trapping system call**), 并传递给 exception handler, handler 在 kernel mode 运行;

## Refer

- csapp 8.2.4
- [Understanding User and Kernel Mode](https://blog.codinghorror.com/understanding-user-and-kernel-mode/)
- [从根上理解用户态与内核态](https://segmentfault.com/a/1190000039774784)