---
title: "OpenResty 从入门到实战"
date: 2024-06-25T22:00:38+08:00
draft: false
image: 1.png
categories:
    - computer
---

# OpenResty 从入门到实战：基于 Nginx 与 Lua 的高性能 Web 开发指南

在高并发 Web 服务领域，Nginx 凭借其轻量高效的非阻塞 I/O 模型占据半壁江山，而 Lua 则以小巧快速的特性成为脚本语言中的佼佼者。当两者相遇，便诞生了 OpenResty 这一高性能 Web 平台。本文将从基础认知、安装部署、核心原理到实战示例，带您全面掌握 OpenResty 的应用精髓。

## 一、OpenResty 是什么？

OpenResty 并非全新的技术框架，而是一个基于 Nginx 与 Lua 构建的高性能 Web 平台。它内部集成了海量精良的 Lua 库、第三方模块及依赖项，核心目标是让 Web 服务直接运行在 Nginx 内部，充分复用 Nginx 的非阻塞 I/O 优势 —— 不仅能高效响应 HTTP 客户端请求，还能对 MySQL、Redis 等远程后端实现一致的高性能交互。

其技术基石由两部分构成：

*   **Nginx**：轻量级高性能 Web 服务器，以高并发、低内存占用著称，采用事件驱动的非阻塞架构，是处理海量请求的理想载体。

*   **Lua & LuaJIT**：Lua 是一种轻量可移植的脚本语言，而 LuaJIT 作为即时编译器，能将频繁执行的 Lua 代码编译为本地机器码，大幅提升执行效率，OpenResty 默认启用 LuaJIT。

借助这一技术组合，OpenResty 可轻松搭建超高并发、高扩展性的动态 Web 应用、服务及网关，广泛应用于流量网关、API 网关、缓存层等场景。

## 二、OpenResty 安装与验证（CentOS 环境）

OpenResty 的安装流程简洁高效，以下是基于 CentOS 系统的完整步骤：

### 1. 安装依赖包

首先安装编译所需的基础依赖：

```shell
yum install pcre-devel openssl-devel gcc curl
```

### 2. 配置 YUM 源

添加 OpenResty 官方 YUM 源以简化安装：

```shell
wget https://openresty.org/package/centos/openresty.repo

sudo mv openresty.repo /etc/yum.repos.d/openresty.repo

sudo yum check-update
```

### 3. 安装 OpenResty

执行安装命令并验证版本：

```shell
yum -y install openresty

# 验证安装，成功则输出 Nginx 版本（含 OpenResty 标识）

/usr/local/openresty/nginx/sbin/nginx -v
```

### 4. 启动与测试

OpenResty 的核心是 Nginx，因此启动与配置方式与 Nginx 一脉相承：

1.  **修改配置文件**：编辑 `/usr/local/openresty/nginx/conf/nginx.conf`，添加含 Lua 代码的 location 块：

```shell
server {

   listen 80;

   server_name localhost;

   location /test {

       content_by_lua_block {

           ngx.say("Hello, LuaJIT!")

       }

   }

}
```

   2.**检查配置并启动**：

```shell
# 语法检查，确保配置无误

sudo /usr/local/openresty/nginx/sbin/nginx -t

# 启动服务

sudo /usr/local/openresty/nginx/sbin/nginx

# 若已启动，重新加载配置

sudo /usr/local/openresty/nginx/sbin/nginx -s reload
```

​	3.**验证结果**：访问 `http://localhost/test`，页面应返回 `Hello, LuaJIT!`，表明 OpenResty 已正常运行。

## 三、OpenResty 核心工作原理

OpenResty 的高效运行本质是对 Nginx 请求处理流程的深度扩展，其核心逻辑围绕「Nginx 执行阶段 + Lua 指令注入」展开。

### 1. Nginx 的 11 个请求处理阶段

Nginx 处理 HTTP 请求时会按固定顺序经过 11 个阶段，可从源码 `ngx_http_core_module.h` 中看到定义：

```c
typedef enum {

   NGX_HTTP_POST_READ_PHASE = 0,        // 读取请求后

   NGX_HTTP_SERVER_REWRITE_PHASE,       // 服务器级重写

   NGX_HTTP_FIND_CONFIG_PHASE,          // 查找配置

   NGX_HTTP_REWRITE_PHASE,              // 重写

   NGX_HTTP_POST_REWRITE_PHASE,         // 重写后

   NGX_HTTP_PREACCESS_PHASE,            // 访问前

   NGX_HTTP_ACCESS_PHASE,               // 访问控制

   NGX_HTTP_POST_ACCESS_PHASE,          // 访问后

   NGX_HTTP_PRECONTENT_PHASE,           // 内容前

   NGX_HTTP_CONTENT_PHASE,              // 内容处理

   NGX_HTTP_LOG_PHASE                   // 日志记录

} ngx_http_phases;
```

### 2. OpenResty 的 Lua 指令映射

巧合的是，OpenResty 提供了 11 个 `*_by_lua` 指令，与 Nginx 的 11 个阶段一一对应。这些指令允许开发者在请求处理的特定阶段注入 Lua 代码，实现自定义逻辑。

例如：

*   `access_by_lua_block`：在 `NGX_HTTP_ACCESS_PHASE` 阶段执行，用于访问控制（如 IP 限流）；

*   `content_by_lua_block`：在 `NGX_HTTP_CONTENT_PHASE` 阶段执行，用于生成响应内容；

*   `log_by_lua_block`：在 `NGX_HTTP_LOG_PHASE` 阶段执行，用于自定义日志。

这种「阶段注入」机制让开发者既能复用 Nginx 的原生性能，又能通过 Lua 实现灵活的业务逻辑，实现了性能与扩展性的平衡。

## 四、OpenResty 核心模块详解

OpenResty 的强大之处在于其丰富的集成模块，这些模块扩展了 Nginx 的原生能力，以下是最常用的核心模块及实战示例。

### 1. ngx_lua 模块（核心基础）

ngx_lua 是 OpenResty 的灵魂模块，提供 Lua 脚本与 Nginx 的无缝集成，支持在配置中嵌入 Lua 代码实现动态逻辑。

**示例：基础响应生成**

```shell
server {

   listen 80;

   server_name example.com;

   location /lua_demo {

       default_type 'text/plain';

       content_by_lua_block {

           -- 获取客户端 IP

           local client_ip = ngx.var.remote_addr

           -- 生成动态响应

           ngx.say("Hello, OpenResty! Your IP is: ", client_ip)

       }

   }

}
```

访问 `/lua_demo` 会返回包含客户端 IP 的文本响应，体现了 Lua 与 Nginx 变量的结合使用。

### 2. ngx_stream_lua 模块（TCP/UDP 处理）

与 ngx_lua 针对 HTTP 不同，ngx_stream_lua 专注于 TCP/UDP 流量处理，可用于构建自定义代理或协议解析服务。

**示例：TCP 数据接收**

```shell
stream {

   server {

       listen 12345;  # 监听 TCP 端口

       content_by_lua_block {

           -- 获取连接套接字

           local sock, err = ngx.req.socket()

           if not sock then

               ngx.log(ngx.ERR, "Socket error: ", err)

               return

           end

           -- 读取客户端数据

           local data, err = sock:receive()

           if data then

               ngx.say("Received from client: ", data)

           else

               ngx.log(ngx.ERR, "Read error: ", err)

           end

       }

   }

}
```

通过 `telnet ``localhost`` 12345` 发送数据，服务端会打印接收的内容。

### 3. ngx_http_headers_more 模块（HTTP 头部控制）

扩展了 Nginx 对 HTTP 头部的操作能力，支持添加、修改、删除响应头，比原生 `add_header` 指令更灵活。

**示例：自定义头部与隐私保护**

```shell
server {

   listen 80;

   server_name example.com;

   # 添加自定义业务头部

   location /add_header {

       more_set_headers "X-Service: OpenResty";

       more_set_headers "X-Version: 1.0";

       return 200 "Custom headers added";

   }

   # 移除默认 Server 头部（隐藏技术栈）

   location /hide_server {

       more_clear_headers "Server";

       return 200 "Server header removed";

   }

}
```

### 4. ngx_http_redis 模块（Redis 交互）

提供与 Redis 数据库的直接交互能力，无需通过后端服务中转，可用于缓存读取、计数器等场景。

**示例：Redis 数据读取**

```shell
http {

   server {

       listen 80;

       server_name example.com;

       location /redis_demo {

           default_type 'text/plain';

           content_by_lua_block {

               -- 加载 Redis 模块

               local redis = require "ngx.redis"

               local red = redis:new()

               -- 设置超时

               red:set_timeout(1000)

               -- 连接 Redis（本地默认端口）

               local ok, err = red:connect("127.0.0.1", 6379)

               if not ok then

                   ngx.say("Redis connect failed: ", err)

                   return

               end

               -- 读取 key 为 "username" 的值

               local res, err = red:get("username")

               if res == ngx.null then

                   res = "Unknown"

               end

               ngx.say("Username from Redis: ", res)

               -- 释放连接到连接池

               red:set_keepalive(10000, 100)

           }

       }

   }

}
```

### 5. ngx_brotli 模块（高效压缩）

支持 Brotli 压缩算法，相比 Gzip 有更高的压缩比，尤其适合文本类内容（HTML、JS、CSS）。

**示例：启用 Brotli 压缩**

```shell
http {

   # 全局启用 Brotli 压缩

   brotli on;

   # 压缩级别（1-11，6 为平衡值）

   brotli_comp_level 6;

   # 对静态文件启用压缩

   brotli_static on;

   # 压缩的 MIME 类型

   brotli_types text/plain text/css application/json application/javascript;

   server {

       listen 80;

       server_name example.com;

       location /compressed {

           default_type 'text/plain';

           content_by_lua_block {

               -- 长文本示例，会被 Brotli 压缩传输

               local long_text = string.rep("OpenResty Demo ", 100)

               ngx.say(long_text)

           }

       }

   }

}
```

## 五、实战案例：API 接口限流

结合前面的知识，我们实现一个实用场景：对 API 接口进行 IP 级别的请求频率限制，防止恶意请求。

### 实现思路

1.  使用 `lua_shared_dict` 定义共享内存，存储每个 IP 的请求计数；

2.  在 `access_by_lua_block` 阶段（访问控制阶段）统计请求次数；

3.  若 60 秒内请求超过 10 次，返回 429（Too Many Requests）。

### 完整配置

```
http {

   # 定义 10MB 共享内存，用于存储限流数据

   lua_shared_dict ip_limit 10m;

   server {

       listen 80;

       server_name api.example.com;

       location /api {

           # 访问控制阶段执行限流逻辑

           access_by_lua_block {

               local limit_dict = ngx.shared.ip_limit

               local client_ip = ngx.var.remote_addr  # 客户端 IP 作为键

               local max_req = 10  # 最大请求数

               local expire = 60   # 过期时间（秒）

               # 获取当前请求数

               local req_count, err = limit_dict:get(client_ip)

               if req_count then

                   # 超过限制，返回 429

                   if req_count > max_req then

                       ngx.exit(ngx.HTTP_TOO_MANY_REQUESTS)

                   else

                       # 未超过，计数 +1

                       limit_dict:incr(client_ip, 1)

                   end

               else

                   # 首次请求，初始化计数并设置过期时间

                   limit_dict:set(client_ip, 1, expire)

               end

           }

           # 内容处理阶段返回 API 响应

           default_type 'application/json';

           content_by_lua_block {

               ngx.say('{"code": 200, "message": "success", "data": {}}')

           }

       }

   }

}
```

### 测试验证

使用 `ab` 工具模拟高并发请求：

```shell
ab -n 20 -c 5 http://api.example.com/api/
```

结果中前 10 次请求返回 200，后 10 次返回 429，限流逻辑生效。

## 六、总结

OpenResty 通过「Nginx 性能底座 + Lua 灵活扩展」的组合，解决了传统 Web 服务中「高性能」与「高扩展性」难以兼顾的问题。从简单的动态响应到复杂的 API 网关、流量控制，OpenResty 都能胜任。

本文仅覆盖了 OpenResty 的基础与核心能力，其生态中还有更多实用模块（如 `ngx_http_geoip2` 地理位置解析、`ngx_http_lua_upstream` 上游管理等）等待探索。建议结合官方文档深入学习，根据实际业务场景灵活运用，充分发挥其高性能优势。
