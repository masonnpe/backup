---
title: DNS域名解析
categories: 分布式
tags: [DNS]
---

## DNS域名解析流程

1. 用户输入域名按下回车
2. 浏览器检查缓存中是否这个域名对应的解析过的IP地址 没有则继续
3. 查找操作系统缓存 没有则继续
4. 请求本地域名服务器LDNS 没有则继续
5. 请求Root DNS Server，返回主域名服务器（gTLD server）地址
6. LDNS请求gTLD，gTLD返回Name Server域名服务器的地址 给LDNS
7. LDNS查询Name Server 得到IP地址
8. LDNS返回IP地址给用户

<!--more-->