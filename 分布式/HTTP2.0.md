---
title: HTTP2.0
categories: 分布式
tags: [HTTP2.0]
---

HTTP2.0 的目的是通过支持请求与响应的多路复用来减少延迟，通过压缩HTTPS首部字段将协议开销降低，同时增加请求优先级和服务器端推送的支持。 二进制分帧层，是HTTP 2.0性能增强的核心。HTTP 1.x在应用层以纯文本的形式进行通信，而HTTP 2.0将所有的传输信息分割为更小的消息和帧，并对它们采用二进制格式编码。这样，客户端和服务端都需要引入新的二进制编码和解码的机制。 

## 二进制分帧层

HTTP/2.0 将报文分成 HEADERS 帧和 DATA 帧，它们都是二进制格式的。

在通信过程中，只会有一个 TCP 连接存在，它承载了任意数量的双向数据流（Stream）。

- 一个数据流（Stream）都有一个唯一标识符和可选的优先级信息，用于承载双向信息。
- 消息（Message）是与逻辑请求或响应对应的完整的一系列帧。
- 帧（Frame）是最小的通信单位，来自不同数据流的帧可以交错发送，然后再根据每个帧头的数据流标识符重新组装。

## 服务端推送

HTTP/2.0 在客户端请求一个资源时，会把相关的资源一起发送给客户端，客户端就不需要再次发起请求了。例如客户端请求 page.html 页面，服务端就把 script.js 和 style.css 等与之相关的资源一起发给客户端。

<!--more-->