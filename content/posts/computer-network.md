---
title: "计算机基础系列-计算机网络"
slug: "computer-network"
date: 2026-08-25T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["计算机网络", "c++", "网络编程"]
categories: ["技术笔记"]
summary: "计算机网络基础概念，包括从家里到大型互联网全链路，网络五层模型、每层协议栈、重要知识点等。"
toc: true
comments: true
description: "计算机网络"
---

# 计算机网络
- 计算机网络=计算机技术+通信技术=硬件+软件+协议
- 通信链路：铜缆、光纤、无线电、卫星
- 拓扑：总线型、星型、环形、树形、网状
- 协议：网络中公认的数据交换的规则
- 泛洪：向除入端以外所有端口转发
- as：同一个机构统一管理的一组路由器
- mss = mtu - 最小ip头（20） - 最小tcp头（20）
- tcp-数据段，udp-数据报，ip-数据包，ethernet-帧
- 常用指标：
  - `bit/s`可以简写成`bps`
  - 带宽：链路物理层面，单位时间可传输的最大比特数，是链路**理论上限**，`100 mb/s`
  - 吞吐量：**成功**通过某个网络链路 / 端到端传输的有效数据量，`6 bit/s`
  - 时延：每个ip数据包传送所需的时间（发送 + 传播 + 处理 + 排队），进一步计算往返时延rtt，`x ms`
  - 丢包率：ip数据包丢包数/已发分组总数，`0.1%`
  - 每秒处理请求数：QPS
    \# 网络四层模型
- 通信子网（物理层、数据链路层、网络层、传输层）+资源子网（会话层、表示层、应用层）
- 五层模型的差别（数据链路层拆分出物理层）

| 层级 | 核心功能 | 典型协议 / 技术 | PDU (数据单元) | 关键设备 |
|----|----|----|----|----|
| **应用层** | 网络服务；业务数据 | HTTP、HTTPS、FTP、DNS、SMTP、SSH | 报文 | 网关、服务器 |
| **传输层** | 端到端通信、端口寻址、流量控制、差错恢复 | TCP、UDP | 数据段 (TCP) / 数据报 (UDP) | 防火墙 |
| **网络层** | 逻辑寻址、路由选择、数据分片 | IP、ICMP、ARP、OSPF | IP 数据包 | 路由器 |
| **数据链路层** | 帧封装、物理寻址、差错检测、介质访问控制 | MAC、Ethernet‑II、802.1Q(VLAN)、CRC32、CSMA/CD | 帧 | 交换机、网卡 MAC |
| **物理层** | 比特流传输、电平、信号、硬件接口 | 网线、光纤、PHY、自动协商 | 比特 (bit) | Hub 集线器、网卡 PHY |

## 应用层

- 7层模型中拆分成会话层、表示层、应用层
- 协议：http、ping、telnet、ospf、dns、ftp、NNTP、Archie、WAIS、Gopher
- 网络应用架构：c/s架构、b/s架构、p2p架构
  \### ==socket api==
- 操作系统提供的用于进程通信的接口，每一个进程都有一个套接字（socket），工作在网络和传输层，tcp/udp通用
- 操作系统维护一个套接字描述符表（socket fd table），存储的是指向套接字数据结构的指针
- tcp通信流程
  ![](../computer-network/ba44d46eadd4a3fabe2ecae8d7cfa4d1b918d6f4.png)
- udp通信流程
  ![](../computer-network/ad95167fc092b20300fe9e142a9e4ef4396e3741.png)
  \#### 核心函数
- 创建socket
  - `domain`：地址族
    - `AF_INET`：IPv4
    - `AF_INET6`：IPv6
  - `type`：套接字类型
    - `SOCK_STREAM`：TCP，流式，可靠连接
    - `SOCK_DGRAM`：UDP，数据报，无连接
    - `SOCK_RAW`：原始套接字
  - `protocol`：一般填 `0`，自动匹配协议
  - 返回：**socket fd**，-1 失败

  ``` cpp
  int socket(int domain, int type, int protocol);
  ```
- 套接字选项
  - level：选项层级
    - **SOL_SOCKET 套接字层（常用）**
    - IPPROTO_TCP TCP 层
    - IPPROTO_IP IP 层
  - optname：选项宏
  - optval：变量地址

  ``` cpp
  // 设置
  int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
  // 获取
  int getsockopt(int sockfd, int level, int optname, void *optval, socklen_t *optlen);
  ```
- 绑定ip端口（服务器）
  - addr：本机信息，包括协议、ip、端口，ip可以设置为INADDR_ANY表示本机所有ip
    \`\`\` cpp
    int bind(int sockfd, const struct sockaddr \*addr, socklen_t addrlen);

struct sockaddr_in {
sa_family_t sin_family; // AF_INET
in_port_t sin_port; // htons(端口)
struct in_addr sin_addr; // inet_addr("127.0.0.1")
};

    - 监听（服务器，仅TCP）
        - backlog：允许连接数，未完成连接队列 + 已完成连接队列最大长度
    ``` cpp
    int listen(int sockfd, int backlog);

- 接受连接（服务器，仅TCP）
  - 阻塞等待
  - addr ：对端地址，可以传 `NULL` 不获取
  - 返回**新的 conn_fd**，专门用于和该客户端通信；listen 的 fd 继续监听

  ``` cpp
  int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
  ```
- 发起连接（客户端）
  - addr：对端地址
  - TCP：三次握手建立连接，阻塞
  - UDP：记录对端地址，不发送信息，配合 send/recv 替代 sendto/recvfrom

  ``` cpp
  int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
  ```
- 关闭套接字（）
  - `close(fd)`：引用计数减一，计数为 0 才真正关闭，close 只是应用层释放 fd，内核还会处理 TIME_WAIT
  - `shutdown`：精细控制半关闭
    - `SHUT_RD`：关闭读
    - `SHUT_WR`：关闭写，发送 FIN，还可以继续读
    - `SHUT_RDWR`：读写全部关闭

    ``` cpp
    int close(int fd);
    int shutdown(int fd, int how);
    ```

    #### 地址转换
- 字符串转ip

``` cpp
inet_addr("127.0.0.1")          // 旧，有bug
inet_pton(AF_INET, "127.0.0.1", &sin_addr.s_addr);
inet_ntop(AF_INET, &sin_addr.s_addr, buf, sizeof(buf));
```

- 端口转换：
  - 大端序：，小端序：
  - 网络端序可能跟本地计算机上端序不同

  ``` cpp
  htons()// 本地字节顺序→网络字节顺序(16bits)
  ntohs()// 网络字节顺序→本地字节顺序(16bits)
  htonl()// 本地字节顺序→网络字节顺序(32bits)
  ntohl()// 网络字节顺序→本地字节顺序(32bits)
  ```

  #### 收发数据
- 接受的fd是conn_fd
- tcp
  - 返回 0 代表对端关闭连接；-1 出错
    \`\`\` cpp
    ssize_t read(int fd, void *buf, size_t count);
    ssize_t write(int fd, const void *buf, size_t count);

ssize_t recv(int sockfd, void *buf, size_t len, int flags);
ssize_t send(int sockfd, const void *buf, size_t len, int flags);

    - udp
        - 自带对端地址，每次收发都携带地址信息
        - flag常用
        - 普通读写设置为0即可
            - `MSG_NOSIGNAL`：send常用，send 时不触发 SIGPIPE（Linux）
            - `MSG_PEEK`：recv常用，偷看数据，不从缓冲区移除
            - `MSG_OOB`：发送 / 接收 TCP 带外数据（紧急数据）
            - `MSG_WAITALL`：recv使用，本次调用阻塞等待完成len数据收集才返回
            - `MSG_DONTWAIT`：recv使用，本次调用非阻塞，无数据返回-1，errno=EAGAIN/EWOULDBLOCK
    ``` cpp
    ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                     struct sockaddr *src_addr, socklen_t *addrlen);

    ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
                   const struct sockaddr *dest_addr, socklen_t addrlen);

#### 常用设置

- 端口复用

``` cpp
int on=1;
setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on));
```

- 多进程同端口监听

``` cpp
int on=1;
setsockopt(fd, SOL_SOCKET, SO_REUSEPORT, &on, sizeof(on));
```

- 关闭nagle降低延迟（连接后）

``` cpp
int on=1;
setsockopt(fd, IPPROTO_TCP, TCP_NODELAY, &on, sizeof(on));
```

- 超时时间设置
  - 仅阻塞socket有效，非阻塞socket用epoll超时

  ``` cpp
  struct timeval tv={3,0};//3s
  setsockopt(fd, SOL_SOCKET, SO_RCVTIMEO, &tv, sizeof(tv));
  ```
- TCP保活

``` cpp
int on=1;
setsockopt(fd, SOL_SOCKET, SO_KEEPALIVE, &on, sizeof(on));
```

#### 示例代码

##### tcp

- 服务端

``` cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

#define PORT 8888
#define BUF_LEN 1024

int main(void)
{
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);

    // 端口复用，bind前设置
    int on = 1;
    setsockopt(listen_fd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on));

    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);

    bind(listen_fd, (struct sockaddr *)&serv_addr, sizeof(serv_addr));
    listen(listen_fd, 5);

    printf("server listen on port %d\n", PORT);

    struct sockaddr_in cli_addr;
    socklen_t cli_len = sizeof(cli_addr);
    int conn_fd = accept(listen_fd, (struct sockaddr *)&cli_addr, &cli_len);

    // 打印客户端信息
    char ip_buf[INET_ADDRSTRLEN];
    inet_ntop(AF_INET, &cli_addr.sin_addr, ip_buf, INET_ADDRSTRLEN);
    printf("client connect: %s:%d\n", ip_buf, ntohs(cli_addr.sin_port));

    char buf[BUF_LEN];
    ssize_t n = recv(conn_fd, buf, BUF_LEN-1, 0);
    if(n > 0)
    {
        buf[n] = '\0';
        printf("recv: %s\n", buf);
        send(conn_fd, "hello from server", strlen("hello from server"), 0);
    }

    close(conn_fd);
    close(listen_fd);
    return 0;
}
```

- 客户端

``` cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

#define PORT 8888
#define BUF_LEN 1024

int main(void)
{
    int sock_fd = socket(AF_INET, SOCK_STREAM, 0);

    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
    inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr);

    connect(sock_fd, (struct sockaddr *)&serv_addr, sizeof(serv_addr));

    send(sock_fd, "hello from client", strlen("hello from client"), 0);

    char buf[BUF_LEN];
    ssize_t n = recv(sock_fd, buf, BUF_LEN-1, 0);
    if(n > 0)
    {
        buf[n] = '\0';
        printf("recv: %s\n", buf);
    }

    close(sock_fd);
    return 0;
}
```

##### udp

- 服务端

``` cpp
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

#define PORT 9999
#define BUF 1024

int main(void)
{
    int fd = socket(AF_INET, SOCK_DGRAM, 0);

    int on = 1;
    setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on));

    struct sockaddr_in serv;
    memset(&serv,0,sizeof(serv));
    serv.sin_family = AF_INET;
    serv.sin_port = htons(PORT);
    serv.sin_addr.s_addr = htonl(INADDR_ANY);

    bind(fd, (struct sockaddr*)&serv, sizeof(serv));

    char buf[BUF];
    struct sockaddr_in peer;
    socklen_t peer_len = sizeof(peer);

    printf("udp server port:%d\n", PORT);
    while(1)
    {
        // 接收，拿到对端地址
        ssize_t n = recvfrom(fd, buf, BUF-1, 0,
                            (struct sockaddr*)&peer, &peer_len);
        if(n <=0) break;
        buf[n] = '\0';

        char ip[INET_ADDRSTRLEN];
        inet_ntop(AF_INET, &peer.sin_addr, ip, INET_ADDRSTRLEN);
        printf("recv %s:%d → %s\n", ip, ntohs(peer.sin_port), buf);

        // 回射给发送方
        sendto(fd, buf, n, 0, (struct sockaddr*)&peer, peer_len);
    }
    close(fd);
    return 0;
}
```

- 客户端

``` cpp
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>

#define PORT 9999
#define BUF 1024

int main(void)
{
    int fd = socket(AF_INET, SOCK_DGRAM, 0);

    struct sockaddr_in serv;
    memset(&serv,0,sizeof(serv));
    serv.sin_family = AF_INET;
    serv.sin_port = htons(PORT);
    inet_pton(AF_INET, "127.0.0.1", &serv.sin_addr);

    const char* msg = "hello udp";
    //发送
    sendto(fd, msg, strlen(msg), 0,
           (struct sockaddr*)&serv, sizeof(serv));

    char buf[BUF];
    struct sockaddr_in peer;
    socklen_t plen = sizeof(peer);
    //接收应答
    ssize_t n = recvfrom(fd, buf, BUF-1, 0,
                         (struct sockaddr*)&peer, &plen);
    if(n>0)
    {
        buf[n] = '\0';
        printf("recv reply: %s\n", buf);
    }

    close(fd);
    return 0;
}
```

### dns（udp）

- 端口：53
- 主要功能：域名-\>IP、主机别名
- 本地域名服务器：配置本地连接时填写的DNS服务器地址，最近的ISP的本地域名解析服务器
- hosts文件：本地域名解析，优先级高于任何DNS服务器，内部完成域名到ip的映射
  \#### 域名
- 域名结构：
  - 顶级域名
    - 国家级：cn、us、uk
    - 通用：com、net、gov
  - 二级域名
  - 三级域名
- 域名查找：
  - 主机到本地dns一般递归，dns服务器之间大多迭代
  - 递归查询：一直往上查到根域名服务器，根代替你接着查，查到就返回给你
  - 迭代查询：本地域名服务器帮你一个一个服务器查，根只返回给本地域名服务器下一个服务器地址
    \#### dns消息（\>12B，超过512B改用tcp）
- **Header 首部（固定 12 字节，必选）**
  - **ID（事务标识符，2B）**：客户端随机生成，请求与响应 ID 相同
  - Flags 标志（2B）：
    - QR：0 = 查询，1 = 响应
    - Opcode：0 = 标准查询
    - AA：1 = 权威服务器给出的回答
    - TC：1 = UDP 报文截断，应改用 TCP
    - RD：1 = 客户端希望递归查询
    - RA：1 = 服务器支持递归
    - RCODE 返回码：0 正常；3 域名不存在 (NXDOMAIN)
  - QDCOUNT：问题段条目数
  - ANCOUNT：回答段资源记录数
  - NSCOUNT：授权段资源记录数
  - ARCOUNT：附加段资源记录数
- **Question 问题段（可变，查询必带）**
  - **QNAME**：待查询域名（长度编码，域名压缩）
  - **QTYPE**：记录类型：A、AAAA、CNAME、MX、NS
  - QCLASS：一般为 IN=1 互联网
- **Answer 回答段（可变，响应放解析结果）**
- **Authority 授权段（可变，权威服务器域名）**
- **Additional 附加段（可变，辅助信息，如 NS 服务器 IP）**
  - 以上三段格式一样
  - NAME：域名
  - TYPE：记录类型
  - CLASS：类别
  - **TTL**：缓存存活秒数
  - RDLENGTH：RDATA 长度
  - RDATA：真正数据（A 记录就是 IPv4 地址）
    ![](../computer-network/3024f1ea6642b75a3cb9e8fdb0a71e550823962e.png)
- dns记录类型：
  - A：IPv4；AAAA：IPv6
  - CNAME：别名，不能填 IP，只能填域名
  - MX：邮件服务器记录，有优先级
  - NS：DNS 服务器记录
  - PTR：反向解析（IP 查域名）
  - SOA：一个域必须有，区域参数记录（过期时间、序列号）
  - SRV：标记服务对应的主机和端口
    \### http（tcp）
- 端口：http-80、https-443
- 无状态、无连接
- WWW=HTML+HTTP+URL
- URI：用字符串标识互联网资源，`协议名://登录信息@服务器地址:服务器端口号/文件路径?查询字符串#片段标识符`
- URL：资源在互联网的位置
- 虚拟主机：一个IP下可以对应多个服务器，服务器之间用主机名或域名区分
- 网关：网络边界设备，不同网段之间的出入口，代理也是网关的一种
  - 正向代理：代替客户端访问外网，隐藏客户端ip
  - 反向代理：代替服务器接收请求，隐藏服务器ip，可以在此做负载均衡
  - 透明代理：中间网络链路设置，流量强制经过，可以在此做流量控制和攻击防护
- 隧道：把完整原始数据包，当作载荷，外面套一层新报文头，在中间网络传输，逻辑通路叫**隧道**，常用于加密安全通信VPN，常见协议：GRE、IPSec‑ESP、L2TP、WireGuard、HTTP‑CONNECT
  \#### HTTP报文
- HTTP 请求报文（客户端→服务端）
  - 请求行：`请求方法 资源路径 HTTP版本`
    - 请求方法：GET、POST、PUT、DELETE 等
    - 资源路径：访问的资源 URI，不带域名
    - HTTP 版本：常用`HTTP/1.1`
  - 请求头：多个键值对 `Key: Value`
    - `Host`：目标域名，HTTP1.1**必须要有**
    - `User‑Agent`：客户端浏览器 / 程序信息
    - `Cookie`：携带会话 cookie
    - `Content‑Type`：请求体的数据格式（application/json 等）
    - `Content‑Length`：请求体字节长度
    - `Accept`：客户端能接收什么类型返回数据
  - 空行：**必须存在**，单独一行，标记请求头结束
  - 请求体（可选）：存放表单、JSON 等提交的数据
- HTTP 响应报文（服务端→客户端）
  - 状态行：`HTTP版本 状态码 状态描述`
    - HTTP 版本：常用`HTTP/1.1`
    - 状态码：3 位数字，代表处理结果
      - 2xx 成功；
      - 3xx 重定向；
      - 4xx 客户端错误；
      - 5xx 服务器错误
    - 状态描述：文本提示，协议不强制解析，客户端只识别 3 位数字状态码
  - 响应头：多个键值对 `Key: Value`
    - `Server`：服务器软件版本
    - `Content‑Type`：返回体的数据类型
    - `Content‑Length`：响应体字节大小
    - `Set‑Cookie`：下发 cookie 给客户端
    - `Cache‑Control`：缓存控制
  - 空行：**必须存在**，单独一行，标记响应头结束
  - 响应体（可选）：返回真实数据：HTML、JSON、图片二进制等
    ![](../computer-network/573db58f7c7b334daf6cf549e3567538636c5a6f.png)

##### 元数据

###### 内容编码Content‑Encoding

- 端到端编码，**对资源本体做压缩**；客户端和服务端协商，客户端解压还原原始资源
- 请求头Accept‑Encoding，响应头Content‑Encoding
- 编码算法：
  - **identity**：不压缩，默认
  - **gzip**：兼容性最好，通用文本压缩
  - **deflate**：zlib 封装 deflate 算法，老旧方案
  - **br（Brotli）**：压缩率比 gzip 更高，现代浏览器支持好，优先用于静态资源
    \###### 数据格式Content-Type
- MIME 类型：请求 / 响应体的数据格式
- 常见mime类型：
  - `text/plain`：纯文本
  - `text/html`：HTML 网页
  - `text/css`：CSS 样式表
  - `application/javascript`：JS 脚本
  - `application/json`：JSON 数据（接口最常用）
  - `application/x‑www‑form‑urlencoded`：表单 url 编码，普通 form 表单默认
  - `multipart/form‑data`：表单上传文件，表单多部分格式
  - `image/jpeg`、`image/png`：图片类型
    \###### 内容协商
- 服务器驱动协商、客户端驱动协商、透明代理协商等等
- Accept、Accept-Charset、Accept-Encoding、Accept-Language、Content-Language
  \###### 数据分片
- 分块传输编码
  - 特点：客户端没有选择片段的权力，服务器不知道总大小，把全部内容分块吐过来
  - 响应头部：\`Transfer‑Encoding: chunked
  - 应用：动态生成内容、流式输出、大流式响应
  - 报文示例：
    \`\`\` txt
    HTTP/1.1 200 OK
    Content‑Type: text/plain
    Transfer‑Encoding: chunked

7 //第一块 7 字节内容
Hello,
6 //第二块 6 字节内容
World!
0 //大小为 0 的块，代表传输结束

    - 范围请求 Range
        - 特点：**客户端主动指定想要哪一段字节**，服务器只返回这部分片段，不是完整资源，返回状态码`206`，如果范围非法 / 不支持，返回 200，下发完整文件
        - 请求头：`Range: bytes=start‑end`；服务器支持会返回`Accept‑Ranges: bytes`，响应头：`Content‑Range`标记返回片段位置
        - 应用：断点续传、视频拖拽播放、多线程下载
        - 报文示例：
    ``` txt
    // 请求头
    GET /test.txt HTTP/1.1
    Host: example.com
    Range: bytes=0‑31

    // 响应头
    HTTP/1.1 206 Partial Content
    Accept‑Ranges: bytes
    Content‑Range: bytes 0‑31/1000
    Content‑Length: 32
    Content‑Type: text/plain

    这里是0‑31的32字节片段内容

###### 多部分对象集合Multipart

- 一个 HTTP 报文体内部包含多个独立子对象，使用`boundary`分隔字符串切割各个部分；每一个 part 可以拥有自己独立的头部和数据类型
- 语法规则：
  - `--boundary值`：代表一个 part 的开始
  - `--boundary值--`：代表整个多部分内容结束
- multipart/form‑data：

``` txt
POST /upload HTTP/1.1
Host: test.com
Content‑Type: multipart/form‑data; boundary=AaB03x

--AaB03x
Content‑Disposition: form‑data; name="username"

zhangsan
--AaB03x
Content‑Disposition: form‑data; name="avatar"; filename="a.jpg"
Content‑Type: image/jpeg

···图片二进制数据···
--AaB03x--
```

- multipart/byteranges：

``` txt
HTTP/1.1 206 Partial Content
Content‑Type: multipart/byteranges; boundary=THIS_SEP

--THIS_SEP
Content‑Type: text/plain
Content‑Range: bytes 0‑49/1000

0‑49字节内容
--THIS_SEP
Content‑Type: text/plain
Content‑Range: bytes 100‑149/1000

100‑149字节内容
--THIS_SEP--
```

##### 请求方法

- **GET**：获取资源数据
- **POST**：提交数据，创建资源
- **PUT**：全量替换 / 更新资源，PUT和DELETE有安全性问题，只有Restful标准应用才开放使用
- **DELETE**：删除指定资源
- **PATCH**：局部更新资源
- **HEAD**：同 GET，仅返回响应头，不返回响应体
- **OPTIONS**：查询服务器支持的请求方法，用于 CORS 预检，返回Allow字段告知支持的方法
- **TRACE**：回显接收到的请求，用于链路调试，Max-Forwards字段说明中转跳数
- **CONNECT**：建立 TCP 隧道，用于代理转发 HTTPS 流量，会使用SSL和TLS把通信内容加密
  \##### 状态码
- 信息类
  - 100 Continue：继续
  - 101 Switching Protocols：切换协议
- 成功类
  - 200 OK：成功
  - 201 Created：已创建
  - 202 Accepted：已接受
  - 203 Non‑Authoritative Information：非权威信息
  - 204 No Content：无内容
  - 205 Reset Content：重置内容
  - 206 Partial Content：部分内容
- 重定向类
  - 300 Multiple Choices：多种选择
  - 301 Moved Permanently：永久移动
  - 302 Found：临时找到
  - 303 See Other：查看其他位置
  - 304 Not Modified：未修改
  - 307 Temporary Redirect：临时重定向
  - 308 Permanent Redirect：永久重定向
- 客户端错误
  - 400 Bad Request：错误请求
  - 401 Unauthorized：未授权
  - 403 Forbidden：禁止访问
  - 404 Not Found：未找到
  - 405 Method Not Allowed：方法不允许
  - 406 Not Acceptable：不可接受
  - 407 Proxy Authentication Required：需要代理认证
  - 408 Request Timeout：请求超时
  - 409 Conflict：冲突
  - 410 Gone：资源已移除
  - 411 Length Required：需要长度
  - 412 Precondition Failed：预处理失败
  - 413 Payload Too Large：请求体过大
  - 414 URI Too Long：URI 过长
  - 415 Unsupported Media Type：不支持媒体类型
  - 416 Range Not Satisfiable：范围无法满足
  - 417 Expectation Failed：期望失败
  - 422 Unprocessable Entity：无法处理实体
  - 429 Too Many Requests：请求过多
- 服务端错误
  - 500 Internal Server Error：服务器内部错误
  - 501 Not Implemented：未实现
  - 502 Bad Gateway：错误网关
  - 503 Service Unavailable：服务不可用
  - 504 Gateway Timeout：网关超时
  - 505 HTTP Version Not Supported：HTTP 版本不支持
    \#### 持久连接
- HTTP1.0默认短连接，一次请求‑响应完成，TCP 连接立刻断开，HTTP/1.1 **默认开启持久连接**，**同一个 TCP 连接上，连续发送多次 HTTP 请求与响应，连接不会马上关闭**，复用 TCP 连接，减少握手挥手开销。
- 长连接不是永久不关闭，有超时时间、最大请求次数限制
- 开启：`Connection: keep‑alive`
- 关闭：`Connection: close`
- 问题：串行阻塞，前面请求慢，后面请求必须等待，解决：HTTP/2 多路复用
  \#### cookie和session
  \##### cookie
- 是**服务器下发给浏览器，保存在客户端的一小段文本数据**，保存状态信息弥补http无状态缺陷
- 服务器响应头`Set‑Cookie` 下发，浏览器后续同源请求自动携带`Cookie` 元数据
- 存储位置：客户端浏览器本地，不安全，不能存储敏感数据
- 内容：`name=value`；附加安全属性 Domain（域名）、Path（路径）、Expires/Max‑Age（过期时间）、Secure（仅https才发送）、HttpOnly（js禁止读取）、SameSite（禁止跨站携带）
  \##### session
- 服务端会话技术，会话数据保存在服务器，为每一个访问用户生成唯一的`sessionId`，一般通过 Cookie 发送 sessionId
- 用户第一次访问服务器生成唯一 sessionId，通过`Set‑Cookie` 下发
- 存储位置：**服务器内存 / Redis / 数据库**
- 安全：设置 session 超时时间，超时销毁会话
  \#### HTTPS
- HTTPS 不是应用层全新协议，是 HTTP 套在 TLS 上面。
- 端口：443
- HTTPS=HTTP+加密（防中间人）+认证（防钓鱼）+完整性保护（防恶意篡改）
- HTTPS = HTTP + TLS/SSL
- 不轻易选择HTTPS的理由：消耗网络带宽，还有证书费
  \##### 加密方式
- 对称加密：客户端服务器共用一把密钥
- 非对称加密：公钥公开，私钥服务器自己保管
  \##### 数字证书
- 数字证书 = 域名 + 服务器公钥 + CA 的数字签名
- CA的公钥传输：不传输，浏览器内置常用认证机关的公钥
- 自签名证书：使用OpenSSL构建自己的认证机构，给自己颁发服务器证书，但是一般没啥可信度，仅内网测试使用
- 信任链：
  - 中间 CA 证书：根证书签发中间证书；中间证书签发网站业务证书
  - 根证书：操作系统、浏览器内置，CA 自己给自己签发（自签名证书）
    \##### 通信流程
- TCP 握手：建立 TCP 连接，端口 443
- TLS 握手（明文）：
  - 客户端发送 `Client Hello`：TLS 版本、支持的加密套件、客户端随机数
  - 服务器返回 `Server Hello`：选定加密套件、服务端随机数、下发 CA 数字证书（内含服务器公钥）
  - **客户端校验证书**：用操作系统内置 CA 根证书校验证书是否合法
- 密钥传递（明文）：
  - 客户端生成预主密钥，**使用服务器公钥加密**发送给服务端
  - 服务器用自己的私钥解密得到预主密钥
  - 双方结合客户端随机数、服务端随机数、预主密钥，各自计算出相同的**对称密钥**
  - 两端发送`Change Cipher Spec`，通知后续全部报文开启加密；`Finished`消息校验握手完整性
- 业务传输（密文）：使用**会话密钥（对称加密）**加密原始 HTTP 请求 / 响应报文传输
  ![](../computer-network/59cfaab6ee941eb72093b04933ba6fb5ec0d699e.png)
  \#### http认证
- 服务器：`401 Unauthorized` + `WWW‑Authenticate`告诉客户端需要什么认证方式
- 客户端：再次请求带上`Authorization`请求头携带凭证
- **Proxy‑Authenticate**：代理服务器质询认证；**Proxy‑Authorization**：客户端给代理的认证凭证
- 认证类型：
  - Basic 基本认证：`用户名:密码`做**Base64 编码**放到`Authorization: Basic xxx`，必须HTTPS
  - Digest 摘要认证：服务器下发随机数 nonce，客户端做 MD5 哈希运算生成响应摘要，`Authorization: Digest xxx`
  - Bearer 令牌认证：token 令牌，放在`Authorization: Bearer <token>`
  - 域认证（Negotiate、Keberos、NTLM）：企业内网使用，浏览器自动完成身份校验，[网络安全](../11.%20网络安全/网络安全.md)有相关攻击
- 业务级认证：
  - Form-Base认证：某个关键操作需要账号密码通过才可以提交
  - 双因素认证 2FA：
    - 你知道的（知识因子）：密码、PIN 码、安全问题
    - 你拥有的（持有因子）：手机、TOTP 令牌 App、硬件密钥 YubiKey
    - 你是谁的（固有因子）：指纹、人脸等生物识别
    - 取**两种不同类别因子**组合，安全程度相当高
      \#### http缓存
- 强缓存：不向服务器发请求，直接读取本地缓存，状态码 200
- 协商缓存：客户端带上校验标识去向服务器询问资源是否更新，未修改返回 **304 Not Modified**
  \#### http代理
- 位于**客户端与源服务器中间**，转发 HTTP 请求与响应
- 逐跳头部（Hop‑by‑hop）：只对当前这一跳代理生效，代理不会转发给下一跳
  - 生效元数据：Connection、Transfer‑Encoding、Keep‑Alive、Upgrade、Via
- 端到端头部（End‑to‑end）：要一直传递到最终客户端 / 源服务器
  - 生效元数据：Cache‑Control、Content‑Type、Cookie、Set‑Cookie、ETag、Last‑Modified
    \#### html5
- 新一代 HTML 标准，包含 HTML 标记、JS API、CSS 配套，淘汰 Flash，原生音视频
- localStorage/sessionStorage 是 HTML5 本地存储，**不会随 http 请求自动携带，和 Cookie 不一样**
- HTML5 只是浏览器给 JS 开放了调用 [websocket](6.2%20计算机网络.md#websocket) 的接口
- 新增语意标签：
  - `<header>`：页面 / 区块头部
  - `<nav>`：导航区域
  - `<main>`：页面主体内容
  - `<section>`：内容区块
  - `<article>`：独立文章内容
  - `<aside>`：侧边栏附属信息
  - `<footer>`：页脚
- 新增api：
  - `<canvas>`：画布，JS 绘制 2D 图形
  - `<video>/`：音视频媒体
  - **WebSocket**：浏览器全双工通信 API
  - **Fetch API**：新一代 AJAX 接口，替代 XMLHttpRequest
  - LocalStorage / SessionStorage：浏览器本地存储
    - localStorage：永久存储，手动清除
    - sessionStorage：会话存储，标签关闭清空
  - Geolocation：获取设备地理位置
  - File API：JS 读取本地文件
  - History API：前端路由，`pushState`实现 SPA 单页应用无刷新跳转
  - Web Worker：JS 后台子线程，不阻塞 UI 渲染
    \##### ajax
- 浏览器 JS 技术，异步发起 HTTP 请求，页面不用整体刷新，局部更新网页内容，现在普遍使用 JSON；**底层仍然是普通 HTTP 请求**
- api
  - 原生：`XMLHttpRequest` 对象，传统 AJAX
  - 现代标准：`fetch()` API（Promise 风格）
    \#### websocket
- **WebSocket 是在 HTTP 握手基础上升级得到的**全双工通信协议，一条连接双向收发数据
- **使用 HTTP/1.1 完成握手**，发送升级请求，完成后协议切换为 WebSocket，不再使用 HTTP
- 报文不再是 HTTP 文本，是**二进制帧格式**
- 端口：ws:// 默认 80；wss://(加密 WebSocket) 默认 443
- 应用：聊天、实时通知、数据大屏
- 握手报文（http）
  - 请求：
    - 请求元数据：
      - `Upgrade: websocket`：告知服务器想要升级协议到 websocket
      - `Connection: Upgrade`：标记本次要做协议升级
      - `Sec‑WebSocket‑Key`：客户端随机生成 Base64 字符串，用于握手校验，防误连接
      - `Sec‑WebSocket‑Version:13`：WebSocket 版本
  - 响应：
    - 状态码：**101 Switching Protocols**
    - 响应元数据
      - `Upgrade: websocket`
      - `Connection: Upgrade`
      - `Sec‑WebSocket‑Accept`：服务器对 Sec‑WebSocket‑Key 做固定算法计算返回，客户端校验握手合法性
- 二进制帧格式：用的不多，需要再补充
  \#### http/2
- **二进制帧**：把 HTTP 报文拆成二进制帧传输，不再是文本；解析高效。
- **多路复用（核心）**：**一条 TCP 连接上，多个请求 / 响应并行交错传输，互不阻塞**，彻底解决 HTTP1.1 的队头阻塞。
  - 逻辑流 stream，每个请求分配独立 stream id；帧携带 stream id，接收端按 id 组装。
- **头部压缩 HPACK**：专门压缩 HTTP 头部，消除重复头部开销。
- **服务端推送 Server‑Push**：服务器可以主动向客户端推送资源，不用客户端发起请求。
- **流优先级**：给不同 stream 设置优先级，重要资源优先调度。
- 依然保留语义：请求方法、状态码、MIME、Cookie 全部和 HTTP1 一致，只是传输表达方式改变。
  \### dhcp（udp）
- 端口：服务器-67、客户端-68
- 广播报文不能跨路由器转发（最多支持2层），如果 DHCP 服务器不在同一个局域网，将广播报文改成单播，转发给远端 DHCP 服务器
- 特殊地址：收不到任何 DHCP 服务器响应，169.254.0.0/16 APIPA，只能2层以内通信
- 全流程：
  - Discover 发现（客户端广播）：源 IP：`0.0.0.0`，目的 IP：`255.255.255.255`
  - Offer 提供（服务器回复）：返回预分配的 IP 地址、掩码、租期
  - Request 请求（客户端广播）：从多个 Offer 挑选一个（通常第一个到达），**广播 DHCP Request**，表示使用这个服务器的IP
  - Acknowledge 确认（服务器确认）：正式分配 IP，携带完整参数：IP、子网掩码、网关、DNS、**租约时间**
- 分配方式：
  - **自动分配**：第一次获取永久使用。
  - **动态分配**：租期模式，到期回收，主流。
  - **静态分配（地址保留）**：服务器配置 MAC 地址绑定固定 IP，同一个设备永远拿到同一个 IP。
- 续租：
  - **50% 租期（T1）**：客户端单播向 DHCP 服务器发起续约请求 Request
  - **87.5% 租期（T2）**：单播续约失败，则**广播**发起续约，找任意 DHCP 服务器
  - **100% 租期到期**：IP 直接失效。主机必须停止使用该 IP，重新 Discover
    \### 其他协议
    \#### email
- smtp发送pop3接收
- imap：接收 + 远程管理邮件，多端同步
  \#### bt
- p2p协议
- torrent(洪流): 交换同一个文件的文件块的节点组
- chunk(文件块)：一个洪流中的对等方下载等长度的文件块，典型长度256KB
- 邻近对等方：成功创建一个TCP连接的对等方
- Tracker服务器：追踪参与torrent的节点。
- 索引方法：（chunk信息到邻近节点IP+端口号的映射）
  - 集中式索引
  - 洪泛式查询
  - 层次式覆盖网络（对等方中有超级节点，连接快速）
    \## 传输层（端到端）
- 负责端到端连接控制、复用分用（不同进程用同一套socket）、流量控制、差错控制、拥塞控制
- 协议：tcp、udp、sctp
- 复用分用：
  - tcp：面向连接，为不同客户端开不同connect Socket，源IP，源端口，目的IP，目的端口
  - udp：无连接，不同源IP和端口的数据被导向同一个Socket，目的IP，目的端口
    \#### 伪首部：
- 在数据payload中，把 IP 层的源目 IP 纳入校验，防止数据包错投到别的主机进程
- `源IP(4B) | 目的IP(4B) | 0(1B) | 协议号=17(1B) | 总长度(2B)`
- 协议号：udp-17、tcp-6
  \### udp
- 特点：不可靠、无连接、低时延、全双工
- 应用：TFTP、DNS、SNMP、RTP、音视频多媒体应用
  \##### udp报文（\>8B）
- 源端口 Source Port：发送方端口；**可选，不需要回复填 0**
- 目的端口 Dest Port：接收进程端口
- UDP 长度 Length：整个 UDP 报文总字节 = UDP 头部 + 数据
- 校验和 Checksum：差错检测；IPv4 可填 0 关闭校验；IPv6 强制开启；**计算包含伪首部**
- 数据载荷（Payload）：具体发送数据
  ![](../computer-network/6a424b4cd1233548cecb53d43a38921beb772b25.png)
  \### tcp
- 特点：可靠、面向连接、高时延、全双工
- 应用：FTP、HTTP、TELNET
  \##### tcp报文（\>20B）
- MSS：一个 TCP 报文最大数据载荷
- 源端口 (16) / 目的端口 (16)：应用进程端口
- 序列号 seq：逻辑序号（SYN/FIN）也要占用
- 确认号 ack：期望收到**下一片数据**的 seq，代表之前字节已经全部收到
- 头部长度：TCP 头部长度，4bit，单位4字节
- 未使用：保留，支持拓展
- **6 位标志位（重点）**
  - `URG`：紧急指针有效
  - `ACK`：**确认号有效，绝大多数包置 1**
  - `PSH`：推送，接收方立刻上交应用层，不缓冲
  - `RST`：重置连接，异常断开
  - `SYN`：同步，建立连接
  - `FIN`：结束，关闭连接
- 窗口大小 window：**接收窗口 rwnd**，流量控制，告诉对方我还能接收多少字节。
- 校验和：互联网校验和
- 紧急指针：URG 标志置位才生效，指向紧急数据偏移量
- 选项：
  - MSS：最大分段大小，TCP 数据部分最大字节，不含 TCP 头
  - SACK：选择确认，实现 SR 选择重传，只重传丢失段
  - Window Scale：窗口扩大因子，解决 16bit 窗口上限
  - Timestamp：时间戳，RTT 计算、防序号回绕
    ![](../computer-network/5503eaa303107ebc100dc00c4cc44dd173e5affa.png)
    \##### 缓冲区
- 缓冲区是**内核空间的内存**，不是应用程序内存。每一个 TCP 连接，内核都会分配独立的发送缓冲区、接收缓冲区。
- 应用层`send()`、`recv()`本质是和内核缓冲区交互，返回成功仅仅是成功读出或写入缓冲区
- 缓冲区满会阻塞进程直到有空闲空间，空闲空间与`rwnd`有关
- 半关闭时会把剩下数据发完才发送FIN
- 接收缓冲区：接收到会自动排序去重放入缓冲区
- 发送缓冲区：必须接收到ack，才把数据从发送缓冲区清除（因为有重传）
- close与shutdown：
  - close全关闭：缓冲区直接丢弃未处理数据
  - shutdown半关闭：发送缓冲发送剩余数据，发完才发出FIN，接收缓冲可以继续读
- Linux内核配置：
  - `net.core.wmem_default`：发送缓冲区默认大小
  - `net.core.wmem_max`：发送缓冲区最大值
  - `net.core.rmem_default`：接收缓冲区默认大小
  - `net.core.rmem_max`：接收缓冲区最大值
    \##### 连接管理
- 关键易错：seq是各自进程决定的两边不一样，SYN、FIN 会推进1 个序列号；ACK 不推进序列号
- 为什么握手3次挥手4次：握手的SYN和ACK可以合并，挥手时服务端收到FIN可能还有剩余业务数据要发送，ACK 和 FIN 不能合并
- `CLOSE_WAIT`大量堆积，代表**服务端程序忘记 close 套接字**
- 半连接：服务端三次握手收到第一次SYN，就建立半连接存入半连接队列（容量在linux内核参数`tcp_max_syn_backlog`中配置而不是`listen`的`backlog`），等待超时才释放
  \###### 三次握手
- 全流程：SYN、SYN+ACK、ACK
- 客户端调用`connect()`：CLOSED → SYN_SENT
- 服务端调用`accept()`：SYN_RCVD → ESTABLISHED，返回 conn_fd，**应用程序正式拿到连接**

```
sequenceDiagram
    participant Client as 客户端进程
    participant Server as 服务端进程
    Note over Server: 初始状态：LISTEN<br/>socket→bind→listen后进入LISTEN

    Client->>Server: SYN (seq=x) 【第一次握手】
    Note over Client: 状态：SYN_SENT

    Server->>Client: SYN+ACK (seq=y, ack=x+1) 【第二次握手】
    Note over Server: 状态：SYN_RCVD

    Client->>Server: ACK (ack=y+1) 【第三次握手】
    Note over Client: 状态：ESTABLISHED
    Note over Server: 收到ACK → 状态ESTABLISHED<br/>accept返回conn_fd，应用拿到连接
```

###### 四次挥手

- 全流程：FIN、ACK、FIN、ACK
- 客户端调用 close ()：ESTABLISHED → FIN_WAIT_1
- CLOSE_WAIT：**内核收到 FIN，通知应用层对端关闭；此时服务端应用还可以 send 发送剩余数据**
- 服务端应用调用 close ()：CLOSE_WAIT → LAST_ACK
- TIME_WAIT：保持2msl
  - 保证最后一个ack能成功到达，防止丢包导致对方重传fin
  - 等待信道内残旧报文消失

  ```
  sequenceDiagram
    participant Client as 客户端
    participant Server as 服务端
    Note over Client,Server: 初始状态 ESTABLISHED

    Client->>Server: FIN, seq=m 【第一次挥手】
    Note over Client: 状态 FIN_WAIT_1

    Server->>Client: ACK, ack=m+1 【第二次挥手】
    Note over Client: 状态 FIN_WAIT_2
    Note over Server: 状态 CLOSE_WAIT
    Note over Server: 此时服务端还可以继续发数据给客户端

    Server->>Client: FIN, seq=n 【第三次挥手】
    Note over Server: 状态 LAST_ACK

    Client->>Server: ACK, ack=n+1 【第四次挥手】
    Note over Client: 状态 TIME_WAIT<br/>等待2MSL
    Note over Server: 收到ACK → CLOSED

    Note over Client: 2MSL超时 → CLOSED
  ```

  ###### SYN洪泛攻击
- 利用半连接机制，伪造大量随机虚假源 IP，疯狂发送 SYN 报文不回复，堵塞服务器内核内存和半连接队列
- 防御手段：
- SYN-Cookie：不保存半连接，将连接信息加密编码为SYN-Cookie作为SYN+ACK的序列号seq，以后拿到ACK后直接从ACK的seq解码还原连接信息，在linux内核参数`net.ipv4.tcp_syncookies`中设置，0表示关闭1表示队列满开启2表示无条件开启
- 防火墙：代替服务器完成三次握手，先确认是真实客户端才握手
- 调内核参数：增大半连接队列`tcp_max_syn_backlog`，减少等待时间和重传次数`tcp_synack_retries`
  \##### 可靠传输
- seq+ack保证按顺序传输，seq防止重复，seq乱序重排
- 发送方实际发送上限：`min(rwnd, cwnd)`，`rwnd`是接收方通知的，`cwnd`是根据网络情况自己计算的
  \###### 校验和
- **16 位二进制反码求和（One's‑Complement Sum）**，RFC1071 定义，IP/UDP/TCP 全部用这套算法
- 校验和只能检测错误不能修复错误，出错直接丢包
- 编码：
  - 按 16bit 分组，如果数据字节为奇数，末尾补1字节0
  - 校验和字段置 0
  - 所有 16bit 分组做**反码加法**：相加产生的进位，回加到低 16 位
  - 将最终总和做**按位取反**，填入校验和字段
- 解码：
  - 同样构造参与校验的全部数据
  - 比较生成的校验和一致不一致
    \###### 超时重传机制
- RTT：报文发送到收到ACK的往返时间
- RTO：重传计时器超时时间，根据RTT计算得来（一些算法比如加权平均RTT、Karn），最低200ms
- 发送端发送后启动计时器，报文或ACK超时未收到触发重传，且触发超时后退
  - 超时重传：重传计时器到期（RTO）
  - 快速重传：收到3次冗余ack（重复强调需要的序号）
- 超时后退机制：每次发生一次重传，下一次RTO翻倍，持续多次到达最大重传次数后直接断开连接报告连接错误
  \###### 流量控制机制
- 防止发送方发送速率太快，接收方接收缓冲区装不下，造成报文丢失
- 基础设施：接收方的`rwnd`窗口字段
- 接收方每次回复 ACK 报文时，把自己缓冲区剩余可用大小放进头部`window`字段，告诉发送方：**我现在最多还能收多少字节**
- 窗口更新：接收方使用`recv()`读取数据，会发送**窗口更新 ACK**通知发送方
- 窗口探测计时器：为了防止窗口更新ACK丢失，在窗口为0后发送方定期发送1 字节探测报文
  \###### 拥塞控制机制
- 防止发送方发得太快，把中间路由器链路压垮、路由器缓存溢出丢包
- 基础设施：发送方的`cwnd`窗口配置（本地维护）

1.  慢启动：不知道网络负载，从小窗口开始试探，一般1-10MSS，每收到一个 ACK，cwnd翻倍，直到阈值`ssthresh`，退出慢启动进入拥塞避免
2.  拥塞避免：每经过 1 个 RTT，`cwnd += 1 MSS`，线性增长
    - 超时重传：`ssthresh`减半，`cwnd = 1 MSS`
    - 快速重传：见[超时重传机制](6.2%20计算机网络.md#超时重传机制)，快恢复：触发快重传后，`ssthresh`减半，`cwnd = ssthresh`
      \## 网络层（逐跳路由）

- 负责逻辑寻址、路由选择
- 协议：ip，icmp算2.5层，arp/rarp算1.5层
  \### ip
- 不可靠、无连接、可分片
- MTU决定了ip数据包最大能多大
  \#### ip地址
- 用于网络层唯一标识一台主机，用于路由寻址和转发
- ipv4地址：32位二进制，点分十进制
- 可用主机数 = **2\^主机位数 − 2（网络地址、广播地址）**
- 表示：
  - 网络位（网段）+主机位（设备）/ 网络位数
  - ip地址 + 子网掩码（表示网络位数）
- 分类：
  - **A类中127.0.0.0/8 预留为回环地址**

| 类别 | 高位  | 网络位 | 主机位 | 网络范围                  | 默认子网掩码     |
|------|-------|--------|--------|---------------------------|------------------|
| A    | 0     | 8 位   | 24 位  | 1.0.0.0‑126.255.255.255   | `255.0.0.0`      |
| B    | 10    | 16 位  | 16 位  | 128.0.0.0‑191.255.255.255 | `255.255.0.0`    |
| C    | 110   | 24 位  | 8 位   | 192.0.0.0‑223.255.255.255 | `255.255.255.0`  |
| D    | 1110  | ---    | ---    | 224.0.0.0‑239.255.255.255 | **组播地址**     |
| E    | 11110 | ---    | ---    | 240.0.0.0‑255.255.255.255 | 保留实验，不使用 |

- 无类别编码CIDR：
  - 自由划分掩码位数，基于**VLSM可变长子网掩码（内网细分）和CIDR路由聚合**
- 内网ip：
  - A 类私有：`10.0.0.0/8` 10.0.0.0 \~ 10.255.255.255
  - B 类私有：`172.16.0.0/12` 172.16.0.0 \~ 172.31.255.255
  - C 类私有：`192.168.0.0/16` 192.168.0.0 \~ 192.168.255.255
- 特殊地址：
  - **127.0.0.0/8**：回环地址，本机自测，不经过网卡
  - **0.0.0.0**：本机任意地址、未获取IP时默认源地址、服务端监听所有网卡
  - **255.255.255.255**：有限广播，路由器不转发
  - **169.254.0.0/16**：APIPA自动私有地址，DHCP失败自动分配
  - **网段网络地址**：主机位全0，不可分配给设备
  - **网段广播地址**：主机位全1，不可分配给设备
    \#### NAT地址转换
- 公网地址有限，**NAT 让多个内网主机共享少量公网 IP 访问互联网**。
- 映射表，内网ip端口对应公网ip端口
- 映射方式：
  - 静态NAT：手动配置，一个内网ip永久绑定一个公网ip
  - 动态NAT：内网地址池与公网地址池自动分配一对一
  - NAPT网络地址端口转换（常用）：多个内网地址映射一个公网ip，依靠不同端口区分
- 映射表老化，有超时时间，过期删除释放公网资源
  \#### ip数据包（\>20B）
- **版本 (4bit)**：IPv4 值 = 4；IPv6 值 = 6。
- **IHL 头部长度 (4bit)**：单位：**4 字节**。基础头部 20 字节，最大 60 字节头部。
- 服务类型（8bit）：现代网络把该字段复用为 DS 字段，前 6 位 DSCP 做流量分类，后 2 位 ECN 拥塞通知，路由器根据此字段做 QoS 调度
- **总长度 (16bit)**：整个 IP 数据包总字节数（IP 头部 + 数据）。最大 65535 字节。
- **标识 ID (16bit)**：IP 数据包唯一编号；**用于 IP 分片**，同一个数据包分片，ID 全部相同。
- **标志位 (3bit)**
  - bit0：保留
  - bit1：DF 位（Don't Fragment），置 1 代表**禁止分片**
  - bit2：MF 位（More Fragment），1 代表**后面还有分片**；最后一个分片 MF=0。
- **片偏移 (13bit)**：该分片数据相对于原始数据包开头的偏移，**单位 8 字节**。
- **TTL 生存时间 (8bit)**：每经过一台路由器TTL‑1，减到 0 直接丢弃。Windows 默认 128，Linux 默认 64。
- **协议 (8bit)**：标识上层是什么协议：
  - `6` → TCP
  - `17` → UDP
  - `1` → ICMP
- **头部校验和 (16bit)**：只校验 IP 头部，不校验数据部分（上层自带），互联网校验和。每经过一台路由器 TTL 改变，头部重新计算校验和。
- 源 IP (32bit)、目的 IP (32bit)：IPv4 地址。
- **选项**：可选，最多 40 字节，4 字节对齐。
  ![](../computer-network/9cce7c6dd0e9a4f84441e2fa40b8a8f22976a432.png)
  \#### MTU探测
  DF 置 1，遇到 MTU 更小链路，包丢弃，返回 ICMP 类型 3 代码 4
  \#### ip分片
- 很多防火墙、NAT 设备会丢弃 IP 分片报文，尽量避免分片
- 路由器只负责分片和传输，只有到达目的主机才重组
- 当数据包大小超过mtu就需要分片，分片的数据部分长度必须是8字节的整数倍（最后一片除外）
- 参与分片的字段：
  - 标识：同一个数据包
  - 标志：df=0允许分片，mf=1表示不是最后一片，mf=0表示是最后一片
  - 片偏移：本片在整体的偏移量，单位是8字节
- 因为ip协议本身不可靠，如果分片中途丢失，就会整个包错误，等待上层协议触发重传
  \#### PMTU探测
- 通过数据报探测源主机到目的主机整条路径上，所有链路 MTU 的最小值，tcp默认开启，udp需要应用实现
- 实现流程：
  - 发送方把 IP 头部 **DF 位置 1（禁止分片）**
  - 发送较大的 IP 数据包
  - 如果某一跳路由器 MTU 比数据包小，**不能分片（DF=1）**，直接丢弃数据包，回复 **ICMP 报文：类型 3，代码 4**，其中携带该路由器的MTU 值
  - 源主机收到这个 ICMP 应答，降低发送的 IP 包大小，再次发送不断循环直到满足所有路由器条件
- PMTU黑洞：返回的ICMP报文被防火墙丢弃或过滤，会造成小包通信正常，大包完全不通，大内容完全加载不出来
  \#### ipv6
- 地址：128位二进制，冒分十六进制，8组每组4位十六进制，前导零压缩
- 取消广播，使用组播
- 方法：`2001:db8::/64`
- 地址类型：
  - 单播地址 Unicast：标识单个接口，发送给该地址
    - 公网ipv6：前缀一般`2000::/3`
    - 链路本地地址：前缀固定`fe80::/10`
    - 唯一本地地址：`fc00::/7`
  - 组播地址 Multicast：`ff00::/8`，格式：`ff<flag><scope>::组播组ID`
    - `ff02::1`：**链路范围所有节点**（替代 IPv4 二层广播）
    - `ff02::2`：链路范围所有路由器
  - 任播地址 Anycast：从单播地址空间拿，同一个地址配置在**多个不同设备接口**。数据包路由送到**距离最近**的一台设备。
- 特殊地址：
  - `::1/128` 回环地址，等价 IPv4 `127.0.0.1`
  - `::/128` 未指定地址，等价 IPv4 `0.0.0.0`
- ipv6数据包：
  - **Version (版本)：4bit**，固定为 6，标识 IPv6 协议
  - **Traffic Class (流量类别)：8bit**，兼容 IPv4 的 TOS/DSCP+ECN，用于 QoS 拥塞标记
  - **Flow Label (流标签)：20bit**，IPv6 独有，标记同一数据流，加速设备转发
  - **Payload Length (载荷长度)：16bit**，**不含基础头部**，代表扩展头 + 传输层数据总长
  - **Next Header (下一头)：8bit**，标识后续是传输层协议 (TCP/UDP/ICMPv6) 或扩展头部
  - **Hop Limit (跳数限制)：8bit**，等价 IPv4 的 TTL，每过一台路由器 - 1，到 0 丢包防环路
  - **Source Address (源 IPv6)：128bit**，报文发送端地址
  - **Destination Address (目的 IPv6)：128bit**，报文接收端地址
    ![690](../computer-network/7fafb98b280c8455fe8e096dbb225b6fd2bcfb1a.png)
- ipv6与ipv4的区别
  - 地址长度：32bit，点分十进制｜128bit，冒分十六进制
  - 地址总量：约 42 亿｜(2\^{128})，近乎无限
  - 头部长度：20‑60 字节可变 (IHL)｜固定 40 字节基本头部，功能放在扩展头部
  - 头部校验和：**有 IP 头部校验和**｜取消网络层校验和，校验交给上层
  - 分片处理：源主机、路由器均可分片｜**仅源主机分片，路由器禁止分片，使用分片扩展头**
  - 寿命字段：TTL (生存时间)｜Hop‑Limit (跳数限制)，功能等价
  - QoS 标记：TOS 字段｜Traffic Class 流量类别，DSCP+ECN；新增 Flow‑Label 流标签
  - 广播：**支持广播地址**｜没有广播，全部使用组播
  - ARP：使用 ARP 协议（二层广播）｜NDP（ICMPv6 组播）替代 ARP
  - ICMP：ICMPv4，协议号 1｜ICMPv6，协议号 58，集成 NDP 邻居发现
  - 地址配置：静态 / DHCPv4｜静态 / DHCPv6 / **SLAAC 无状态自动配置**
  - 内网地址：RFC1918 私有地址，依赖 NAT 上网｜`fc00::/7`唯一本地地址，**默认不需要 NAT，端到端可达**
  - 帧标识：以太网类型 `0x0800`｜以太网类型 `0x86DD`
  - UDP 校验和：可选，可以关闭｜**强制开启 UDP 校验和**
  - 选项 / 扩展：选项内嵌 IP 头部｜全部功能剥离为链式扩展头部
  - PMTU：建议开启，可被分片绕过｜**强制 PMTU 探测**
    \### ICMP
- 网络层协议（接近传输层），封装在IP数据包的数据中（所以也是不可靠的），用于网络层的差错报告
- traceroute：发送 TTL=1、2、3... 的 UDP 包，每跳 TTL‑1，路由器返回 ICMP 超时，拿到每一跳路由器 IP。
- ping：就是发送 ICMP Echo 请求，目标主机回复 Echo 应答；统计往返 RTT、丢包
  \##### icmp报文（\>8B）
- **Type 类型 (1 字节)**：报文大类
- **Code 代码 (1 字节)**：细分子类型
- **Checksum 校验和 (2 字节)**：ICMP 整体校验和
- 后面 4 字节：根据不同类型用途可变
- 数据部分：路由器返回差错 ICMP 时，会把出错 IP 包的头部 + TCP/UDP 前 8 字节放在 ICMP 数据载荷里
  ![](../computer-network/7bdc53076507db1600dfe98de1dc28f883065e62.png)
  \#### icmp类型
- 差错报告
  - Type 3：目的不可达
    - Code 0：网络不可达
    - Code 1：主机不可达
    - Code 2：协议不可达
    - Code 3：端口不可达（UDP 收不到对应端口，返回）
    - **Code4：需要分片，但 DF 置 1【PMTU 探测核心】**
  - Type 4 code 0：源站抑制（已废弃）
  - Type 5 code 0：重定向，路由器告诉主机更优网关地址
  - Type 11：超时，TTL 减到 0，路由器丢弃，返回 ICMP 超时。**traceroute 原理就是靠这个**
- 查询
  - Type 8 code 0：回显请求，ping 请求
  - Type 0 code 0：回显应答，ping 回复
    \### 路由选择
    \#### 路由表
- Linux：`ip route` / `route -n` 查看路由表
- RIB：软件路由表（路由协议学习到的所有路由）；FIB：硬件路由表（只保存筛选后的最优路由）
- 核心转发规则：**最长前缀匹配**
- 路由记录：
  - 目的网络 / 掩码：目标网段，例`192.168.2.0/24`
  - 协议：路由来源（直连、static 静态、RIP、OSPF、BGP）
  - 管理距离 AD (优先级)：代表路由来源可信度，数值越小越优先
  - Metric (度量值)：同一协议内部，路径开销，越小越优
  - 下一跳：转发给哪个邻居 IP；直连路由下一跳为`直接‑连接`
  - 出接口：本地路由器转发出去的网卡接口（eth0、G0/0）
- 路由来源：
  - 直连路由：接口配置 IP 后自动生成，AD=0
  - 静态路由：人工手动配置，拓扑变化不会自动更新
  - 动态路由：RIP、OSPF、IS‑IS，同 AS 内自动学习路由；BGP，不同自治系统 AS 之间学习路由
- 最优路由：同一目的网段，优先AD，再比Metric，如果都一样就一起放进FIB做负载均衡
  \#### 路由算法
- **IGP 内部网关协议**：同一个 AS 内部；代表：RIP、OSPF、IS‑IS
- **EGP 外部网关协议**：不同 AS 之间；代表：BGP（路径向量）
- 距离‑矢量算法 DV（代表协议 RIP）：周期向**直连邻居**广播全部路由表，只知道下一跳，最大有效 15 跳，16 跳代表不可达，容易路由环路
- 链路‑状态算法 LS（代表协议 OSPF/IS‑IS）：只泛洪直连链路状态，所有路由器得到同一张全网拓扑图，根据Dijkstra 计算最短路径树，生成路由表
  \#### 路由寻址
- 寻址全流程中ip是端到端全程不变的（除非nat地址转换），mac地址是每一跳都会改变的

``` txt
主机A(192.168.1.10)
      ↓(二层目的MAC=网关MAC，IP:A→B)
路由器G1(网关)
      ↓(重写二层MAC，IP不变)
路由器G2
      ↓(重写二层MAC，IP不变)
主机B(10.0.2.20)
```

- 主机 A（发送端）
  1.  用 IP & 掩码运算，判断目标不在同一网段
  2.  IP 头：**源 IP=A，目的 IP=B【全程不变，NAT 除外】**
  3.  二层帧：源 MAC=A 网卡 MAC，**目的 MAC = 网关 G1 的 MAC**
  4.  ARP 无网关 MAC 则广播 ARP 获取网关 MAC，发给网关
- 路由器每一跳处理（G1、G2 逻辑完全相同）
  1.  接收帧：匹配入接口 MAC，剥离二层头部，取出 IP 包
  2.  **路由表：最长前缀匹配，选出下一跳、出接口**
  3.  TTL = TTL‑1；TTL=0 丢弃，回复 ICMP 超时
  4.  ARP 解析**下一跳设备的 MAC 地址**
  5.  **重写二层帧头**
      - 源 MAC：本路由器出接口 MAC
      - 目的 MAC：下一跳 MAC
      - ✅IP 头部源、目的 IP 保持原样
  6.  从出接口转发出去
- 到达主机 B
  1.  网卡接收帧，校验 MAC，上交 IP 层
  2.  IP 校验目的 IP 为本机，剥 IP 头交给 TCP/UDP
      \## 数据链路层
- 7层模型中拆分成物理层、数据链路层
- 负责物理寻址、差错检测（CRC）、介质访问控制、有流量控制、差错控制能力
- 协议：ethernet（以太网）、802.11、令牌环网、PPP、PPPoE
- 冲突域：同一时间，只能有一个设备发送数据；多台同时发就会产生信号冲突的网络范围，交换机可以分割
- 广播域：广播帧能够到达的全部设备范围，VLAN可以分割
- 局域网LAN：小范围网络，广播域内，主流为以太网802.3，还有令牌环网、WIFI802.11、FDDI、ATM
  \#### 信道划分
- 多个用户共享同一传输信道，信道划分就是把共享信道切分，让多用户互不冲突
- 传输模式
  - 单工：**只能单向传输**，一方只发，一方只收，不能反向
  - 半双工：双方都可以收发，**但是不能同时收发**
  - 全双工：收发完全独立，可以同时发送和接收
    \##### 静态信道划分（预先分配好）
- FDM 频分复用：按频率划分，不同用户占用不同频段，例如有线电视
- TDM 时分复用：按时间划分，划分等长时隙
- STDM 统计时分复用：改进版TDM；有数据才分配时隙，不空闲浪费
- WDM 波分复用：光纤，不同波长（光频率）；本质就是光域的FDM
- CDM/CDMA 码分复用：每个用户分配唯一正交码片序列；所有用户同时使用全部频率、全部时间，接收端用对应码片解码，提取自己信号，例如手机移动通信
  \##### 动态信道划分（按需竞争）
- ALOHA：
  - 纯ALOHA：有帧直接发送；冲突就超时重传；冲突概率高，效率低
  - 时隙ALOHA：时间分成时隙，只能时隙起点发送；降低冲突，效率提升
- CSMA 系列（载波监听）：
  - CSMA：发送前先监听信道；信道空闲发送，忙就等待；依然会冲突（传播时延带来冲突）
  - CSMA/CD：载波监听+冲突检测；**半双工以太网(Hub)使用**。发送过程持续检测冲突；一旦检测冲突，立刻停止发送，发送干扰信号，随机退避重传。（全双工交换机以太网**不用CSMA/CD**）
  - CSMA/CA：载波监听 + 冲突避免；**无线局域网 802.11 WiFi 使用**。无线不能边发边检测冲突；用 RTS/CTS、退避机制减少冲突概率。
- 轮询访问：
  - 轮询：主节点依次询问各个从节点，轮到才允许发送；无冲突；适合总线、工业链路
  - 令牌传递（令牌环网）：拿到令牌才有权发送，发完把令牌传递下一站；无冲突；旧技术现已淘汰
    \### mac地址
- 硬件地址，出厂固化在网卡 ROM 芯片，每块网卡每个路由器都有一个唯一的MAC地址
- 格式：`AA‑BB‑CC‑DD‑EE‑FF`或`AA:BB:CC:DD:EE:FF`，48bit/6字节；前24bit厂商OUI，后24bit设备编号
- MAC只在同一个广播域生效，路由器剥离二层帧，MAC不会传到另一个局域网，跨网段一般只需要知道网关的mac就行了
- [arp](6.2%20计算机网络.md#arp)协议可以将ip转换为mac地址
  \### 流量控制
- 停止‑等待 Stop‑and‑Wait：发送 1 帧 → 停止，等待接收方 ACK 确认；收到 ACK 再发下一帧。出错 / 超时就重传当前帧
- 后退 N 帧协议 GBN（Go‑Back‑N，滑动窗口）：发送方维持发送窗口，可连续发送多帧，ACK (n) 代表 n 号帧及之前全部正确收到，丢弃同窗口所有帧，出错窗口内全部帧重传
- 选择重传 SR（Selective Repeat）：只重传出错的那一个帧，正确到达的帧先缓存，不丢弃，接收方需要缓存乱序帧实现复杂
  \### 差错控制
- 差错来源：
  - 位错：噪声干扰，个别比特0↔1翻转
  - 帧丢失：帧完全没到达
  - 帧重复：ACK丢失
  - 帧失序：帧到达顺序乱掉
- 检错：
  - 奇偶校验：
    - 发送端：
      - 统计原始数据比特中 `1` 的总个数
      - 根据校验模式生成 1 位校验位
        - 偶校验：若 1 的个数为偶数 → 校验位 = 0；若奇数 → 校验位 = 1，保证整体 1 总数是偶数
        - 奇校验：若 1 的个数为奇数 → 校验位 = 0；若偶数 → 校验位 = 1，保证整体 1 总数是奇数
      - 将【原始数据 + 校验位】一起发送
    - 接收端：
      - 收到：数据比特 + 校验位
      - 统计全部比特（数据 + 校验位）里面`1`的数量
      - 和约定模式比对
        - 偶校验：1 总数不为偶数 → 判断传输出错
        - 奇校验：1 总数不为奇数 → 判断传输出错
  - CRC循环校验：
    - 发送端：
      - 原始数据比特 M；生成多项式 G，位数 r+1，则 FCS 长度 = **r 位**。
      - M 后面补 r 个 0。
      - 将补零后的数据，和 G 做**模 2 除法**，得到余数，余数就是 FCS（余数不足 r 位前面补 0）。
      - 将原来 M 去掉补的 0，拼接 FCS，构成最终发送帧：`原始数据 + FCS`。
    - 接收端：
      - 收到完整帧：`原始数据 + FCS`。
      - 使用同样生成多项式 G 做模 2 除法。
      - 余数=0：判定传输无差错
      - 余数≠0：判定出错，直接丢弃帧
- 纠错：
  - 海明码 Hamming：
    - 发送端：
      - 根据数据位数 D，求出满足 $2^r \ge D+r+1$ 的最小校验位数目 r。
      - 将 r 个校验位摆放在编号为 $2^0,2^1,2^2…$ 的位置；剩下位置依次填入原始数据。
      - 每一个校验位，负责一组编号二进制第 i 位为 1 的位置，做**偶校验（常用）**。
      - 算出全部校验位的值，完整海明码比特串发送出去。
    - 接收端：
      - 接收完整海明码比特串。
      - 分别对每一组校验覆盖的比特重新做偶校验，得到 $S\_1\,S\_2\,S\_3$ 校正因子。
      - 把 $S\_3S\_2S\_1$拼接成二进制数，这个数值就是出错位的位置编号。
      - 纠正完成后，剔除全部校验位，恢复原始数据。
- ARQ 自动重传：检错发现出错，依靠 ARQ 重传来实现差错恢复，同时兼具流量控制
  \### ppp
- 是点对点协议，不是局域网协议，把 PPP 封装在以太网帧里面就变成了PPPoE
- 只有两端两个设备，所以不需要mac地址
  \#### 子协议
- LCP 链路控制协议：建立、配置、测试链路；协商参数，链路断开
- NCP 网络控制协议：协商网络层参数；最常用 IPCP，协商双方 IP 地址
  \#### ppp帧
- Flag 标志字段（1 字节）：固定值：`0x7E`，标记 PPP 帧的开始与结束
- Address 地址字段（1 字节）：固定值：`0xFF`，逻辑广播地址，只有两个设备没意义
- Control 控制字段（1 字节）：固定值：`0x03`，代表无编号 UI 帧，默认不使用序号、确认
- Protocol 协议字段（2 字节）：封装的协议
  - `0x0021`：信息字段是**IP 数据报**
  - `0xC021`：信息字段是**LCP 链路控制报文**
  - `0x8021`：信息字段是**IPCP 网络控制报文（NCP）**
- FCS 帧校验序列（2 字节，CRC‑16；可协商为 4 字节 CRC‑32）：CRC 检错，可选择开启 ARQ 重传机制
- Flag 标志字段（1 字节）：同样`0x7E`，帧结束标记
  ![](../computer-network/ba41f90a656a9c4b7db6735d1700ef57356c7fc6.png)
  \#### 工作流程

1.  链路建立：LCP 协商链路参数（最大帧长、认证方式）
2.  认证阶段（可选）：PAP 明文认证 / CHAP 密文挑战认证
3.  网络层配置：NCP (IPCP) 分配 IP 地址
4.  数据传输
5.  链路终止：LCP 断开链路
    \### ethernet以太网

- MTU：链路层一次能发送的最大帧的数据部分大小，单位字节，以太网默认1500
- 使用CRC-32检错、CSMA/CD，没有什么重传和流量控制策略，所以是不可靠的
- 拓扑：早期总线型，现在星型
- 速率：10M/100M/1G/100G
  \#### CSMA/CD
- 半双工总线以太网使用，全双工交换机链路不再需要
- 发送前先听总线，信道空闲才允许发
- 发送数据同时持续监听线路，遇冲突，停发，向上发阻塞信号
- 执行**二进制指数退避算法**随机等待一段时间后重试
- 二进制指数退避：冲突 k 次，随机取 0 \~ $2^k-1$之间的数作为等待时隙；冲突次数越大，等待范围越大，减少再次冲突概率。最多重试 16 次，失败丢弃帧
  \#### 以太网帧（\>64B）
- 物理层
  - 前导码 Preamble（7B）：0xAA，10101010 循环，收发时钟同步
  - SFD 帧起始定界符（1B）：0xAB，标记**真正 MAC 帧开始**
- DMAC 目的 MAC（6B）：接收方 48 位 MAC
- SMAC 源 MAC（6B）：发送方 48 位 MAC 地址
- VLAN标签（4B）：可选，trunk链路
  - TPID(2B): `0x8100`，代表这是 VLAN 帧
  - TCI (2B): 3bit 优先级 + 1bit CFI + **12bit VID(VLAN ID，1‑4094)**
- Type（2B）：上层协议标识
  - `0x0800` IPv4
  - `0x0806` ARP
  - `0x86DD` IPv6
- Payload 载荷 + PAD 填充（46-1500B）：上层 IP/ARP 报文；不足 46 字节自动 PAD 填充，保证帧最小长度
- FCS 帧校验序列（4B）：CRC32 校验，校验 DMAC\~Payload 全部内容
  ![](../computer-network/3e533537ff6f42f546e82570818695b37f7caeaf.png)
  \#### VLAN虚拟局域网
- 在同一台物理交换机上，逻辑划分多个广播域，一个 VLAN = 一个独立广播域，帧、ARP 报文只能在本 VLAN 内部转发
- 不同 VLAN 之间二层不能直接通信，**必须经过三层设备（路由器 / 三层交换机）做 VLAN 间路由**
- 核心标签：802.1Q，在以太网帧头部插入**4 字节 VLAN 标签（Tag）**，里面有12位的VLAN ID（1-4094）
- 端口类型：
  - Access 接入端口：属于**1 个 VLAN**，收发的帧**不带 VLAN 标签（untagged）**，直接对接端系统
  - Trunk 中继端口：用于交换机‑交换机、交换机‑路由器之间互连，必须带VLAN标签通行
- 搭建架构：
  - 单臂路由器：1 台普通路由器 + 1 台二层交换机，**1 个物理网口**连接交换机的 Trunk 口
    - 路由器：真实物理接口不配置 IP，在这个物理口下创建多个**逻辑子接口**配置vlan网关ip
    - 交换机：允许不同vlan网关带标签通过
  - 三层交换机：本身带三层模块，给每个vlan创建SVI虚接口（vlan网关ip），直接在内部完成二层到三层的路由
    \#### LLC
- IEEE802.3 原始帧，2 字节是 Length，不是 Type，配合 LLC 头
- 现在 PC、互联网全部使用 Ethernet‑II，LLC 多见于工业老设备
  \### arp
- 从IP 地址解析得到 MAC 地址
- arp协议封装在以太网帧内，但是从功能上来说应该属于网络层
  \#### arp表
- Linux命令`ip - neigh`查看arp缓存
- IP地址：下一跳设备的IP（可以是主机、网关）
- MAC地址：该IP对应的48位硬件MAC
- 接口：本机网卡，这条ARP条目从哪个网口学到
- 类型：条目来源
  - 动态 (dynamic)：收到 ARP 应答自动学习，**有老化时间**，超时删除
  - 静态 (static)：人工手动配置，永久有效，不会老化
- 老化时间/生存时间：动态条目倒计时，超时失效，不同系统10‑20分钟
  \#### arp报文
- 硬件类型：2 字节，值为 1：代表以太网
- 协议类型：2 字节，值为0x0800：代表 IPv4
- 硬件地址长度 (HLEN)：1 字节，MAC 地址长度
- 协议地址长度 (PLEN)：1 字节，IPv4 地址长度
- 操作码 OP：2 字节
  - 1：ARP 请求
  - 2：ARP 应答
- 发送方硬件 MAC 地址：6 字节，本机 MAC
- 发送方 IP 地址：4 字节，本机 IPv4
- 目标硬件 MAC 地址：6 字节
  - ARP 请求：填充全 0，未知对端 MAC
  - ARP 应答：填入请求方的 MAC 地址
- 目标 IP 地址：4 字节，待解析的目标 IPv4
  ![](../computer-network/37efa2277dad7bd6d5fe3bb800af907303fa213c.png)
  \#### arp寻址
- 发送端：
  - 主机 A 根据下一跳IP（路由选择）需要知道下一跳的 MAC
  - 查询本地arp表
  - 缓存**没有该条目**：发送 ARP 广播请求
  - 等待 ARP 应答。收到应答后，把`目标IP‑MAC`存入 ARP 缓存，设置老化时间
  - 使用得到的 MAC，封装以太网帧发送 IP 报文
- 接收端：
  - 收到 ARP 请求，对比报文中的**目标 IP**和自己本机 IP
  - IP 不匹配：直接丢弃 ARP 请求报文，不回复
  - IP 匹配本机 IP：构造 ARP 应答，**单播回复请求主机**，同时把请求方的 IP‑MAC 更新进自己 ARP 缓存
    \#### arp攻击
- 主机发送伪造的 ARP 应答，篡改局域网内主机的 ARP 缓存表，实现中间人窃听或者断网
  \# 网络设备
  \|设备\|冲突域\|广播域\|工作层\|
  \|---\|---\|---\|---\|
  \|Hub 集线器\|全部端口同一冲突域\|全部同一广播域\|物理层\|
  \|交换机 (无 VLAN)\|每个端口独立冲突域\|全部同一广播域\|数据链路层\|
  \|交换机 + VLAN\|每个端口独立冲突域\|每个 VLAN 独立广播域\|数据链路层\|
  \|路由器\|每个接口独立冲突域\|每个接口独立广播域\|网络层\|
  \## 网卡
- 工作在数据链路层（物理层-\>数据链路层）
- 硬件：
  - MAC 芯片（数据链路层）：组装解析以太网帧、FCS 校验、硬件帧过滤、管理 DMA 描述符环，保存 MAC 地址
  - PHY 芯片（物理层）：数字信号↔网线电信号转换；**自动协商速率 / 双工；检测链路 UP/DOWN**
  - PCIe：网卡与主机内存总线
- 模式：
  - 默认模式：只收本机 MAC、广播、订阅组播帧，其余硬件丢弃。
  - **混杂模式 promisc**：接收所有帧，Wireshark 抓包需要；仅镜像口才能抓到其他主机流量。
  - 监控模式：WiFi 网卡专用。
- 全流程：
  - 接收：PHY→MAC 做 FCS 校验→硬件过滤→**DMA 直接写入主机内存（不耗 CPU 拷贝）** →==硬件中断==
  - 发送：内核缓冲区 → DMA 搬运到网卡 → MAC 加前导码 / SFD/FCS → PHY 发出
    \## 集线器（半双工）
- 工作在数据链路层（物理层）
- 共享总线
- 所有端口同一个冲突域，同一个广播域
  \## 交换机（全双工）
- 工作在数据链路层（物理层-\>数据链路层）
- 每个端口独立冲突域；所有端口默认同一个广播域
- 计算机如果有多块网卡也可以当交换机用
- **MAC 地址表 (MAC‑Address‑Table)**：`MAC地址 + 端口 + VLAN`
- 工作流程：
  - 收到帧，记录源 MAC 和入端口到 MAC 表
  - 查询目的 MAC：
    - 表中有该 MAC：**单播转发**，从对应端口发出
    - 表中无该 MAC：**泛洪 (Flood)**，向除入端口外所有端口转发
    - 目的 MAC 是广播 MAC：直接泛洪
  - MAC 表条目老化：默认 300s 超时删除
    \## 路由器（全双工）
- 工作在网络层（物理层-\>数据链路层-\>网络层）
- 主要是[路由选择](6.2%20计算机网络.md#路由选择)功能
- 内网路由器还有[NAT地址转换](6.2%20计算机网络.md#NAT地址转换)功能，不仅能转换ip，还能做端口映射哦
- 包过滤：路由器收到 IP 报文，**根据报文头部信息（ip头、tcp/udp头）做匹配，允许通过 or 丢弃**，可以设置出入方向，只看单个数据包，不跟踪完整连接状态
  - IP 层：源 IP、目的 IP、IP 协议号
  - 传输层：源端口、目的端口
  - 其他：TCP 标志位 (SYN、ACK 等)
- 内置交换机完成数据链路层功能
  \# 互联网
- 全层级：端系统 → 接入网 → 城域网 → 骨干网
  - 接入网：光猫 ONU、OLT 分光器、5G 基站、路由器、FTTH 光纤、PON、PPPoE 拨号、ISDN、专线、ADSL、CATV
  - 城域网（城市内部）：BAS/BRAS、汇聚路由器、CGNAT
  - 骨干网（跨省市）：高端核心路由器、100G/400G 高速光纤
    ![](../computer-network/276ebc28a2ed5a41f5ff2cc9f1d54c50bc8ca5b3.png)
- 运营商isp：互联网服务提供商，分三级，骨干运营商、区域运营商、本地接入商，电信、联通、移动都是
- ixp（ix）：互联网顶层互联基础设施，**不是运营商，不是用户接入网**，是多个自治网络汇聚交换的中立机房枢纽
- cdn内容分发网络：就近缓存静态资源，主要使用遍布各地的POP节点服务，用户 DNS 请求→CNAME 指向 CDN 调度→选择就近边缘节点→节点有缓存直接返回
- **内容提供者**：大型网站、业务源站 IDC，会直接接入骨干 / IXP，很多同时部署 CDN
  \# WEB安全
- 这里的分类和具体攻击手段说的比较奇怪垃圾，具体需要参考网络安全篇
  \## 主动攻击
- SQL注入
- 命令注入
- 邮件首部注入
- HTTP首部注入
- 目录遍历、目录上传
- 远程文件包含
  \## 被动攻击
- 内网渗透
- XSS跨站脚本
- 后门
  \## 其他漏洞
- 强制浏览
- 错误消息提示过多
- 开放重定向
- 会话劫持
- 跨站请求伪造
- 密码破解
- DDos攻击
  \## 防御
- 客户端验证
- 服务端验证
  \# 附录
  \## HTTP元数据字段
  \### 通用头
- **Cache‑Control**：缓存控制指令，控制缓存行为（max‑age、no‑cache 等）
- **Connection**：控制连接，keep‑alive 保持长连接、close 关闭连接
- **Date**：报文构建的 GMT 标准时间
- **Transfer‑Encoding**：传输编码，典型`chunked`分块传输，改变报文传输形式，不改变资源本身
- **Trailer**：分块传输模式，声明报文末尾会携带的头部字段
- **Upgrade**：请求升级至其他协议，例如 HTTP 升级 WebSocket
- **Via**：记录报文经过的代理网关链路信息
- **Warning**：携带错误警告信息
  \### 实体头（也是通用的）
- **Content‑Type**：报文体 MIME 数据类型，标记 body 格式稀土掘金
- **Content‑Length**：报文体字节总长度，chunked 分块传输时不出现稀土掘金
- **Content‑Encoding**：内容编码，资源本体压缩算法（gzip/br）稀土掘金
- **Content‑Range**：206 范围响应，标记返回片段在完整资源中的字节位置
- **Content‑Disposition**：控制浏览器处理 body，inline 展示 /attachment 附件下载
- **Last‑Modified**：资源最后修改时间，用于条件缓存请求
- **Expires**：资源缓存过期绝对时间
- **Allow**：告知该资源支持哪些 HTTP 请求方法
- **Content‑Language**：资源内容的自然语言
  \### 请求头
- **Host**：指定目标服务器域名与端口，HTTP/1.1 必须携带CSDN博...
- **User‑Agent**：标识客户端浏览器、程序、设备信息CSDN博...
- **Accept**：声明客户端能够接收的响应 MIME 数据类型
- **Accept‑Encoding**：客户端支持的内容压缩算法（gzip、br 等）
- **Accept‑Language**：客户端偏好的自然语言
- **Cookie**：携带服务端下发的会话 Cookie 数据CSDN博...
- **Referer**：标识请求来源页面地址CSDN博...
- **Origin**：跨域请求来源，只包含协议 + 域名端口，不含路径CSDN博...
- **Authorization**：携带身份认证凭证（Basic、Bearer）
- **If‑Modified‑Since**：条件请求，资源未更新返回 304，使用资源修改时间判断
- **If‑None‑Match**：条件请求，比对 ETag，资源未变更返回 304
- **If‑Match**：条件请求，ETag 匹配才执行操作，用于并发防护
- **If‑Range**：结合缓存校验的范围请求头
- **Range**：范围请求，指定想要获取文件的字节区间
- **Expect**：告知服务器期望的请求行为，典型`100‑continue`
- **Proxy‑Authorization**：代理服务器身份认证信息
- **From**：客户端用户邮箱，较少使用
  \### 响应头
- **Server**：服务器软件名称版本，建议隐藏避免泄露信息稀土掘金
- **Location**：重定向目标地址，配合 3xx 状态码使用稀土掘金
- **Set‑Cookie**：下发 Cookie 给客户端，可携带过期、域、安全属性
- **ETag**：资源实体标记，资源唯一标识，用于缓存校验
- **Accept‑Ranges**：告知客户端服务器是否支持字节范围请求
- **Age**：资源在代理缓存中存放的时长
- **Retry‑After**：服务不可用时，建议客户端多久之后重试
- **WWW‑Authenticate**：401 响应，告知客户端需要使用何种认证方式
- **Proxy‑Authenticate**：代理服务器认证质询
- **Vary**：指定哪些请求头会影响缓存的版本
- **X‑Frame‑Options**：安全头，控制页面是否允许被 iframe 嵌套，防点击劫持稀土掘金
- **X‑Content‑Type‑Options**：禁止浏览器 MIME 嗅探，强制遵循 Content‑Type
