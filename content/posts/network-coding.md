---
title: "网络编程"
slug: "network-coding"
date: 2026-08-26T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["计算机网络", "c++", "网络编程"]
categories: ["技术笔记"]
summary: "网络编程相关知识点，包括网络IO模型、socket编程、多线程、事件驱动模型等。"
toc: true
comments: true
description: "网络编程"
---
# 网络io模型

- IO的两个阶段：
  - 等待数据接收完成
  - 数据从内核接收缓冲区拷贝到进程缓冲区
- 同步 IO：**用户线程参与 IO 拷贝过程**
- 异步 IO：**内核完成全部 IO 工作，用户线程只接收完成通知**
- 关键socket：
  - 监听socket：bind+listen生成，只用来接收连接，不执行数据收发
  - 连接socket：accept生成，专门执行数据收发

| IO 模型 | 同步 / 异步 | 关键区别 | 单线程管理多client |
|----|----|----|----|
| 阻塞 IO（blocking IO） | 同步 IO | 线程挂起等待；实现最简单；每一个连接占用一个线程 | 不能 |
| 非阻塞 IO（non‑blocking IO） | 同步 IO | 线程不阻塞，但 CPU 消耗高；单纯轮询不适合大量连接 | 可以（轮询） |
| IO 多路复用（select/poll/epoll） | 同步 IO | 单线程管理成千上万个连接；**读写本身依旧阻塞**；网络编程主流 | 可以（主流） |
| 信号驱动 IO（signal driven IO） | 同步 IO | 无需轮询；信号机制限制多，工程几乎不用 | 理论可以（信号机制缺陷多） |
| 异步 IO（AIO） | 异步 IO | 用户线程完全不参与 IO 拷贝；真正异步；Linux 原生 AIO 使用场景有限 | 可以 |

## 阻塞 IO

![](../network-coding/fb8758cbd1a5b81ceec0cd013510ff71875d86a8.png)
\## 非阻塞 IO
![](../network-coding/eb202dc8aad859c923caa1a2645334156ddf765c.png)

## IO 多路复用（主流）

![](../network-coding/3bf4410d5d648aae0e08232f5c73ca2de50b7b24.png)
\## 信号驱动 IO
![](../network-coding/b7b10f94a631ef933e48b83bb38eef7355b6d36d.png)
\## 异步 IO
![](../network-coding/2b0c532101c6fafd27ed46c78509dd8d1be571ba.png)
\## 多线程（事件驱动模型）
- 网络io模型与多线程经常搭配使用，不同 IO 模型和线程的组合方案不一样
\### 经典BIO：阻塞IO+多线程
- 连接量大，空闲连接占住线程大量阻塞，内存、上下文切换开销爆炸，不适合高并发
- 每次建立一个connection，服务器就要单独开一个线程完成数据收发任务
- 进一步优化：服务器建立一个线程池进行复用
![](../network-coding/f4b2de5694034a6268cfd5e664f16476d969566d.png)
\### Reactor：非阻塞 IO / IO 多路复用+多线程（主流）
- Reactor：事件监听器
- Acceptor：连接器
- Handler：业务处理器
- 应用：Netty、libevent、Nginx
\#### 单Reactor-单线程
- 全流程：一个主线程完成监听、连接、业务
- 缺陷：业务一旦阻塞，整个接收全部阻塞并发程度低，且无法利用多核
![](../network-coding/70094b1fbc4aa003b6e14c85eadce6f148fbc56f.png)
\#### 单 Reactor‑多线程
- 全流程：
- 主线程完成监听、连接
- 耗时业务交给独立 Worker 线程池执行，业务完成后交回 Reactor 线程 write 回写
![](../network-coding/c270d9304a4f540a820a4ea2d979354bc08c26dd.png)
\#### 主从 Reactor‑多线程（Netty）
- 全流程：
- MainReactor：io多路复用建立监听socket，accept 接收 TCP 新连接，把 connect socket 分配给某一个 SubReactor
- SubReactor：独立epoll注册connect socket，epoll_wait监听读写，一个connect socket永久绑定同一个 SubReactor线程，规避多线程锁竞争，简单任务直接在循环内执行，复杂任务交给线程池
- **业务线程池 M（可选）**：DB、RPC 等耗时业务丢线程池；纯内存业务直接在 SubReactor 执行
![](../network-coding/4f0e4fdaffe9a73ade8310e269b2cb68e4497e0d.png)
- 为什么每个 SubReactor（Worker 线程）要有独立 epoll 实例？为了保证一个 socket 的所有 IO 事件，全部由同一个 Worker 线程处理，避免多线程竞争不需要锁，还可以做负载均衡
\### Proactor：异步 IO (AIO) + 多线程
- Linux 原生 socket AIO 弱
- 应用：IOCP（Windows ）；io_uring（Linux）
- IO完成队列：接收内核的IO完成事件
- 全流程：
- 主线程先在内核创建一个IO完成队列
- **主线程**调用`aio_read`，提交异步读请求，传入**用户缓冲区地址 + ==完成回调==**，调用**立刻返回，不阻塞**
- 操作系统内核全权执行整套 IO，把数据拷贝到**用户提前提供好的用户缓冲区**，并把 IO 完成事件投递到**IO 完成队列**
- 线程池一直在完成端口阻塞等待读出，从队列取出事件，执行业务回调，读取数据完成任务
![](../network-coding/bfa8ca58830668c3a8c63ffba3e19db2b39c0531.png)
\# 高性能服务器
\## 服务器类型
- 防火墙（传输层）
- 代理服务器：负载均衡、攻击防护
- 缓存服务器：提速
- 后端服务器：提供主要功能
- 数据服务器：提供数据存储
\## Libevent

## Muduo
