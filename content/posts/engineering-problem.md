---
title: "工程实践"
slug: "engineering-problem"
date: 2026-10-23T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["Linux", "并发编程", "架构设计"]
categories: ["面试八股"]
summary: "聚焦如何面对实际业务设计架构和解决相关问题。"
toc: true
comments: true
description: "工程实践"
---

> 大厂常问的有技术深度和需要一定工程经验才能回答的问题，聚焦如何面对实际业务设计架构和解决相关问题，尤其注意设计降级策略、Kafka前端削峰填谷。
- ⭐⭐⭐⭐⭐：极高频考点，必须掌握
# Linux实战
- ⭐⭐⭐⭐⭐常用命令：
	- top：综合监控，p按cpu排序，m按内存排序
	- uptime：load average负载，越接近 CPU 核数代表压力高
	- vmstat：全局状态，包括CPU、内存、IO、中断、上下文切换
	- mpstat：多核CPU细分数据
	- pidstat：进程级细分数据
	- free：空闲内存
	- ps aux：详细进程内存
	- df -h：磁盘空间大小
	- iostat：看磁盘IO读写、CPU等待IO的时间占比
	- netstat/ss -tulnp：端口、连接
	- tcpdump：抓包
	- ping/traceroute：网络连通性
	- lsof：看进程打开的文件句柄、端口
- ⭐⭐⭐⭐⭐CPU占用过高：top-mpstat-pidstat
- ⭐⭐⭐⭐⭐性能瓶颈定位：perf+火焰图
- ⭐⭐⭐⭐⭐内存泄露：free-vmstat-top/ps
- ⭐⭐⭐⭐⭐IO过高：iostat
- ⭐⭐⭐⭐⭐端口占用：netstat
- ⭐⭐⭐⭐⭐句柄泄露：lsof
# 手搓代码
- 参考computer-science-practice项目的代码
- poll
	- ObjectPool 对象池
	- ThreadPool 线程池
	- ConnectionPool 连接池
- deadlock 死锁埋点
- reactor 主从reactor
- pandc 生产者消费者模型
	- BoundedQueue 有界阻塞队列（MPMC和优雅退出）
	- RingBuffer 无锁环形队列（SPSC）
- randw 读者写者模型
- thread-safe 线程安全代码
	- Singleton 线程安全单例
- coroutine 协程编程
	- Generator 生成器
	- Asynctask 异步任务
# 架构设计
- ⭐⭐⭐⭐⭐怎么实现高并发？怎么设计一个高并发架构？
	- 选用并发模型：
		- 半同步半异步模型：Reactor、Proactor
		- 领导者追随者模型
	- 并发粒度：
		- 进程级：IPC
		- 线程级：std::thread，结合锁（互斥锁、读写锁）、条件变量、原子变量
		- 协程级：用户态c++20，co_wait、co_yield、co_return
	- 架构设计：
		- 主从reactor：
			- 主reactor：负责新建连接（listen socket）
			- 从reactor：负责处理连接读写事件（connection socket），有各自epoll
			- 线程池：不阻塞IO通信，单独处理耗时任务
		- 分布式：
			- 前端负载均衡，服务无状态可水平扩容
			- 热点数据缓存，数据库分库分表，消息队列异步削峰
			- 有降级、限流、熔断策略
	- 性能优化：
		- 原子操作：原子变量内存序、barrier、ABA问题
		- 池化技术：对象池、线程池
		- 伪共享：alignas(64)
		- 分片锁：IO线程与业务线程分离
		- socket优化
- ⭐⭐⭐⭐⭐生产者生产太快，消费者又来不及消费怎么办？
	- 缓冲：有界队列做阻塞队列缓冲，如果内存不够就换成消息队列Kafka落盘（业务批量消费提交）
	- 优化：生产者流量控制
	- 扩容：消费者扩容
	- 兜底：熔断、降级、限流
	- 分布式：扩展节点负载均衡，一致性哈希
- ⭐⭐⭐⭐⭐消息队列的真实应用？
	- 是进线程之间常用的通信手段
	- 线程间无锁队列：
		- subreactor与工作线程池间事件传递
		- 连接跨线程投递（主从之间）
		- 日志异步队列
		- 定时器任务队列
	- Kafka之类的进程间消息队列：
		- 削峰填谷：前端流量直接丢MQ，下游慢慢消费
		- 服务解耦
		- 消息持久化
		- 广播事件
- ⭐⭐⭐⭐怎么实现淘宝的高并发购物车功能？
	- 用Redis作为主存储，MySQL异步兜底，区分登录用户和游客
	- 增删改查全部操作Redis，通过MQ异步同步到MySQL
	- 并发：原子命令
	- 容灾：Redis集群和哨兵
- 量化后端与普通后端区别？
	- 时延要求高：内存池（不能临时动态分配）、无锁结构、少系统调用（状态切换）
	- 强一致性：严禁重试，状态机
	- 单线程：核心线程负责核心业务链，多线程只做日志统计类任务
	- 低频量化（分钟/小时策略）：接近普通后端
- ⭐⭐⭐⭐⭐为什么 wait 要传入 unique_lock 而不是普通 lock_guard？
	- 因为`cv.wait`**需要临时解锁 + 重新上锁**的能力，`lock_guard`不支持手动 unlock/relock
- ⭐⭐⭐怎么定位并降低时延？
	- 定位：埋点追踪确认时延位置（代码、数据库、网络）
	- 代码：无锁、零拷贝、异步化、内存池、冷热分离、二进制替代JSON
	- OS：关闭swap、减少线程数
	- 网络：UDP部分替代TCP、负载均衡
- ⭐⭐⭐数据库时延怎么定位和解决？
	- 系统排查：linux系统排查（tcp、iostat、vmstat、pidstat）
	- 慢查询日志
	- performance_schema底层性能观察库查看等待事件（IO、锁）
	- 原因：
		- 整体架构：主从延迟、读写分离、合理分片
		- SQL慢：索引失效、索引覆盖、避免全表扫描
		- 锁等待：事务锁（行锁、表锁）、内核锁
		- IO瓶颈：redo log刷盘压力
		- 内存瓶颈、CPU瓶颈：缓存命中率低、CPU耗尽
- ⭐⭐⭐⭐⭐工程中怎么检测解决死锁？
	- clang
	- valgrind
	- 埋点：所有锁带编号封装，加锁id顺序校验，维护全局等待图，后台线程定时DFS找环，有环就告警
	- gdb调试
	- 给锁封装设置超时时间
- ⭐⭐⭐⭐⭐工程中怎么检测内存泄露？
	- clang
	- valgrind
	- 埋点：重载new/delete
	- linux命令观测
	- RAII多用、内存池
- ⭐⭐⭐⭐⭐工程中频繁出现的锁粒度问题怎么解决？
	- lock_guard、unique_lock要配合作用域
	- 埋点：检测锁耗时
	- 无锁替代：原子、SPSC队列
	- 数据快照：将所内数据复制到局部变量