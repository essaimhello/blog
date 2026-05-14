---
title: Nginx-后端服务健康检查
tags: [nginx]
categories: nginx
author: Judy
cover: /images/default-cover.jpg
published: true
date: 2026-02-06 12:03:05
---

# Nginx-后端服务健康检查

参考文章

1. [[技术分享] Nginx实战系列之功能篇----后端节点健康检查](http://www.iyunv.com/thread-38535-1-1.html)

nginx默认通过检测端口状态判断后端upstream节点是否在线的, 服务直接挂掉甚至服务器宕机都可以检测到.

```
upstream backend {
        server 10.1.1.110:8080 max_fails=2 fail_timeout=20s;
        server 10.1.1.122:8080 max_fails=2 fail_timeout=20s;
}
```

`max_fails=指定数值`: 设置Nginx与后端服务器通信的尝试失败的次数。在`fail_timeout`参数定义的时间段内，如果失败的次数达到此值，Nginx就认为该后端节点不可用。在下一个`fail_timeout`时间段，服务器不会再被尝试。 失败的尝试次数默认是1(设为0就会停止统计尝试次数，认为此节点是一直可用的). 可以通过指令`proxy_next_upstream`、`fastcgi_next_upstream`和`memcached_next_upstream`来配置什么是失败的尝试。 默认配置时, `http_404`状态不被认为是失败的尝试。

`fail_timeout=时间`: 设置服务器被认为不可用的时间段以及统计失败尝试次数的时间段。在这段时间中，服务器失败次数达到指定的尝试次数，服务器就被认为不可用。并且下一个时间段内的请求将不再发给出问题的节点. 默认情况下，该超时时间是10秒。

但是很多时候服务本身出了问题如执行缓慢, 磁盘空间满导致无法写入日志时, nginx还是会照样向该节点转发请求.

使用内置的`proxy_next_upstream`命令, 根据后端服务返回状态码确定"失败的尝试". 对于判断已经失败的节点, 可以停止对其的转发.

语法

```
proxy_next_upstream error | timeout | invalid_header | http_500 | http_502 | http_503 | http_504 | http_404 | off ...;
```

使用方法

```
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
```

...不建议使用404状态码判断, 因为某个页面找不到而导致整个节点用不了还是相当那啥的.


`timeout`连接/响应超时配置（配合健康检查）
```
proxy_connect_timeout 5s;    # 连接后端超时时间
proxy_send_timeout 10s;      # 发送请求超时
proxy_read_timeout 10s;      # 读取响应超时
```

完整配置
```
# 定义上游服务器池 + 被动健康检查参数
upstream backend_pool {
    # 后端节点（可多个）
    server 192.168.1.100:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.101:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.102:8080 backup;  # 备用节点（主节点都故障时启用）
    
    # 可选：负载均衡策略（默认轮询，推荐ip_hash保持会话）
    ip_hash;  # 或 least_conn（最少连接）、weight=2（权重）
}

# 反向代理配置
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://backend_pool;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # 连接/响应超时配置（配合健康检查）
        proxy_connect_timeout 5s;    # 连接后端超时时间
        proxy_send_timeout 10s;      # 发送请求超时
        proxy_read_timeout 10s;      # 读取响应超时
        
        # 自定义失败的尝试
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_next_upstream_tries 2;  # Nginx 最多再换 2 个“不同的后端”重试
    }
}
```