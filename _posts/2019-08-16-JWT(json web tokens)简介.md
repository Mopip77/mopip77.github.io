---
layout: post
title: JWT(json web tokens)简介
tags:
- jwt
---

我们知道我们经常使用的session+cookie实现http的认证有一些缺点。
1. 由于session存储在服务器，并且对每个session服务器都会在jvm中分配一部分内存，在单机模式下，所以一旦用户量过多，那么有可能导致内存不足的问题。
2. 使用服务器集群或者跨域时又需要满足多台服务器共享session的问题，横向扩展性不好。

所以索性服务器就不存session信息了，全部存储在客户端里，每次访问服务器都附带该信息即可。

详细介绍：http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html