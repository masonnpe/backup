---
title: Get与Post
categories: 分布式
tags: [Get,Post]
---

HTTP定义了与服务器交互的不同方法，最基本的方法有4种，分别是GET，POST，PUT，DELETE

## GET

GET用于信息获取，而且应该是安全的和幂等的

GET请求的数据会附在URL之后

GET方式提交的数据最多只能是1024字节

## POST

POST表示可能修改变服务器上的资源的请求

POST把提交的数据则放置在是HTTP包的包体中

理论上POST没有限制，可传较大量的数据

<!--more-->