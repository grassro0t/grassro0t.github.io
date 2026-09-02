---
title: "C++工程常用工具链-推荐后端架构"
slug: "backend-architecture"
date: 2026-09-02T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["架构", "c++", "网关", "RPC"]
categories: ["技术笔记"]
summary: "基础的后端架构推荐，包括整体架构、API网关、RPC集群、存储层等。"
toc: true
comments: true
description: "C++工程常用工具链-推荐后端架构"
---

``` txt
客户端(APP/Web)
     ↓ HTTP/HTTPS
【边缘接入层】Nginx / OpenResty（防攻击、SSL、静态资源、四层负载）
     ↓ HTTP
【API网关层】Kong / APISIX / BFF服务
        ↓ 协议转换：HTTP → RPC（protobuf/thrift）
【内网RPC集群】←————————————
    ├─ 用户服务(C++ brpc)
    ├─ 订单服务(C++ brpc)
    └─ 库存服务(C++ brpc)
        ↓ RPC互相调用（服务间通信核心）
【存储层】Redis / MySQL / RocksDB / 对象存储
```

## REST‑API

- 遵循 REST 风格设计的 HTTP 接口
- URL 定位「资源」（必须名词），HTTP Method 表达「对资源做什么操作
- GET-查询，POST-新增，DELETE-删除，PUT-全量修改，PATCH-部分修改
- 返回值大都是JSON（状态码沿用http）
  \## Nginx
- Nginx(前端反向代理、负载均衡、SSL、静态资源) → C++后台程序（muduo、boost、oat++）
  \### 基本运行原理
- 概念：Nginx 是 HTTP 和反向代理服务器，也是一个 IMAP/POP3/SMTP 代理服务器。高并发、低内存占用、稳定性强，常用于 Web 服务部署、反向代理、负载均衡等。
- 网络模型：**多进程 + 事件驱动**（epoll/kqueue）的异步非阻塞模型
- **Master 进程**：管理进程，负责读取配置、启动 Worker 进程、接收外界信号、平滑重启
- **Worker 进程**：工作进程，负责处理网络请求，数量通常设置为 CPU 核心数，单线程，通过 epoll 处理成千上万的并发连接
- 请求处理流程：客户端请求 → 监听端口 → Nginx 匹配 location 规则 → 按配置处理（静态文件/反向代理/重定向等）→ 返回响应
  ![](../backend-architecture/6b0111ae0a088e6e020058d50692fabb84e4753c.png)
  \### 目录结构（CentOS yum）
- 安装目录：`/usr/local/nginx/`

``` bash
/etc/nginx/
├── nginx.conf          # 主配置文件
├── conf.d/             # 子配置目录，通常存放各站点配置
│   └── default.conf
├── mime.types          # MIME 类型映射
├── fastcgi_params      # FastCGI 参数
├── uwsgi_params        # uWSGI 参数
└── scgi_params         # SCGI 参数

/usr/share/nginx/html/  # 默认网站根目录

/var/log/nginx/         # 日志目录
├── access.log          # 访问日志
└── error.log           # 错误日志

/var/run/nginx.pid      # 进程 PID 文件a
```

### 配置文件

- 常用内置变量

1.  `$remote_addr`客户端 IP
2.  `$request_method`请求方法 GET/POST 等
3.  `$request_uri`原始请求 URI（带参数）
4.  `$uri`规范化后的 URI
5.  `$args`请求参数
6.  `$host`请求头中的 Host
7.  `$status`响应状态码
8.  `$http_user_agent`客户端 UA
9.  `$scheme`请求协议 http/https

- 基础配置

``` conf
# 运行用户
user nginx;
# Worker 进程数，建议等于 CPU 核心数
worker_processes auto;
# 错误日志
error_log /var/log/nginx/error.log warn;
# PID 文件
pid /var/run/nginx.pid;

# 事件模块
events {
    # 每个 Worker 最大连接数
    worker_connections 1024;
    # 使用 epoll 事件模型
    use epoll;
}

# HTTP 核心模块
http {
    # 引入http mime类型
    include       /etc/nginx/mime.types;
    # 如果mime类型没匹配上默认使用二进制流传输
    default_type  application/octet-stream;

    # 日志格式
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    # 高效文件传输（linux的sendfile数据0拷贝）
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    # 允许上传最大文件
    client_max_body_size 10M;
    
    # 连接超时时间
    keepalive_timeout 65;
    
    # server配置也在这里

    # 引入子配置文件
    include /etc/nginx/conf.d/*.conf;
}
```

### 常用命令

    nginx -s start    # 启动
    nginx -s stop     # 强制停止
    nginx -s quit     # 优雅停止（处理完请求后退出）
    nginx -s reload   # 平滑重载配置
    nginx -t          # 检查配置文件语法
    nginx -v          # 查看版本
    nginx -V          # 查看版本及编译参数

### 虚拟主机

- 虚拟主机指一台服务器上运行多个网站，通过**域名**、**端口**、**IP** 区分
- 配置在`nginx.conf`的http块内部，或者`/etc/nginx/conf.d/`下`www.example.com.conf`文件里
- 基于域名

``` conf
server {
    listen       80;
    server_name  www.example.com;

    location / {
        root   /usr/share/nginx/example;
        index  index.html index.htm;
    }
}

server {
    listen       80;
    server_name  www.test.com;

    location / {
        root   /usr/share/nginx/test;
        index  index.html index.htm;
    }
}
```

- 基于端口

``` conf
server {
    listen 8080;
    server_name localhost;

    location / {
        root /usr/share/nginx/app1;
        index index.html;
    }
}
```

- 基于IP（服务器本身有多个对外ip）

<!-- -->

    # 站点1：绑定 IP 192.168.1.100 的80端口
    server {
        listen       192.168.1.100:80;
        server_name  _;  # 纯IP匹配时，server_name 无实际作用，可写占位符

        location / {
            root   /usr/share/nginx/site1;
            index  index.html index.htm;
        }
    }

    # 站点2：绑定 IP 192.168.1.101 的80端口
    server {
        listen       192.168.1.101:80;
        server_name  _;

        location / {
            root   /usr/share/nginx/site2;
            index  index.html index.htm;
        }
    }

- 匹配优先级

1.  精确匹配：`www.example.com`
2.  前缀通配：`*.example.com`
3.  后缀通配：`www.example.*`
4.  正则匹配：`~^(?<subdomain>.+)\.example\.com$`
5.  默认 server（第一个或 default_server）
    \### 反向代理（核心）

- 用户 →【Nginx（反向代理）】→ 后端业务服务 (C++/Java 程序)，代理的是【后端服务】，用户**完全感知不到后端真实服务器**
- 配置在`nginx.conf`的http块内部，或者`/etc/nginx/conf.d/`下`www.example.com.conf`文件里
- 基础配置

<!-- -->

    server {
        listen 80;
        server_name api.example.com;

        location / {
            # 转发到后端服务
            proxy_pass http://127.0.0.1:8080;

            # 设置请求头，传递客户端真实信息
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            # 超时设置
            proxy_connect_timeout 30s;
            proxy_read_timeout 60s;
            proxy_send_timeout 60s;
        }
    }

- 常用proxy选项

1.  `proxy_pass`：后端服务地址
2.  `proxy_set_header`：设置转发请求头
3.  `proxy_buffering`：是否开启响应缓冲
4.  `proxy_redirect`：重写后端返回的重定向地址
    \### 负载均衡

- 配置在`nginx.conf`的http块内部，或者`/etc/nginx/conf.d/`下`upstream.conf`文件里
- 负载均衡策略

1.  **轮询（默认）**：请求按顺序逐一分配
2.  **weight**：加权轮询，权重越高分配越多
3.  **ip_hash**：按客户端 IP hash 分配，保证同一用户始终访问同一后端
4.  **least_conn**：优先分配给连接数最少的后端
5.  **fair**：按响应时间分配（第三方模块）
6.  **url_hash**：按 URL hash 分配（第三方模块）

- 基础配置

``` conf
# 定义上游服务器组
upstream backend {
    server 192.168.1.101:8080;
    server 192.168.1.102:8080;
    server 192.168.1.103:8080;
}

# 加权轮询示例
upstream backend {
    server 192.168.1.101:8080 weight=3;
    server 192.168.1.102:8080 weight=2;
    server 192.168.1.103:8080 weight=1 backup; # backup 备用服务器
    server 192.168.1.104:8080 down;          # down 标记不可用
}
```

### 动静分离

- 静态资源（HTML、CSS、JS、图片）由 Nginx 直接处理，动态请求转发给后端应用
- 配置的是server里的不同location
- 基础配置

<!-- -->

    server {
        listen 80;
        server_name www.example.com;

        # 静态资源直接由 Nginx 处理
        location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2?)$ {
            root /usr/share/nginx/static;
            expires 7d;      # 浏览器缓存 7 天
            access_log off;  # 不记录访问日志
        }

        # 动态请求转发给 Tomcat/Node/Python 等
        location /api/ {
            proxy_pass http://127.0.0.1:8080;
            proxy_set_header Host $host;
        }

        # 首页
        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }

- location匹配规则

1.  `=` 精确匹配
2.  `^~` 前缀匹配（不继续正则匹配）
3.  `~` 正则匹配（区分大小写）
4.  `~*` 正则匹配（不区分大小写）
5.  普通前缀匹配
6.  `/` 默认匹配

- 重定向规则（location内部）：rewrite 正则 替换 标志位

1.  `last`：停止当前指令，重新匹配 location
2.  `break`：停止当前指令，重新进入当前location
3.  `redirect`：302 临时重定向
4.  `permanent`：301 永久重定向

<!-- -->

    server {
        listen 80;
        server_name example.com;

        # 永久重定向到 www
        rewrite ^/(.*)$ https://www.example.com/$1 permanent;

        # 伪静态示例
        rewrite ^/article/(\d+)\.html$ /article.php?id=$1 last;
    }

### 主备切换

- 两台 Nginx 服务器通过 Keepalived 实现 VIP 漂移，主节点故障时自动切换到备节点
- 需要提前安装keepalived，配置写在`/etc/keepalived/keepalived.conf`
- MASTER配置

<!-- -->

    vrrp_instance VI_1 {
        state MASTER
        interface eth0
        virtual_router_id 51
        priority 100
        advert_int 1

        authentication {
            auth_type PASS
            auth_pass 1111
        }

        virtual_ipaddress {
            192.168.1.100   # 虚拟 VIP
        }
    }

- BACKUP配置

<!-- -->

    vrrp_instance VI_1 {
        state BACKUP
        interface eth0
        virtual_router_id 51
        priority 90        # 优先级低于主节点
        advert_int 1

        authentication {
            auth_type PASS
            auth_pass 1111
        }

        virtual_ipaddress {
            192.168.1.100
        }
    }

- 配置存活检测

<!-- -->

    vrrp_script chk_nginx {
        script "/etc/keepalived/check_nginx.sh"
        interval 2
        weight -20
    }

    vrrp_instance VI_1 {
        # ... 其他配置
        track_script {
            chk_nginx
        }
    }

- 存活检测脚本check_nginx.sh

``` sh
#!/bin/bash
if [ $(ps -C nginx --no-header | wc -l) -eq 0 ]; then
    systemctl start nginx
    sleep 2
    if [ $(ps -C nginx --no-header | wc -l) -eq 0 ]; then
        exit 1
    fi
fi
exit 0
```

### Gzip压缩配置

    http {
        gzip on;
        gzip_min_length 1k;          # 小于 1k 不压缩
        gzip_comp_level 6;           # 压缩级别 1-9
        gzip_types text/plain text/css application/json application/javascript text/xml;
        gzip_vary on;                # 添加 Vary: Accept-Encoding
        gzip_disable "MSIE [1-6]\."; # IE6 不压缩
    }

### HTTPS证书配置

- 请求强制跳转HTTPS

<!-- -->

    server {
        listen 80;
        server_name www.example.com;
        # 301 永久重定向
        return 301 https://$host$request_uri;
    }

- HTTPS配置

<!-- -->

    server {
        listen 443 ssl;
        server_name www.example.com;

        # 证书文件
        ssl_certificate /etc/nginx/cert/server.crt;
        ssl_certificate_key /etc/nginx/cert/server.key;

        # 会话缓存
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 10m;

        # 加密协议和套件
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;

        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }

### 防盗链

- none：检测referer头域不存在的情况
- blocked：检测referer头域被防火墙或代理服务器删除或伪装的情况
- server_name：设置一个或多个黑名单url检测是否是其中的某一个

<!-- -->

    location ~* \.(jpg|jpeg|png|gif|flv|mp4)$ {
        valid_referers none blocked server_names *.example.com;
        if ($invalid_referer) {
            return 403;
        }
    }

### 访问控制

- 黑白名单（location）

<!-- -->

    location /admin/ {
        allow 192.168.1.0/24;
        deny all;
    }

- 连接数限制（http/server）

<!-- -->

    http {
        limit_conn_zone $binary_remote_addr zone=addr:10m;

        server {
            location /download/ {
                limit_conn addr 1;  # 每个 IP 只能一个连接
            }
        }
    }

- 速率限制（http/server）

<!-- -->

    http {
        limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;

        server {
            location /api/ {
                limit_req zone=one burst=20 nodelay;
            }
        }
    }

### 日志排故

- 查看访问日志：`tail -f /var/log/nginx/access.log`
- 查看错误日志：`tail -f /var/log/nginx/error.log`
- 配置语法检查：`nginx -t`
- 查看已加载配置：`nginx -T`
- 查看进程：`ps aux | grep nginx`

## protobuf

- Google 推出**结构化数据序列化方案**，主流标准：**Proto3**，Protobuf Message ==**非线程安全**==，禁止多线程并发读写同一个对象。
- 编译器`protoc`会将`.proto`描述文件生成各种语言代码（对c++来说就是`.pb.h / .pb.cc`）然后序列化为二进制数据
- 二进制编码、跨语言统一规范、支持嵌套
- 其他数据序列化方案：JSON，数据以文本方式传输，可读性更好，适合对外接口，protobuf更适合服务内部消息传递
- ==遵循开闭原则：对扩展开放，对删除和修改关闭，支持新增字段==
  \#### .proto描述文件

``` protobuf
// 第一行必须指定语法版本
syntax = "proto3";

// 包名，防止命名冲突（类似namespace）
package demo;

// C++ 专属选项：生成代码所在命名空间（可选，默认使用package）
option csharp_namespace = "Demo";
option cc_enable_arenas = true; // 开启Arena内存分配，提升性能（推荐开启）
```

##### 数据类型映射

| Protobuf 类型 | C++ 类型      | 说明                                         |
|---------------|---------------|----------------------------------------------|
| `int32`       | `int32_t`     | 有符号 32 位整数，负数编码效率一般           |
| `int64`       | `int64_t`     | 有符号 64 位，id、时间戳首选                 |
| `uint32`      | `uint32_t`    | 无符号 32 位                                 |
| `uint64`      | `uint64_t`    | 无符号 64 位                                 |
| `sint32`      | `int32_t`     | 负数较多场景推荐，zigzag 编码                |
| `sint64`      | `int64_t`     | 负数较多 64 位场景                           |
| `fixed32`     | `uint32_t`    | 固定 4 字节，数值很大且稳定时使用            |
| `fixed64`     | `uint64_t`    | 固定 8 字节                                  |
| `sfixed32`    | `int32_t`     | 固定 4 字节有符号                            |
| `sfixed64`    | `int64_t`     | 固定 8 字节有符号                            |
| `float`       | `float`       | 32 位单精度浮点                              |
| `double`      | `double`      | 64 位双精度浮点                              |
| `bool`        | `bool`        | 布尔类型                                     |
| `string`      | `std::string` | UTF-8 文本字符串                             |
| `bytes`       | `std::string` | 原始二进制，支持`\0`，不要当做普通字符串处理 |

##### import导入

``` protobuf
import "common/base.proto";
```

##### 消息（对应类）

- 数据成员：
  - Tag **1\~15 使用 1 字节编码；16 + 占用 2 字节**，高频字段尽量 1\~15
  - 一旦上线，Tag 永不允许修改、不允许复用！
  - 废弃字段：不要删除，使用 `reserved` 保留 tag 防止复用

  ``` protobuf
  message User {
  reserved 5,6;
  reserved "phone";
  // 类型 字段名 = 唯一编号;
  int64 id = 1;
  string name = 2;
  bool enable = 3;
  float score = 4;
  }
  ```
- 代码中调用：

``` protobuf
// 读字段
int64_t uid = user.uid();
std::string name = user.username();

// 设置字段
user.set_uid(10001);
user.set_username("jack");

// 判断字段是否存在（proto3特性：默认值不会序列化）
bool has_avatar = user.has_avatar_data();

// 清空字段
user.clear_avatar_data();
```

- 嵌套

``` protobuf
message Order {
  int64 order_id = 1;
  message Item {
    string goods = 1;
    int32 num = 2;
  }
  repeated Item items = 2;
}
```

代码中调用

``` cpp
demo::Order order;
demo::Order::Item* item = order.add_items();
item->set_goods("book");
```

##### 数组

``` protobuf
message UserList {
  repeated int64 uids = 1;
  repeated User users = 2;
}
```

- 代码中调用：

``` cpp
demo::UserList list;
// 添加元素
list.add_uids(1001);
list.add_uids(1002);

// 获取数量
int cnt = list.uids_size();
// 遍历
for(int i=0; i < list.uids_size(); i++){
    int64_t id = list.uids(i);
}

// 添加子消息
demo::User* p_user = list.add_users();
p_user->set_uid(2001);
```

##### 枚举

``` protobuf
enum UserStatus {
  USER_STATUS_UNKNOWN = 0;
  USER_STATUS_NORMAL = 1;
  USER_STATUS_FORBIDDEN = 2;
}

message User {
  UserStatus status = 1;
}
```

- 代码中调用：

``` cpp
user.set_status(demo::USER_STATUS_NORMAL);
demo::UserStatus s = user.status();
```

##### union联合体

``` protobuf
message Notify {
  oneof content {
    string text = 1;
    bytes image_bin = 2;
  }
}
```

- 代码中调用：

``` cpp
notify.set_text("hello");
if(notify.has_text()){}
notify.clear_text();
```

##### service

``` protobuf
message GetUserReq {
  int64 uid = 1;
}
message GetUserResp {
  User info = 1;
}

service UserService {
  // 一元调用 1req -> 1resp
  rpc GetUser(GetUserReq) returns(GetUserResp);
  // 服务端流
  rpc SubscribeLog(GetUserReq) returns(stream LogItem);
  // 客户端流
  rpc UploadData(stream DataChunk) returns(UploadResult);
  // 双向流
  rpc Chat(stream ChatMsg) returns(stream ChatMsg);
}
```

#### 常用命令

- `xxx.proto`会生成`xxx.pb.h`和`xxx.pb.cc`

``` bash
# 生成 .pb.h 与 .pb.cc
protoc --cpp_out=. user.proto
# 搭配 gRPC，额外使用 grpc_cpp_plugin
protoc --cpp_out=. --grpc_out=. --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` user.proto
```

#### 序列化与反序列化

``` cpp
#include "user.pb.h"
#include <fstream>
#include <string>

// 序列化 对象 -> 二进制字符串
std::string Serialize(const demo::User& user){
    std::string bin;
    user.SerializeToString(&bin);
    return bin;
}

// 反序列化 二进制 -> 对象
bool Deserialize(const std::string& bin, demo::User& out){
    return out.ParseFromString(bin);
}

int main(){
    demo::User user;
    user.set_uid(10001);
    user.set_username("alice");

    std::string data = Serialize(user);

    demo::User target;
    if(Deserialize(data, target)){
        // 解析成功
    }
    return 0;
}
```

#### Arena 内存优化

- 默认每条消息单独堆分配；高并发场景使用 Arena 批量管理内存

``` cpp
google::protobuf::Arena arena;
demo::User* user = google::protobuf::Arena::CreateMessage<demo::User>(&arena);
user->set_uid(1001);
// arena销毁时统一回收内存，无需手动delete
```

#### cmake集成

- 必须下载安装protoc编译器和libprotobuf库
- 强烈建议 protoc 编译器版本 ≈ libprotobuf 库版本
- protoc编译器：`sudo apt install protobuf-compiler`，可以用`protoc --version`查看版本
- libprotobuf 库：`sudo apt install libprotobuf-dev`，只有在c++代码运行时查询了

``` cpp
#include <iostream>
#include <google/protobuf/version.h>

int main() {
    std::cout << "libprotobuf version: "
              << GOOGLE_PROTOBUF_VERSION << std::endl;
    std::cout << "version string: "
              << GOOGLE_PROTOBUF_VERSION_STRING << std::endl;
    return 0;
}
```

- 使用FetchContent进行protoc包下载：

``` txt
include(FetchContent)
FetchContent_Declare(
  protobuf
  GIT_REPOSITORY https://github.com/protocolbuffers/protobuf.git
  GIT_TAG v3.21.12
)
# 拉取并编译
FetchContent_MakeAvailable(protobuf)
```

- `protobuf_generate_cpp`：在 `${CMAKE_CURRENT_BINARY_DIR}` 生成 `user.pb.h` / `user.pb.cc`，所以需要`target_include_directories(demo PRIVATE ${CMAKE_CURRENT_BINARY_DIR})`来引入生成的头文件

``` txt
# --------------------------
# 1. 查找Protobuf，cmake自带，给Protobuf_INCLUDE_DIRS、Protobuf_LIBRARIES、Protobuf_PROTOC_EXECUTABLE自动赋值
# --------------------------
find_package(Protobuf REQUIRED)

# 打印信息（调试用，可以删掉）
message(STATUS "Protobuf头文件位置: ${Protobuf_INCLUDE_DIRS}")
message(STATUS "Protobuf库文件位置: ${Protobuf_LIBRARIES}")
message(STATUS "protoc可执行文件位置: ${Protobuf_PROTOC_EXECUTABLE}")

# --------------------------
# 2. 自动调用 protoc 生成 pb.h / pb.cc
# --------------------------
set(PROTO_FILE user.proto)
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS ${PROTO_FILE})

# --------------------------
# 3. 构建可执行程序
# --------------------------
add_executable(demo
    main.cpp
    ${PROTO_SRCS}
    ${PROTO_HDRS}
)

# 头文件路径
target_include_directories(demo PRIVATE
    ${CMAKE_CURRENT_BINARY_DIR}
    ${Protobuf_INCLUDE_DIRS}
)

# 链接protobuf库
target_link_libraries(demo PRIVATE
    ${Protobuf_LIBRARIES}
)
```

## gRPC

- RPC：Remote Procedure Call，远程过程调用，**客户端像调用本地函数一样调用另一台服务器上的方法**，不用手动处理网络请求、序列化等内容（框架会代替执行）。
- gRPC默认基于HTTP/2，使用protobuf定义接口和完成序列化
- protobuf生成两类代码：
  - 消息定义：`xxx.pb.h / xxx.pb.cc`
  - 客户端存根服务端服务：`xxx.grpc.pb.h / xxx.grpc.pb.cc`，所以客户端和服务端接口是在同一个文件里的，使用时多了一点点无关符号，不影响
    \### 核心组件
- **==Protobuf== (.proto)**：接口描述文件，定义服务、方法、数据结构，二进制序列化，体积小解析快
- **Client（客户端）**：调用远程方法的一方
- **Server（服务端）**：提供远程方法实现
- **Stub（存根）**：客户端的代理类，看起来像本地函数，底层自动封装网络请求发给远端服务。
- **==HTTP/2==（信道）**：二进制帧传输、多路复用（频分时分复用之类的，一个连接可以多个请求）、头部压缩、支持双向通信和流传输
  \### 通信模式
- **Unary 一元调用**：客户端发 1 次请求 → 服务端返回 1 次响应（类似普通 HTTP 接口）

``` protobuf
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}
```

- **Server Streaming 服务端流**：客户端发 1 次请求 → 服务端连续返回多条数据流（例如日志推送、批量数据）

``` protobuf
rpc ListData(QueryReq) returns (stream DataResp);
```

- **Client Streaming 客户端流**：客户端连续发送多条请求 → 服务端返回 1 次响应（批量上传）

``` protobuf
rpc UploadLog(stream LogItem) returns (UploadResult);
```

- **Bidirectional Streaming 双向流**：双方持续互相发送消息（聊天、实时传感器数据）

``` protobuf
rpc Chat(stream ChatMsg) returns (stream ChatMsg);
```

### 全流程

1.  编写 `.proto` 文件：定义服务、方法、请求体、响应体
2.  通过 gRPC 工具链（protoc+grpc_cpp_plugin）**自动生成对应语言代码**

- `xxx.pb.h/xxx.pb.cc`：protobuf 消息序列化代码
- `xxx.grpc.pb.h/xxx.grpc.pb.cc`：gRPC Client/Service 骨架接口

3.  服务端：继承生成的 `Greeter::Service`，重写 rpc 函数实现业务逻辑
4.  启动 gRPC Server，注册服务，监听端口
5.  客户端：创建 Stub（存根），直接调用`stub->SayHello()`
6.  调用Stub方法，对象通过protobuf序列化为二进制，通过http2发送，protobuf反序列化解包，执行后返回
    \### cmake集成

- 编译依赖：
  - 工具
    - `protoc`：protobuf 编译器
    - `grpc_cpp_plugin`：给 protoc 使用，生成 grpc service 代码
  - 库
    - `libprotobuf`
    - `libgrpc++`、`libgrpc`、abseil 等依赖库
- gRPC 内部依赖 protobuf，必须由 gRPC 统一管理 protobuf，不要单独再 FetchContent protobuf，避免版本冲突

``` protobuf
cmake_minimum_required(VERSION 3.18)
project(grpc_demo LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ===================== FetchContent 拉取 gRPC（内置protobuf） =====================
include(FetchContent)

# gRPC编译开关，关闭测试、示例减少编译耗时
set(ABSL_BUILD_TESTS OFF CACHE BOOL "")
set(gRPC_BUILD_TESTS OFF CACHE BOOL "")
set(gRPC_BUILD_CSHARP_EXT OFF CACHE BOOL "")
set(gRPC_BUILD_EXAMPLES OFF CACHE BOOL "")
set(gRPC_BUILD_SHARED_LIBS OFF CACHE BOOL "") # 静态链接

FetchContent_Declare(
  gRPC
  GIT_REPOSITORY https://github.com/grpc/grpc.git
  GIT_TAG        v1.64.0
  GIT_SHALLOW    TRUE
)
FetchContent_MakeAvailable(gRPC)

# ===================== 编译proto，生成pb与grpc代码 =====================
set(PROTO_FILE "${CMAKE_SOURCE_DIR}/proto/greeter.proto")
grpc_generate_cpp(
  PROTO_SRCS
  PROTO_HDRS
  ${PROTO_FILE}
)

# ===================== 服务端目标 =====================
add_executable(server
    src/server.cpp
    ${PROTO_SRCS}
    ${PROTO_HDRS}
)
target_link_libraries(server PRIVATE gRPC::grpc++)
# grpc_generate_cpp会生成头文件放在 CMAKE_CURRENT_BINARY_DIR，需要引入
target_include_directories(server PRIVATE ${CMAKE_CURRENT_BINARY_DIR})

# ===================== 客户端目标 =====================
add_executable(client
    src/client.cpp
    ${PROTO_SRCS}
    ${PROTO_HDRS}
)
target_link_libraries(client PRIVATE gRPC::grpc++)
target_include_directories(client PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
```

### proto消息和服务注册

- **没有 required /optional**，proto3 所有字段默认隐式 optional：不存在字段会返回默认值，无法区分 "未赋值" 和 "赋值默认值"
- 服务名、rpc 名大小写敏感
- 具体使用参见[protobuf](#protobuf)

``` protobuf
// 指定proto版本，必须写在第一行
syntax = "proto3";

// 包名，防止命名冲突，对应C++命名空间
package demo;

// 生成C++代码的命名空间，可选
option csharp_namespace = "Demo";
option java_package = "com.demo";

// 消息结构体（相当于结构体/DTO）
message HelloRequest {
  // 字段编号 1~536870911，不能重复，删除的字段号保留不要复用
  string name = 1;
  int32 age = 2;
  bool is_ok = 3;
}

message HelloResponse {
  string msg = 1;
}

// 服务定义：rpc方法集合
service HelloService {
  // 四种调用模式
  rpc SayHello (HelloRequest) returns (HelloResponse);
}
```

### 服务端实现

#### 四种通信模式生成的service基类

``` cpp
namespace demo {

class DemoSvc::Service {
 public:
  virtual ~Service() = default;

  // -------- 1️⃣ 一元 Unary(Req)->Resp --------
  virtual grpc::Status Unary(
      grpc::ServerContext* context,
      const Req* request,
      Resp* response
  ) = 0;

  // -------- 2️⃣ 服务端流 ServerStreamRpc(Req)->stream Resp --------
  // 第二个参数：请求；第三个参数：ServerWriter，服务端用来多次写响应
  virtual grpc::Status ServerStreamRpc(
      grpc::ServerContext* context,
      const Req* request,
      grpc::ServerWriter<Resp>* writer
  ) = 0;

  // -------- 3️⃣ 客户端流 ClientStreamRpc(stream Req)->Resp --------
  // 第二个参数：ServerReader，服务端循环读取客户端发来的多个Req
  virtual grpc::Status ClientStreamRpc(
      grpc::ServerContext* context,
      grpc::ServerReader<Req>* reader,
      Resp* response
  ) = 0;

  // --------4️⃣双向流 BidiRpc(stream Req)->stream Resp --------
  // 参数：ServerReaderWriter，同时可读可写
  virtual grpc::Status BidiRpc(
      grpc::ServerContext* context,
      grpc::ServerReaderWriter<Resp, Req>* reader_writer
  ) = 0;

  // ...内部虚函数、调度相关，由gRPC框架调用，业务不用碰
};

}
```

#### ClientContext配置

##### 获取调用结果

``` cpp
// 读取metadata
auto& meta = ctx->client_metadata();
auto trace_it = meta.find("trace-id");
if(trace_it != meta.end()){
    std::string trace_id(trace_it->second.data(), trace_it->second.size());
}

// 获取客户端对端地址
std::string peer = ctx->peer();
```

##### response metadata

- 响应头：

``` cpp
// 1. Initial metadata：越早调用越好，流开启之后不能再改
ctx->AddInitialMetadata("svc-name", "game‑svc");
```

- 响应尾：

``` cpp
// 2. Trailing metadata（绝大多数业务用这个）rpc结束返回
ctx->AddTrailingMetadata("svc-version", "v2.3.0");
ctx->AddTrailingMetadata("x-cost-ms", "42"); // 服务端耗时
ctx->AddTrailingMetadata("x-trace-id", "550e8400-e29b-41d4-a716-446655440000");
```

- 业务内容
  - x‑trace‑id：回传 traceId，客户端打印日志
  - svc‑version：服务端版本号
  - x‑cost‑ms：服务端处理耗时毫秒
  - x‑warn‑msg：警告信息（业务成功但有告警）
    \##### 其他
    \`\`\` cpp
    // 主动取消服务端侧RPC
    ctx-\>TryCancel();

// 判断客户端是否已经取消
bool is_cancel = ctx-\>IsCancelled();

    #### 实现service方法
    ``` cpp
    #include "demo.grpc.pb.h"
    #include <grpcpp/grpcpp.h>
    #include <iostream>
    #include <string>

    using grpc::Server;
    using grpc::ServerBuilder;
    using grpc::ServerContext;
    using grpc::ServerReader;
    using grpc::ServerReaderWriter;
    using grpc::ServerWriter;
    using grpc::Status;

    class EchoSvcImpl final : public demo::EchoSvc::Service {
    public:
        // ===================== 1. 一元 RPC =====================
        Status UnaryEcho(ServerContext* ctx,
                         const demo::EchoReq* req,
                         demo::EchoResp* resp) override
        {
            std::cout << "[UnaryEcho] recv: " << req->content() << std::endl;

            // 服务端返回trailing metadata
            ctx->AddTrailingMetadata("mode", "unary");

            resp->set_result("unary reply -> " + req->content());
            return Status::OK;
        }

        // ===================== 2. 服务端流 RPC =====================
        // 参数：ctx, 请求, ServerWriter 用于多次写回给客户端
        Status ServerStreamEcho(ServerContext* ctx,
                                const demo::EchoReq* req,
                                ServerWriter<demo::EchoResp>* writer) override
        {
            std::cout << "[ServerStreamEcho] recv req:" << req->content() << std::endl;

            // ✅ InitialMetadata 必须在第一次Write之前设置！
            ctx->AddInitialMetadata("stream-type", "server-stream");

            // 循环写3条流式消息给客户端
            for(int i = 0; i < 3; ++i)
            {
                // 判断客户端是否已经断开/取消
                if(ctx->IsCancelled())
                {
                    std::cout << "[ServerStreamEcho] client cancelled" << std::endl;
                    return Status::CANCELLED;
                }

                demo::EchoResp resp;
                resp.set_result("stream item " + std::to_string(i) + " -> " + req->content());
                writer->Write(resp);
            }

            // rpc结束前任意位置添加trailing metadata
            ctx->AddTrailingMetadata("total-items", "3");
            return Status::OK;
        }

        // ===================== 3. 客户端流 RPC =====================
        // 参数：ctx, ServerReader循环读取客户端多条请求, resp输出最终响应
        Status ClientStreamEcho(ServerContext* ctx,
                                ServerReader<demo::EchoReq>* reader,
                                demo::EchoResp* resp) override
        {
            std::cout << "[ClientStreamEcho] start" << std::endl;
            std::string accumulate;

            demo::EchoReq req;
            // 循环读取客户端发来的多条消息，返回false代表流结束
            while(reader->Read(&req))
            {
                std::cout << "[ClientStreamEcho] recv:" << req.content() << std::endl;
                accumulate += req.content() + " | ";

                if(ctx->IsCancelled()){
                    return Status::CANCELLED;
                }
            }

            // 全部读完之后，填充最终响应返回客户端
            resp->set_result("client stream all: " + accumulate);
            ctx->AddTrailingMetadata("mode", "client-stream");
            return Status::OK;
        }

        // ===================== 4. 双向流 RPC =====================
        // ServerReaderWriter：既可Read读客户端，也可Write写回客户端
        Status BidiStreamEcho(ServerContext* ctx,
                               ServerReaderWriter<demo::EchoResp, demo::EchoReq>* rw) override
        {
            std::cout << "[BidiStreamEcho] start" << std::endl;

            // InitialMetadata 必须在第一次Write前调用
            ctx->AddInitialMetadata("stream-type", "bidi-stream");

            demo::EchoReq req;
            // 一边读客户端消息，一边回写
            while(rw->Read(&req))
            {
                std::cout << "[BidiStreamEcho] recv:" << req.content() << std::endl;

                demo::EchoResp resp;
                resp.set_result("bidi reply: " + req.content());
                rw->Write(resp); // 立刻回写给客户端
            }

            ctx->AddTrailingMetadata("mode", "bidi-stream");
            return Status::OK;
        }
    };

#### 服务器构建

``` cpp
int main(int argc, char** argv)
{
    // 1. 实例化我们自己写的服务实现对象
    HelloSvcImpl service;

    // 2. ServerBuilder：服务构建器，用来配置端口、注册服务、配置证书、线程池
    ServerBuilder builder;

    // 绑定监听地址，InsecureServerCredentials 无加密，生产环境要用TLS证书
    // 格式 ip:port，0.0.0.0代表监听本机所有网卡
    builder.AddListeningPort("0.0.0.0:50051", grpc::InsecureServerCredentials());

    // 将服务实现注册到builder，内部保存service指针，对象生命周期要长于server
    builder.RegisterService(&service);

    // 构建并启动服务，BuildAndStart之后服务就开始接收连接
    std::unique_ptr<Server> server = builder.BuildAndStart();
    std::cout << "gRPC服务启动，监听 0.0.0.0:50051" << std::endl;

    // Wait()：阻塞主线程，等待server退出；会内部处理rpc请求，内部有线程池处理请求
    // gRPC内部自带线程池，每一个rpc请求跑在内部线程，不要在rpc实现里阻塞太久
    server->Wait();

    // 退出逻辑（收到信号触发关闭后走到这里）
    std::cout << "服务已关闭" << std::endl;
    return 0;
}
```

### 客户端实现

#### 四种通信模式生成的client存根stub类

- 同步接口

``` cpp
namespace demo {

class DemoSvc::Stub {
public:
  // -----1️⃣一元 Unary -----
  grpc::Status Unary(
      grpc::ClientContext* context,
      const Req& request,
      Resp* response
  );

  // -----2️⃣服务端流 ServerStreamRpc -----
  // 返回 ClientReader<Resp>：客户端用来读取服务端多次推送
  std::unique_ptr<grpc::ClientReader<Resp>> ServerStreamRpc(
      grpc::ClientContext* context,
      const Req& request
  );

  // -----3️⃣客户端流 ClientStreamRpc -----
  // 返回 ClientWriter<Req>：客户端多次写Req；Finish拿到最终Resp
  std::unique_ptr<grpc::ClientWriter<Req>> ClientStreamRpc(
      grpc::ClientContext* context,
      Resp* response
  );

  // -----4️⃣双向流 BidiRpc -----
  // 返回 ClientReaderWriter<Resp,Req>，可读可写
  std::unique_ptr<grpc::ClientReaderWriter<Resp, Req>> BidiRpc(
      grpc::ClientContext* context
  );

  // ...异步接口、构造析构省略
};

}
```

- 异步接口
  暂时不讨论，实际使用较少，有CompletionQueue CQ和Callback‑Reactor两种。
  \#### ClientContext配置
  \##### 获取调用结果

``` cpp
// 获取服务端返回的trailing metadata（最常用）
std::multimap<grpc::string_ref, grpc::string_ref> meta = ctx.GetTrailingMetadata();

// 获取服务端返回的initial metadata（流RPC拿到第一个响应后读取）
std::multimap<grpc::string_ref, grpc::string_ref> init_meta = ctx.GetInitialMetadata();

// 获取本次RPC实际使用的peer地址（服务端地址）
std::string peer_addr = ctx.peer();

// 获取RPC状态详情，调试用
grpc::Status status = ctx.status();
```

##### 超时时间

``` cpp
// 设置RPC绝对截止时间，超过这个时间直接超时失败
ctx.set_deadline(std::chrono::system_clock::now() + std::chrono::milliseconds(3000));

// 查询当前设置的deadline
std::chrono::system_clock::time_point ddl = ctx.deadline();
```

##### request metadata

- 通过 `ClientContext::AddMetadata(key,value)` 发送
- authorization：token 鉴权，最常用，`Bearer xxxxxjwt_token`
- x‑request‑id：请求链路追踪 traceId，全链路打日志，`trace‑uuid‑123456`
- x‑client‑version：客户端版本号，`v1.2.0`
- x‑client‑ip：客户端透传真实 ip（网关场景），`192.168.1.100`
- x‑app‑id：客户端应用标识，`game‑backend`
- x‑tenant‑id：多租户场景租户 ID，`10086`
- my‑data‑bin：自定义二进制字符，`std::bytes bin_data = "\x00\x01\x02";`
  \##### 其他
- `COMPRESS_NONE` 不压缩
- `COMPRESS_GZIP` gzip 压缩
- `COMPRESS_DEFLATE` deflate

``` cpp
// 设置是否开启压缩，kNoCompression / kEnableCompression
ctx.set_compression_algorithm(grpc::COMPRESS_GZIP);

// 主动取消正在进行的rpc，服务端会收到取消事件
ctx.TryCancel();

// 设置认证相关的凭据，TLS场景使用，一般构造channel时指定即可，很少单独设置
ctx.set_credentials(credentials);

// 设置RPC调用的日志标签，内部日志打印会带上这个字符串，便于排查问题
ctx.set_log_tag("player‑rpc‑10001");

// 阻塞等待channel就绪，不建议放在业务RPC里
ctx.WaitForReady();
```

#### 客户端构建

- `channel`建议复用，`clientcontext`严禁复用
- 同步通信

``` cpp
#include "demo.grpc.pb.h"
#include <grpcpp/grpcpp.h>
#include <iostream>

using grpc::Channel;
using grpc::ClientContext;
using grpc::ClientReader;
using grpc::ClientReaderWriter;
using grpc::ClientWriter;
using grpc::Status;

int main()
{
    // 创建channel，多个stub可复用同一个channel
    std::shared_ptr<Channel> channel = grpc::CreateChannel(
        "127.0.0.1:50051",
        grpc::InsecureChannelCredentials()
    );
    std::unique_ptr<demo::EchoSvc::Stub> stub = demo::EchoSvc::NewStub(channel);


    // ========== 1. 一元调用 UnaryEcho ==========
    {
        std::cout << "\n===== 1.UnaryEcho =====" << std::endl;
        ClientContext ctx;
        ctx.AddMetadata("x-request-id", "unary-001");
        ctx.set_deadline(std::chrono::system_clock::now() + std::chrono::seconds(3));

        demo::EchoReq req;
        demo::EchoResp resp;
        req.set_content("hello unary");
        
        // 同步阻塞调用，这里线程卡住，直到收到响应/超时
        Status st = stub->UnaryEcho(&ctx, req, &resp);
        if(st.ok())
        {
            std::cout << "resp:" << resp.result() << std::endl;
            auto trail = ctx.GetTrailingMetadata();
        }
        else
        {
            std::cout << "fail:" << st.error_message() << std::endl;
        }
    }


    // ========== 2.服务端流 ServerStreamEcho ==========
    {
        std::cout << "\n===== 2.ServerStreamEcho =====" << std::endl;
        ClientContext ctx;
        ctx.AddMetadata("x-request-id", "server-stream-001");
        ctx.set_deadline(std::chrono::system_clock::now() + std::chrono::seconds(5));

        demo::EchoReq req;
        req.set_content("hello server stream");

        // ClientReader<T>：读取服务端多条返回
        std::unique_ptr<ClientReader<demo::EchoResp>> reader(stub->ServerStreamEcho(&ctx, req));

        demo::EchoResp resp;
        //循环读取服务端推送，Read返回false代表流结束
        while(reader->Read(&resp))
        {
            std::cout << "stream item: " << resp.result() << std::endl;
        }

        // 获取最终状态，必须调用Finish()
        Status st = reader->Finish();
        if(!st.ok())
        {
            std::cout << "stream finish fail:" << st.error_message() << std::endl;
        }
        // 可读取initial / trailing metadata
        auto init_meta = ctx.GetServerInitialMetadata();
        auto trail_meta = ctx.GetTrailingMetadata();
    }


    // ==========3.客户端流 ClientStreamEcho ==========
    {
        std::cout << "\n===== 3.ClientStreamEcho =====" << std::endl;
        ClientContext ctx;
        ctx.AddMetadata("x-request-id", "client-stream-001");
        ctx.set_deadline(std::chrono::system_clock::now() + std::chrono::seconds(5));

        demo::EchoResp resp;
        // ClientWriter<T>：客户端多次写消息
        std::unique_ptr<ClientWriter<demo::EchoReq>> writer(stub->ClientStreamEcho(&ctx, &resp));

        // 连续发送3条消息
        for(int i=0;i<3;i++)
        {
            demo::EchoReq req;
            req.set_content("client send msg " + std::to_string(i));
            writer->Write(req);
        }

        // Finish：告知服务端发送完毕，等待接收最终响应
        Status st = writer->Finish();
        if(st.ok())
        {
            std::cout << "final resp:" << resp.result() << std::endl;
        }
        else
        {
            std::cout << "finish fail:" << st.error_message() << std::endl;
        }
    }


    // ==========4.双向流 BidiStreamEcho ==========
    {
        std::cout << "\n===== 4.BidiStreamEcho =====" << std::endl;
        ClientContext ctx;
        ctx.AddMetadata("x-request-id", "bidi-001");
        ctx.set_deadline(std::chrono::system_clock::now() + std::chrono::seconds(5));

        // ClientReaderWriter：同时可读、可写
        std::unique_ptr<ClientReaderWriter<demo::EchoResp, demo::EchoReq>> rw(stub->BidiStreamEcho(&ctx));

        // 先发送2条消息
        for(int i=0;i<2;i++)
        {
            demo::EchoReq req;
            req.set_content("bidi send " + std::to_string(i));
            rw->Write(req);
        }

        // 循环读取服务端回包
        demo::EchoResp resp;
        while(rw->Read(&resp))
        {
            std::cout << "bidi recv:" << resp.result() << std::endl;
        }

        // 流结束，获取状态
        Status st = rw->Finish();
        if(!st.ok())
        {
            std::cout << "bidi finish fail:" << st.error_message() << std::endl;
        }
    }

    return 0;
}
```

### 高阶特性

#### ChannelArguments配置

``` cpp
    grpc::ChannelArguments args;

    grpc::CreateCustomChannel("static:///127.0.0.1:50051,127.0.0.1:50052",
    grpc::InsecureChannelCredentials(), args);
```

#### keepalive（客户端+服务端）

``` cpp
    // 客户端：
    grpc::ChannelArguments args;
    args.SetInt(GRPC_ARG_KEEPALIVE_TIME_MS, 30000);
    args.SetInt(GRPC_ARG_KEEPALIVE_TIMEOUT_MS, 10000);
    args.SetInt(GRPC_ARG_KEEPALIVE_PERMIT_WITHOUT_CALLS, 1);
    args.SetInt(GRPC_ARG_HTTP2_MAX_PINGS_WITHOUT_DATA, 0);
    
    // 服务端：
    grpc::ServerBuilder builder;
    builder.AddChannelArgument(GRPC_ARG_KEEPALIVE_TIME_MS,30000);
    builder.AddChannelArgument(GRPC_ARG_KEEPALIVE_TIMEOUT_MS,10000);
    builder.AddChannelArgument(GRPC_ARG_HTTP2_MAX_PINGS_WITHOUT_DATA,0);
```

#### 负载均衡+resolver（客户端）

- SubChannel 子连接：Channel内部可以包含多个
- Resolver：根据服务名返回后端节点地址列表`[ip:port]`
  - dns resolver（默认）：创建channel时URL`"dns:///lobby-svc:50051"`
  - static resolver：创建channel时URL`"static:///127.0.0.1:6001,127.0.0.1:6002"`
  - custom 自定义 Resolver：创建channel时URL`"nacos:///lobby-svc:50051"`，与各种注册中心（nacos、etcd、consul）对接
- 负载均衡器：根据resolver返回的节点列表，选择其中一个channel发送消息
  - pick_first（默认）：一旦连接建立成功，**后续所有 RPC 全部往这一条连接发**，不会分散流量，单一子连接
  - round_robin：轮询节点，流量均匀打散，多个子连接
  - weighted_round_robin：加权轮询
  - priority：优先级策略，主备集群
- 游戏后端的常见方案是：业务层订阅nacos注册中心+static resolver+round_robin负载均衡

``` cpp
#include <grpcpp/grpcpp.h>
#include "Nacos.h"
#include <vector>
#include <mutex>
#include <memory>

// 全局保存channel，多线程访问加锁
std::mutex g_ch_mtx;
std::shared_ptr<grpc::Channel> g_lobby_channel;

// 根据实例列表生成static resolver target字符串
std::string BuildStaticTarget(const std::vector<Instance>& instances)
{
    std::string target = "static:///";
    for(size_t i = 0; i < instances.size(); i++)
    {
        const auto& inst = instances[i];
        if(!inst.healthy) continue;
        if(i>0) target += ",";
        target += inst.ip + ":" + std::to_string(inst.port);
    }
    return target;
}

// 创建channel，统一配置keepalive + round_robin
std::shared_ptr<grpc::Channel> CreateChannelByTarget(const std::string& target)
{
    grpc::ChannelArguments args;
    args.SetInt(GRPC_ARG_KEEPALIVE_TIME_MS,30000);
    args.SetInt(GRPC_ARG_KEEPALIVE_TIMEOUT_MS,10000);
    args.SetInt(GRPC_ARG_KEEPALIVE_PERMIT_WITHOUT_CALLS,1);
    args.SetInt(GRPC_ARG_HTTP2_MAX_PINGS_WITHOUT_DATA,0);

    args.SetString(GRPC_ARG_SERVICE_CONFIG, R"(
    {
        "loadBalancingConfig": [{"round_robin":{}}]
    })");
    return grpc::CreateCustomChannel(target, grpc::InsecureChannelCredentials(), args);
}

// nacos监听，实例变更回调
class NacosListener : public EventListener
{
public:
    void OnEvent(const ServiceInfo& serviceInfo) override
    {
        std::lock_guard<std::mutex> lock(g_ch_mtx);
        auto new_target = BuildStaticTarget(serviceInfo.hosts);
        if(new_target == "static:///")
        {
            // 无可用实例，channel置空，RPC返回UNAVAILABLE
            g_lobby_channel = nullptr;
            return;
        }
        // 重建channel，shared_ptr接管旧channel资源
        g_lobby_channel = CreateChannelByTarget(new_target);
    }
};

int main()
{
    // 1 nacos初始化
    Properties props;
    props[PropertyKeyConst::SERVER_ADDR] = "127.0.0.1:8848";
    auto* fac = NacosFactoryFactory::getNacosFactory(props);
    auto* naming = fac->CreateNamingService();

    // 2 订阅lobby‑service
    auto watcher = std::make_shared<NacosListener>();
    naming->subscribe("lobby‑service", "DEFAULT", {}, watcher.get());

    // 业务拿channel创建stub
    std::shared_ptr<grpc::Channel> ch;
    {
        std::lock_guard<std::mutex> lock(g_ch_mtx);
        ch = g_lobby_channel;
    }
    if(!ch) return -1;
    auto stub = Greeter::NewStub(ch);
    // 调用rpc……
    return 0;
}
```

#### 日志拦截器（服务端）

- 拦截器-设计模式装饰器模式

``` cpp
class MyServerInterceptor : public grpc::ServerInterceptor
{
public:
    void InterceptUnaryRpc(grpc::ServerInterceptorCallInfo info,
                           grpc::ServerUnaryInterceptorHandler* handler) override
    {
        // ========= RPC进入业务之前（前置） =========
        auto start = std::chrono::steady_clock::now();

        // 可以拿metadata，做鉴权
        auto& meta = info.context->client_metadata();
        auto it = meta.find("trace-id");

        // ✅放行，执行业务Reactor函数；如果不想放行，直接返回错误，不要调用Proceed()
        handler->Proceed();

        // ========= RPC业务执行完成之后（后置） =========
        auto cost = std::chrono::duration_cast<std::chrono::milliseconds>(
                std::chrono::steady_clock::now() - start);

        // info.status 拿到本次RPC最终状态
        bool ok = info.status->ok();
        // 打印日志、埋监控指标：rpc名称、耗时、成功失败、traceId
    }
};

grpc::ServerBuilder builder; builder.AddInterceptor(std::make_shared<MyServerInterceptor>());
```

#### RPC主动取消（客户端+服务端）

- 客户端：

``` cpp
// context必须是堆对象，因为异步回调还会访问它
auto ctx = std::make_unique<grpc::CallbackClientContext>();
ctx->set_deadline(std::chrono::system_clock::now() + std::chrono::seconds(3));

HelloReq req;
// 注册好接收到服务端rpc取消后客户端的行为
stub->AsyncSayHello(ctx.get(), req,
[](grpc::Status st, HelloResp resp)
{
    if(st.error_code() == grpc::StatusCode::CANCELLED)
    {
    }
});

// 客户端主动发起取消本次RPC
ctx->TryCancel();
```

- 服务端：

``` cpp
grpc::ServerUnaryReactor* SayHello(grpc::CallbackServerContext* ctx,
                                   const HelloReq* req, HelloResp* resp) override
{
    // 如果客户端已经取消，直接返回取消状态
    if(ctx->IsCancelled())
    {
        auto* reactor = ctx->DefaultReactor();
        reactor->Finish(grpc::Status::Cancelled("client cancel"));
        return reactor;
    }

    // 正常业务
    resp->set_msg("ok");
    auto* reactor = ctx->DefaultReactor();
    reactor->Finish(grpc::Status::OK);
    return reactor;
}
```

- 双向流：

``` cpp
class TunnelReactor : public grpc::ServerBidiReactor<TunnelMsg, TunnelMsg>
{
    void OnCancel() override
    {
        // 客户端主动调用TryCancel / 网络断开，会进到这里
        // 在这里清理会话、释放玩家资源
    }

    void OnFinish(grpc::Status s) override
    {
        // 流最终结束，一定执行资源回收
    }
};
```

#### callback双向流（客户端+服务端）

- proto

``` protobuf
syntax = "proto3";
package tunnel;

message TunnelMsg {
  uint32 player_id = 1;
  bytes payload = 2;
}

service TunnelService {
  // 双向流：客户端、服务端互相异步收发消息
  rpc MsgTunnel(stream TunnelMsg) returns (stream TunnelMsg);
}
```

- 客户端：

``` cpp
class ClientTunnelReactor : public ClientBidiReactor<TunnelMsg, TunnelMsg>
{
public:
    ClientTunnelReactor()
    {
        StartRead(&recv_msg_);
    }

    // 收到服务端推送消息
    void OnReadDone(bool ok) override
    {
        if(!ok) return;
        // 业务处理服务端返回消息
        StartRead(&recv_msg_);
    }

    void OnWriteDone(bool ok) override {}

    // RPC完成/断开
    void OnFinish(Status s) override
    {
        // 流结束
    }

    // 对外接口：业务发送消息到服务端
    void Send(uint32 pid, const std::string& data)
    {
        TunnelMsg msg;
        msg.set_player_id(pid);
        msg.set_payload(data);
        StartWrite(&msg);
    }

private:
    TunnelMsg recv_msg_;
};

auto stub = TunnelService::NewStub(ch);
auto* reactor = new ClientTunnelReactor();
CallbackClientContext ctx;
stub->AsyncMsgTunnel(&ctx, reactor);

// 业务发送消息
reactor->Send(10001,"hello game");

// 需要关闭流: reactor->StartWritesDone();
```

- 服务端：

``` cpp
#include <grpcpp/grpcpp.h>
#include "tunnel.grpc.pb.h"
using namespace grpc;
using namespace tunnel;

// 双向流Reactor，gRPC管理生命周期，不要手动delete
class BidiTunnelReactor : public ServerBidiReactor<TunnelMsg, TunnelMsg>
{
public:
    explicit BidiTunnelReactor(CallbackServerContext* ctx) : ctx_(ctx)
    {
        // 注册读事件，等待客户端发来消息
        StartRead(&recv_msg_);
    }

    // 收到客户端一条消息
    void OnReadDone(bool ok) override
    {
        if (!ok)
        {
            // 读失败，连接已经关闭，直接返回
            return;
        }

        // ========== 业务处理 recv_msg_ ==========
        uint32 pid = recv_msg_.player_id();

        // 回包示例：原样回写消息
        TunnelMsg resp;
        resp.set_player_id(pid);
        resp.set_payload(recv_msg_.payload());
        StartWrite(&resp); // 异步发送给客户端

        // 继续监听下一条客户端消息
        StartRead(&recv_msg_);
    }

    // Write完成回调
    void OnWriteDone(bool ok) override
    {
        if (!ok) return;
    }

    // 客户端主动TryCancel()触发
    void OnCancel() override
    {
        // 玩家会话在这里做标记断开，释放业务资源
    }

    // 流最终结束，必定执行，做资源清理
    void OnFinish(Status s) override
    {
        // s.error_code 可以拿到关闭原因
    }

private:
    CallbackServerContext* ctx_;
    TunnelMsg recv_msg_;
};

// Service实现
class TunnelServiceImpl : public TunnelService::CallbackService
{
public:
    ServerBidiReactor<TunnelMsg, TunnelMsg>* MsgTunnel(CallbackServerContext* ctx) override
    {
        // gRPC接管reactor生命周期，禁止delete
        return new BidiTunnelReactor(ctx);
    }
};

int RunServer()
{
    TunnelServiceImpl service;
    ServerBuilder builder;

    // keepalive参数
    builder.AddChannelArgument(GRPC_ARG_KEEPALIVE_TIME_MS,30000);
    builder.AddChannelArgument(GRPC_ARG_KEEPALIVE_TIMEOUT_MS,10000);
    builder.AddChannelArgument(GRPC_ARG_HTTP2_MAX_PINGS_WITHOUT_DATA,0);

    builder.AddListeningPort("0.0.0.0:60001", InsecureServerCredentials());
    builder.RegisterService(&service);
    auto server = builder.BuildAndStart();
    server->Wait();
    return 0;
}
```

## pkg-config

- **Linux 下库元信息工具**，不是编译器、不是链接器；作用：输出库的编译 / 链接参数，解决「头文件在哪、库文件在哪、要链接哪些库」的麻烦，不用手写一堆 `-I -L -l`
- 原理是底层读取`.pc` 文件（库的描述文件）
- 库检测

``` bash
# 查看库是否安装、版本
pkg-config --modversion libcurl

# 列出所有能找到的pc库
pkg-config --list-all

# 检查库是否存在，shell脚本可判断 $? 返回值
pkg-config --exists libcurl

# 输出编译参数(CFLAGS：头文件路径)
pkg-config --cflags libcurl

# 输出链接参数(LDFLAGS：-L -l)
pkg-config --libs libcurl

# 输出依赖
pkg-config --libs --static libcurl   # 静态链接输出完整依赖链

# 合并输出 cflags + libs
pkg-config --cflags --libs libcurl
```

- 命令使用

``` bash
g++ main.cpp -o main `pkg-config --cflags --libs libcurl`
```

- cmake集成（优先find_package）

``` cmake
find_package(PkgConfig REQUIRED)
// pkg_check_modules(CURL REQUIRED libcurl)

target_include_directories(app PRIVATE ${CURL_INCLUDE_DIRS})
target_link_directories(app PRIVATE ${CURL_LIBRARY_DIRS})
target_link_libraries(app PRIVATE ${CURL_LIBRARIES})
```

- `PKG_CONFIG_PATH` 设置（找不到库时）
  - 文件位置：`/usr/lib/pkgconfig`、`/usr/lib64/pkgconfig`
  - .pc文件路径必须配置到这个文件里才找的到

  ``` bash
  export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
  ```

  ## Drogon
- **现代 C++17/20 高性能全异步 HTTP Web 框架**，基于 Trantor 事件库，对标 Go‑Gin，追求低延迟、高 QPS
- 功能：C++ 写 REST API、微服务、WebSocket 服务
- IO 模型：Linux `epoll`，Reactor 事件驱动，全异步非阻塞
- 编程范式：回调式异步；**C++20 协程支持**
- 内置：http server + http client、websocket server/client、路由、过滤器、ORM、Redis 客户端、Session/Cookie、gzip/brotli 压缩
- 能力：HTTP、WebSocket、数据库、Redis 异步客户端、简单编译期反射、过滤器链
