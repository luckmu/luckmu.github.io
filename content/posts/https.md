---
title: "About HTTPS"
date: 2023-01-02T23:47:46+08:00
draft: false
categories:
  - network
tags:
  - https
---

主要是 https 的原理和流程, 以及基础的 faq 。

<!--more-->

## Asymmetric encryption

欧拉定理, 使得取模运算有某种程度的可逆性; 即 `x^(p*q) mod N = y;`, 有:

+ `x^p mod N = y;`
+ `y^q mod N = x;`

可以理解为: `x` 用 `p` 加密得 `y`; `y` 用 `q` 解密得 `x`; ~~即 `p` 是公钥, `q` 是私钥;~~

有 `f(x, p) -> y;` 或 `f(y, q) -> x;` 是易求得的;

而如果 `N` 巨大, 未知 `p`、`q` 的情况; 以现有算力互推 `x`、`y`, 在较长时间可以认为无法实现 (只能遍历幂值, 直到等式成立)

`p`, `q` 就是非对称加密的 1 对密钥; 且公钥、私钥都能用于加密 (*公钥加密: 加密; 私钥加密: 签名*)

## HTTPS

`HTTPS = HTTP + SSL/TLS`; `HTTPS Handshake = TCP 3WH + TLS 4WH`

TLS 4WH:
+ 1WH: `ClientHello`, 发送 `client_random`, 支持的 TLS 版本, 支持的加密套件
+ 2WH: `ServerHello`, 发送 `server_random`, 选定的 TLS 版本, 选定的加密套件
+ 3WH:
  + `ClientKeyExchange`: 生成 `pre_master_key (预主密钥)`, 从服务器证书取出服务器公钥, 用服务器公钥加密 `pre_master_key` 发送给服务端
  + `ChangeCipherSpec`: 用 `client_random`, `server_random`, `per_master_key` 计算得 `key_block (会话密钥)`
  + `EncryptedHandshakeMessage`: 握手信息生成摘要(hash), 用 `key_block` 加密给服务端校验, 至此客户端握手结束, 称 `Finished` 报文
+ 4WH:
  + `ChangeCipherSpec`: 用服务器私钥解密得到 `per_mater_key`, 并通过 `client_random`, `server_random`, `per_master_key` 计算得 `key_block`
  + `EncryptedHandshakeMessage`: 握手信息生成摘要, 用 `key_block` 加密给客户端校验, 至此服务端握手结束, 称 `Finished` 报文

## FAQ

+ **HTTPS 是对称加密还是非对称加密?**
  + 既用到了非对称加密也用到了对称加密
    + 4 次握手, 用**非对称加密**交换 3 个随机数
    + 加密通信, 用**会话密钥**进行**对称加密**通信
+ **为什么不都用非对称加密?**
  + 相对来说, 对称加密比非对称加密更快
    + 对称加密主要是位运算, 速度非常快
    + 非对称加密计算1般比较复杂, 涉及大数乘法、大数模等运算
+ **服务器证书是什么? 怎么从里面取出公钥?**
  + 服务器证书, 本质上是 CA 的私钥加密过的服务器公钥
    + 服务器公钥 + CA 私钥 -> 服务器证书
    + 服务器证书 + CA 公钥 -> 服务器公钥
+ **为什么不直接传公钥? 而要以服务器证书的方式加密传输?**
  + 若只传公钥, 在传输过程中公钥可能被黑客替换, 如果客户端用假公钥加密 `pre_master_key` 并返回, 黑客解密后得到 `pre_master_key`, 又 `client_random`, `server_random` 是明文传输, 黑客可以计算出**会话密钥**
  + 所以 CA 私钥加密得服务器证书就是为了保证客户端得到真正的服务器公钥
+ **怎么获取CA公钥?**
  + 请求 CA 官网? 服务器压力太大, 能颁发证书的CA机构并不多, 因此对应CA公钥也不多, 作为配置直接放到操作系统或浏览器中
+ **3 个随机数的安全性?**
  + Client Random 和 Server Random 都是明文的, 可以获取, 但 pre_master_key 通过服务器公钥加密, 只有服务器私钥能解密, 被别人获取了也无法得到原文
+ **为什么用 3 个随机数?**
  + 其实即使没有 Client Random 和 Server Random 也不影响加密功能, 单个 pre_master_key 随机性不足, 多次随机可能得到密钥是相同的, 再引入 2 个随机数, 大大增加**会话密钥**的随机性, 保证每次 HTTPS 通信用的会话密钥都是不同的
+ **为什么 3、4 次握手要传摘要?**
  + 对大段文本进行 hash 操作, 确认通信过程中数据没有被篡改过
    + 第 3 次握手, 客户端生成摘要, 服务器验证, 如果验证通过, 客户端是可信的
    + 第 4 次握手, 服务端生成摘要, 客户端验证, 如果验证通过, 服务端是可信的
+ **为什么要 hash 摘要而不是原文摘要?**
  + hash 让数据变短, 更小的传输成本
+ **整个过程涉及了几对私钥、公钥?**
  + 2 对:
    + 服务器公钥、私钥: 公钥(在数字证书中), 客户端用于加密 3 个随机数得**会话密钥**, 服务端用对应私钥解密
    + CA 公钥、私钥: 二次握手, 服务器数字证书是用CA私钥加密, 客户端用CA公钥解密得到服务器公钥

## Refer

+ [为什么用公钥加密却不能用公钥解密?](https://mp.weixin.qq.com/s/YXVURw55G2hT7BtShdGG4A)
+ [TLS 中的密钥计算](https://halfrost.com/https-key-cipher/)
+ [为什么非对称加密比对称加密慢?](https://cloud.tencent.com/developer/article/1672173)
