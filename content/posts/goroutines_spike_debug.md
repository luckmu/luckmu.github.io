---
title: "Goroutines spike debug"
date: 2023-02-23T23:24:01+08:00
draft: false
---

有关 `http.Client` 正确使用, 根据 `net/http/pprof` 和 `netstat` 排查问题的记录。

<!--more-->

之前一段时间现网服务出现 goroutines 数量尖刺的表现，让排查下问题。

表现为 goroutines 有尖刺，打开文件数有尖刺，两者曲线吻合。

于是先在测试环境来 1 发 `import _ "net/http/pprof"`, 发现经常数量最高是 `write_loop` 和 `read_loop` 两类协程。

观察 `/proc/$pid/fd` 文件数量, 全是 socket 连接, 于是 `netstat -tn | grep ESTABLISED` 发现有大量到同 1 dst 的连接。

本服务需要经常查询 dst，但都是串行，且都有 close 掉 response.Body，为什么会出现这种情况呢？

再仔细排查，每次 query 都新 new 了 1 个 `http.Client`, 自定义 transport 后使用默认 keepalive 时长 15s。

这样即使 query 是串行, connections 和 goroutines 也会在 15s 内累积。

验证：
1. `import _ "net/http/pprof"` 观察实时 goroutines 数量, 与 `watch -n1 "netstat -tn | grep dst:port | grep ESTABLISED"` 观察实时连接数, 发现吻合。
2. `transport.DisableKeepAlives = true`, 发现到 dst 的连接不再驻留, 并且没有之前那样大量 goroutines 驻留。

OK, 剩下的就是 fix 代码了, 使用 1 个 `http.Client` 实例即可, 且本身 `http.Client` 也是可复用的, 还能有效利用连接池, 减少资源浪费。
