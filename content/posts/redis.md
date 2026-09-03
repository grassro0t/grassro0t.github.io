---
title: "数据库基础系列-Redis"
slug: "redis"
date: 2026-09-03T8:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["Redis", "数据库"]
categories: ["技术笔记"]
summary: "Redis基础内容，包括数据结构、对象、事务、持久化、主从复制、集群等。"
toc: true
comments: true
description: "Redis"
---

- 本质：驻留在内存的数据库，一个字典数组，每个字典对应一个数据库
- 列表键：数据库键对应的值为列表对象
- TYPE命令返回键对应的值的类型
# 数据结构与对象
## 数据结构
### 简单动态字符串 SDS
- 应用：字符串、缓冲区（AOF缓冲区、输入缓冲区）
- 长度不计算'\0'，多分配1字节空间给'\0'，自动尾部加'\0'
- 二进制安全：依靠 len 判断边界，中间可以存`\0`
- 与C字符串的区别：
  - 长度信息有记录，O1级
  - 空间预分配：修改后总长度 \< 1MB：**双倍预分配**，修改后总长度 ≥ 1MB：**最多多分配 1MB 空闲**
  - 惰性释放：缩容时只修改空闲字节数，只有调用sdstrim才进行真正的内存释放

  ``` cpp
  struct sdshdr {
    int len;    // buf已使用字节数，不包含'\0'
    int free;   // buf剩余空闲字节
    char buf[]; // 字节数组，末尾主动追加'\0'兼容C函数
  };
  ```

  ### 双向链表 list
- 应用：列表键、发布订阅、慢查询、监视器、客户端状态信息、输出缓冲区
- 特性：双向、无头节点、头尾插入O1、长度缓存、支持多态（节点值void\*）、节点分散在堆上
- 链表

``` cpp
typedef struct list {
    listNode *head;    // 头结点
    listNode *tail;    // 尾结点
    void *(*dup)(void *ptr);  // 复制函数
    void (*free)(void *ptr);  // 释放函数
    int (*match)(void *ptr,void *key); // 对比匹配函数
    unsigned long len; // 结点总数，O(1)获取链表长度
} list;
```

- 链表节点

``` cpp
typedef struct listNode {
    struct listNode *prev;   // 前驱结点
    struct listNode *next;   // 后继结点
    void *value;             // 数据指针，可存SDS、各种对象
} listNode;
```

### 字典 dict（哈希表）

- 应用：数据库、哈希键的底层实现
- size 一定是 2 的整数次幂
- 时间复杂度：查找、插入、删除：平均 O (1)；极端哈希冲突最坏 O (n)
- 哈希寻址：`index = hash(key) & sizemask`，MurmurHash2哈希算法
- 哈希冲突：多个 key 落到同一个桶，entry 通过`next`连成单向链表，旧版本链表头插；新版本改为尾插
- 负载因子 load‑factor：load_factor = ht\[0\].used / ht\[0\].size
- 渐进式rehash：
  - 触发扩容缩容：
    - load_factor ≥ 1（执行 BGSAVE/BGREWRITEAOF 时因为写时复制提升到≥5），触发扩容，h\[1\]分配第一个大于等于 used\*2 的 $2^n$
    - load_factor \< 0.1，触发缩容，h\[1\]分配第一个大于等于 used 的 $2^n$
  - 键值对rehash到ht\[1\]上：
    - 计数器变量rehashidx=0
    - 每次增删改查命令，顺带迁移`rehashidx`指向的那一整个桶，rehashidx++
    - 查找时会在ht\[0\]和ht\[1\]依次查找，新增数据一律写到 ht \[1\]，ht \[0\] 只删不增
    - 迁移完成后，`rehashidx=-1`结束 rehash
  - ht\[0\]清空释放，ht\[1\]变成ht\[0\]，ht\[1\]创建空白hash表
- 哈希表

``` cpp
typedef struct dictht {
    dictEntry **table;      // 哈希桶数组，每个元素是链表头指针
    unsigned long size;     // 桶数组总大小，2的N次方
    unsigned long sizemask; // size‑1，用来做位运算取模 hash & sizemask
    unsigned long used;     // 当前存储元素总数量
} dictht;
```

- 哈希表节点

``` cpp
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next; // 冲突链表，把同一哈希桶的结点串起来
} dictEntry;
```

- 字典

``` cpp
typedef struct dict {
    dictType *type;         // 一组回调函数：hash、dup、free、compare
    void *privdata;
    dictht ht[2];           // 两个哈希表 ht[0]主表，ht[1]用于rehash
    long rehashidx;         // rehash标记；-1=不在rehash；>=0表示正在迁移的桶下标
} dict;
```

- 函数回调结构体（实现泛型字典）

``` cpp
typedef struct dictType {
    // 计算key的哈希值
    unsigned int (*hashFunction)(const void *key);

    // 复制key，dup = duplicate
    void *(*keyDup)(void *privdata, const void *key);

    // 复制value
    void *(*valDup)(void *privdata, const void *obj);

    // 比较两个key是否相等
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);

    // 释放key内存
    void (*keyDestructor)(void *privdata, void *key);

    // 释放value内存
    void (*valDestructor)(void *privdata, void *obj);
} dictType;
```

### 跳跃表 zskiplist

- 应用：有序集合键的底层实现
- Redis 跳表**不是双向完整跳表**：高层没有 backward，仅 0 层有后退指针
- 性能接近平衡树，跳表擅长排序、范围遍历
- zrank：返回有序集合中某个 member 的排名，从 0 开始计数，常用于排行榜之类的
- Redis 最大允许 32 层； 每新建结点，有 1/2 概率升到上一层；随机生成，不需要平衡调整
- 时间复杂度：查找、插入、删除：平均 O (logN)，最坏 O (N)，范围查询**O(logN + M)**，M 为返回元素数量，远优于平衡树，获取排名O (logN)
  ![](../redis/f0d3905e6b909fbb97c1dd4ac7c4acef0531b539.png)
- 有序集合
  - 冗余存储，查询速度O1，跳表保证有序

  ``` cpp
  typedef struct zset {
    zskiplist *zsl;
    dict *dict;    // key=member元素，value=score分值
  } zset;
  ```
- 跳跃表

``` cpp
typedef struct zskiplist {
    struct zskiplistNode *header; // 头哨兵结点，不存真实数据，层数最高
    struct zskiplistNode *tail;   // 尾结点指针
    unsigned long length;         // 真实结点数量（不含header）
    int maxLevel;                 // 当前表最大有效层数
} zskiplist;
```

- 跳跃表节点
  - 表头也有ele、score、backward，只不过没用到而已
  - span 不是指针，是跳过结点计数，用来计算排名 zrank
  - score允许重复，按ele字典序排序，ele不允许重复

  ``` cpp
  typedef struct zskiplistNode {
    sds ele;                     // 数据元素，字符串
    double score;                // 排序分值
    struct zskiplistNode *backward; // 后退指针，仅第0层有，用于倒序遍历

    // 层级数组，每个元素指向同层下一个结点
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned long span;      // 当前forward跳过多少个结点（用于rank排名计算）
    } level[];
  } zskiplistNode;
  ```
- 查找流程：
  - 从跳表的 `header` 哨兵结点、**当前最大层 maxLevel 开始**，从高层往下搜索
  - 在当前层，不断看 `forward` 指针：
    - 如果 `forward != NULL` 且 `forward->score < 目标score`：向右走，继续本层前进；
    - 如果 `forward != NULL` 且 `forward->score == 目标score`，继续比较 member 字符串，相等即找到结点；
    - 不满足条件：**下降一层**，停留在当前结点，到下一层继续循环
  - 一直降到第 0 层结束，第 0 层再判断 forward 结点是否匹配 score+member
  - 搜索过程中沿途累加每一步的 `span` 值，最终累加总和就是 rank 排名
- 插入流程：
  - 遍历查找，记录每层前驱结点
    - 从 header、maxLevel 高层向下遍历
    - update 数组保存 0～maxLevel 每一层的前驱结点，`update[i]` 保存**第 i 层待插入位置的前驱结点**，新结点要插在 update \[i\] 后面
    - 累加 span 得到 rank 值
    - 向下逐层搜索，条件和查找逻辑一致；当不能继续向右，就记录当前结点到 update \[i\]，然后降层
  - 随机生成新结点层数 level
  - 创建新 zskiplistNode 结点
  - 逐层修改链表指针：
    - 新结点.level \[i\].forward = update \[i\]-\>level \[i\].forward
    - update \[i\]-\>level \[i\].forward = 新结点
    - 修正 span 跨度
  - 处理 backward 后退指针：
    - backward = update \[0\]
    - 如果新结点 level \[0\].forward 不为空：后继结点的 backward 指向新结点
    - 如果新结点是尾部结点，更新 zskiplist-\>tail 指向新结点
  - 同步更新 dict 字典（在集合中时）
  - 跳表长度 + 1
- 删除流程：
  1.  和插入第一步一样，搜索得到每层前驱结点`update[]`。
  2.  找到目标 node。
  3.  遍历该结点所有有效层，把前驱结点的 forward 跳过本结点，修正 span。
  4.  处理 0 层 backward 指针，维护 tail 指针。
  5.  更新 zskiplist-\>maxLevel（删除最高层结点后 maxLevel 可能下降）。
  6.  **dict 同步删除对应 key**（在集合中时）
  7.  length--，释放结点内存。
      \### 整数集合 intset
- 只存整数，有序、无重复，连续数组存储
- 应用：有序集合键的底层实现
- 整数集合，数组内元素有序升序排列、不重复
- 编码类型：
  - `INTSET_ENC_INT16`：每个元素 int16_t
  - `INTSET_ENC_INT32`：每个元素 int32_t
  - `INTSET_ENC_INT64`：每个元素 int64_t
- contents：字节数组，需要根据编码解析元素

``` cpp
typedef struct intset {
    uint32_t encoding;   // 编码类型：INTSET_ENC_INT16 / INT32 / INT64
    uint32_t length;      // 元素个数
    int8_t contents[];    // 柔性数组，实际存整数，按对应编码解析
} intset;
```

- 升级：存入超过编码范围的数字
  - 重新分配内存，contents数组扩容
  - 旧元素转成 64 位整数拷贝到新数组（全拷贝）
  - 释放旧内存，修改`encoding = INTSET_ENC_INT64`
  - 插入新数字，二分查找
  - 只支持**向上升级，不支持降级**，就算把大数字删掉，编码也不会回退
    \### 压缩列表 ziplist
- 应用：列表键、哈希键
- ziplist结构
  - `zlbytes`：整个 ziplist 总字节数
  - `zltail`：**最后一个 entry 距离起始地址的偏移**
  - `zllen`：元素个数
  - `zlend`：结束标记固定 0xFF

  ``` txt
  zlbytes(4B) ｜ zltail(4B) ｜ zllen(2B) ｜ entry1 ｜ entry2 ｜ ... ｜ zlend(1B=0xFF)
  ```
- entry结构
  - **prevlen**：前一个 entry 占用多少字节
    - 0\~253：1 字节存储
    - ≥254：5 字节，第 1 字节固定 0xFE，后面 4 字节存真实长度
  - **encoding**：标记 content 是整数 / 字符串，以及长度
  - **content**：真正存的数据

  ``` txt
  prevlen ｜ encoding ｜ content
  ```
- 连锁更新
  - 一个 entry 的长度由小于 254 字节变成≥254 字节 → prevlen 要从 1 字节扩张为 5 字节
  - 当前 entry 变大 4 字节，**会导致下一个 entry 的 prevlen 也必须扩容**，连锁向后传播
  - 删除同理也可能导致连锁更新
- listpack结构
  - 没有zltail尾部偏移

  ``` txt
  lpbytes(4B) ｜ lplen(2B) ｜ entry1 ｜ entry2 ... ｜ lpend(1B=0xFF)
  ```
- entry结构
  - `encoding‑len`：编码
  - `data`：实际数据
  - `backlen`：**当前 entry 自己的字节总大小**，与其他entry解耦

  ``` txt
  encoding‑len ｜ data ｜ backlen
  ```

  ## 对外对象 RedisObject
- 基于以上数据结构创建了对外对象
- 一个对象底层可能有多种不同的数据结构实现
- 每个Redis键值对都对应两个对象，键一般是字符串，值则是5种中的一种
- 所有编码转换都是单向升级，数据变小不会自动切回更省内存的编码
- 对象结构体
  - **type(4bit)**：对外暴露的类型，`type key`命令读取
    - OBJ_STRING：0
    - OBJ_LIST：1
    - OBJ_HASH：2
    - OBJ_SET：3
    - OBJ_ZSET：4
  - **encoding(4bit)**：底层真实存储实现，`object encoding key`查看
    - OBJ_STRING：OBJ_ENC_INT / OBJ_ENC_EMBSTR / OBJ_ENC_RAW
    - OBJ_LIST：OBJ_ENC_LISTPACK / OBJ_ENC_LINKEDLIST
    - OBJ_HASH：OBJ_ENC_LISTPACK / OBJ_ENC_HT
    - OBJ_SET：OBJ_ENC_INTSET / OBJ_ENC_HT
    - OBJ_ZSET：OBJ_ENC_LISTPACK / OBJ_ENC_SKIPLIST
  - **ptr**：指针，指向 SDS、dict、zskiplist、intset、listpack 底层结构
  - refcount 引用计数：
    - `refcount == 0`：立即释放对象内存
    - `refcount == 1`：独占对象，可修改
    - `refcount >1`：**对象共享**，只读，修改会触发拷贝（写时复制）
  - lru 时间戳：记录对象最后一次访问时间，是Redis内部时钟

  ``` cpp
  typedef struct redisObject {
    unsigned type:4;        // 对象类型 4bit
    unsigned encoding:4;    // 底层编码 4bit
    unsigned lru:24;        // LRU时间戳，24bit
    int refcount;           // 引用计数，内存共享、回收
    void *ptr;              // 指向底层数据结构
  } robj;
  ```
- 对象共享：ptr指向一个现有对象，本共享的对象引用计数+1，Redis 会预先创建**0\~9999 共 10000 个整数对象OBJ_ENC_INT**，全局共享，写时复制，字符串对比开销高所以不用而是用整数
- 内存回收：引用计数只有减到0才释放对象
- 内存淘汰：LRU算法
- 多态命令：根据type、encoding决定具体执行什么命令
  \### String 字符串对象
- `OBJ_ENC_INT`：存数字，ptr 直接存 64 位整数
- `OBJ_ENC_EMBSTR`：≤44 字节短字符串；redisObject+SDS 一次 malloc 连续内存，只读，修改转成RAW
- `OBJ_ENC_RAW`：\>44 字节；redisObject 和 SDS 两次 malloc 分开分配
- 浮点数也作为字符串保存
- 字符串对象是唯一一种会被其他对象嵌套的对象
  \### List 列表对象
- `OBJ_ENC_LISTPACK`：元素小于64字节，元素数量小于512个
- `OBJ_ENC_LINKEDLIST`：老版本双向链表，现在极少触发
  \### Hash 哈希对象
- `OBJ_ENC_LISTPACK`：键值对字符串小于64字节，键值对数量小于512个
- `OBJ_ENC_HT(hashtable)`：底层 dict，数据量大切换到此编码
  \### Set 集合对象
- `OBJ_ENC_INTSET`：全部整数，元素数量小于512个
- `OBJ_ENC_HT`：底层 dict，key 存元素，value=nullptr
  \### Zset 有序集合对象
- `OBJ_ENC_LISTPACK`：member+score，元素，元素小于64字节，元素数量小于512个
- `OBJ_ENC_SKIPLIST`：zset 复合结构 `zskiplist + dict`，冗余两份数据
  \# 单机Redis
  \## 底层数据结构的序列化格式
- 变长长度编码：模式（2bit）+内容
  - `00xxxxxx`：低 6bit 代表长度，1 字节，0‑63
  - `01xxxxxx yyyyyyyy`：低 14bit 代表长度，2 字节
  - `10xxxxxx`：后面 4 字节存 32 位长度，共 5 字节
  - `11xxxxxx`：特殊模式，低 6bit 标记：int8/int16/int32/LZF 压缩字符串
- String（SDS）序列化 RDB_TYPE_STRING (0)：
  - **普通字符串**：变长长度编码 + 原始字节
  - 小整数编码（11 开头特殊标记）：
    - `11 000000`：后 1 字节 int8
    - `11 000001`：后 2 字节 int16
    - `11 000010`：后 4 字节 int32
  - **LZF 压缩字符串** `11 000011`

  <!-- -->

      标记(1B)｜压缩后长度｜原始长度｜压缩字节流
- List 列表 RDB_TYPE_LIST_QUICKLIST(14)

<!-- -->

    [变长编码：quicklist节点总数]
    循环每个节点：
        [完整ziplist/listpack二进制块]

- hashtable(dict) RDB_TYPE_SET(2)：

<!-- -->

    [变长编码：元素总个数N]
    循环N次：[字符串编码保存每个element]

- intset 整数集合 RDB_TYPE_SET_INTSET (11)：
  - `intset`二进制块原封不动写入 rdb：encoding+length+contents \[\] 整块拷贝
- ziplist ：将压缩列表转换成字符串对象保存
- zskiplist 跳表 RDB_TYPE_ZSET (3)：

<!-- -->

    [变长编码：元素总个数N]
    循环N次：
        member（字符串编码，SDS内容）
        score（double编码）

### 对象序列化

- Set 集合：
  - hashtable(dict) RDB_TYPE_SET(2)
  - intset 整数集合 RDB_TYPE_SET_INTSET (11)
- ZSet 有序集合：
  - listpack 紧凑编码 RDB_TYPE_ZSET_LISTPACK (12)：整块直接 dump 内存 listpack 二进制，内部交替存储`member、score`
  - zskiplist+dict 复合结构 RDB_TYPE_ZSET (3) / RDB_TYPE_ZSET_2 (5)
    \## Redis常用命令
    \### 通用基础命令
- key 操作
  - `keys *`：遍历所有 key，**生产禁止**，阻塞
  - `scan 0 [match xxx] [count n]`：迭代遍历 key，非阻塞，生产替代 keys
  - `exists key`：判断 key 是否存在，返回 1/0
  - `del key1 key2`：删除 key
  - `unlink key`：非阻塞删除，大 key 优先用
  - `expire key sec`：设置过期秒；`pexpire key ms`毫秒
  - `ttl key`：查看剩余过期秒，-1 永不过期，-2 已不存在
  - `persist key`：取消过期
  - `rename old new`：重命名；`renamenx`不存在才重命名
  - `type key`：查看 value 数据类型
- String 字符串
  - `set k v [ex sec] [px ms] [nx|xx]`
    - nx：key 不存在才设置；xx：key 存在才覆盖
  - `get k` 获取
  - `mset k1 v1 k2 v2`批量设置
  - `mget k1 k2`批量获取
  - `incr k`自增 1；`incrby k 10`；`decr`自减
  - `append k str`追加字符串
- List 列表
  - `lpush k v1 v2`左边插入；`rpush`右边插入
  - `lpop k`左弹出；`rpop`右弹出
  - `blpop k timeout`阻塞弹出
  - `lrange k start end`范围查询，`‑1`代表末尾
  - `llen k`列表长度
- Hash 哈希
  - `hset k f1 v1 f2 v2`设置字段
  - `hget k f`获取单个字段
  - `hgetall k`获取全部字段值，大 hash 谨慎
  - `hmget k f1 f2`批量取字段
  - `hkeys k`只拿所有 key；`hvals`只拿 value
  - `hdel k f1`删除 hash 字段
  - `hexists k f`字段是否存在
- Set 集合
  - `sadd k m1 m2`添加元素
  - `smembers k`全部元素
  - `sismember k m`判断元素是否在集合
  - `srem k m`删除元素
  - `scard k`集合基数大小
- ZSet 有序集合
  - `zadd k score1 m1 score2 m2`
  - `zrange k start end [withscores]`从小到大
  - `zrevrange`从大到小
  - `zrangebyscore`按分数区间
  - `zrem k m`删除成员
  - `zcard k`元素数量
  - `zscore k m`获取成员分数
    \### 服务器信息命令
- `info [section]`
  - `info server`：版本、运行时间、端口
  - `info memory`：内存使用，used_memory、碎片率
  - `info replication`：主从复制状态、offset
  - `info clients`：客户端连接
  - `info stats`：命令统计、key 过期命中率
  - `info persistence`：RDB/AOF 持久化
- `config get *`查看配置；`config set xxx yyy`运行时改配置
- `dbsize` 当前 db key 总数量
- `client list` 所有客户端连接信息
- `client kill ip:port`杀掉异常客户端
- `slowlog get n`慢查询日志
- `monitor`实时打印所有命令，生产慎用
- `flushdb`清空当前 db；`flushall`清空全部 db
  \### 主从复制命令
- `replicaof ip port` 设置为从节点
- `replicaof no one`从节点提升为主节点
- `info replication`看主从状态偏移量
  \### 哨兵模式命令
- `sentinel master mymaster`查看主节点完整信息
- `sentinel slaves mymaster`列出所有从节点
- `sentinel sentinels mymaster`列出其他哨兵
- `sentinel get‑master‑addr‑by‑name mymaster`获取当前 master ip 端口（客户端核心 API）
- `sentinel failover mymaster`手动触发故障转移
  \### 集群命令
- `redis‑cli -c -h ip -p port` **‑c 开启集群自动重定向 MOVED/ASK**
- `redis‑cli --cluster create ip1:port ip2:port ...` 创建集群
- `redis‑cli --cluster add‑node 新ip:port 任意旧ip:port` 添加主节点
- `redis‑cli --cluster add‑node --cluster‑slave` 添加从节点
- `redis‑cli --cluster reshard ip:port`在线槽迁移扩容
- `redis‑cli --cluster check ip:port`集群完整性校验
- `cluster info`集群整体状态，看 cluster_state:ok/fail
- `cluster nodes`完整打印所有节点 node‑id、ip、角色、slot、主从关系
- `cluster slots`槽位分配映射
- `cluster keyslot key`计算 key 对应的 slot 编号
- `cluster meet ip port`节点握手加入集群
- `cluster replicate <master‑node‑id>`设置当前节点为从节点
- `cluster addslots slot1 slot2`手动分配槽
- `cluster delslots slot`删除槽分配
- `cluster forget node‑id`移除故障节点记录
- `cluster setslot <slot> node <nodeid>`修改槽归属（迁移完成后）
  \### 持久化命令
- `bgsave`后台触发 RDB 快照
- `lastsave`上一次 RDB 时间戳
- `bgrewriteaof`触发 AOF 重写
  \## 数据库
- 本质：dict 字典，保存 key‑value
- 所有数据库都保存在redisServer结构的db数组中，dbnum值决定创建多少数据库（由服务器配置的database选项决定）
- redisClient结构的db指针记录了当前使用的数据库，默认0号数据库，可以通过SELECT切换数据库
- 读写数据库会更新Server命中和不命中次数参数，更新LRU，执行过期删除，标记脏位（watched_keys），发送数据库通知
- redisDb结构
  - `dict`：**核心字典**，全部普通 key‑value，key 是 SDS，value 是 redisObject\*
  - `expires`：过期字典。同样 key (SDS)，value 存**unix 毫秒时间戳**
  - `blocking_keys`：blpop/brpop 等阻塞命令，客户端阻塞在哪些 key 上
  - `ready_keys`：key 有数据到来，唤醒阻塞客户端
  - `watched_keys`：事务 WATCH，记录哪个客户端监视哪些 key
  - `id`：数据库编号，默认 db0

  ``` cpp
  typedef struct redisDb {
    dict *dict;                 // 主字典：保存所有 key → robj
    dict *expires;              // 过期字典：key → 过期时间戳(毫秒)
    dict *blocking_keys;        // BLPOP 阻塞等待的key
    dict *ready_keys;           // 阻塞key被push之后唤醒标记
    dict *watched_keys;         // MULTI WATCH 监视key，事务CAS
    int id;                     // db编号，0,1,2...
    long long avg_ttl;          // 统计，平均TTL
  } redisDb;
  ```
- watched_keys作用：保证事务原子性，执行 EXEC 提交事务时，检查 watch 过的 key 是否被修改；如果被修改，事务直接放弃执行返回 nil
  \### 键过期机制
- 设置过期时间

```
expire key sec      // 秒
pexpire key ms      // 毫秒
expireat key timestamp_sec
pexpireat key timestamp_ms
persist key         // 清除过期，把key从expires字典删掉
```

- 过期删除策略：
  - 惰性删除（被动删除）：CPU友好，访问 key 的时候才检查是否过期；如果已过期，删除 key，返回 nil，expireIfNeeded在每次读写前调用
  - 定期删除（主动抽样删除）：内存友好，每隔一段时间（默认 100ms），随机抽取一部分 expires 里的 key，删除已经过期的，activeExpireCycle在每次ServerCron执行时调用
  - 内存淘汰 maxmemory（兜底，不属于过期删除）：当内存达到 maxmemory 上限，按照淘汰策略删掉部分 key，不管有没有过期
- 过期键持久化
  - RDB 生成时：已过期的 key**不会写入 RDB 文件**（主从模式下，主不写入，从写入）
  - AOF：不会被过期键影响，key 过期被删除时，向 AOF 追加一条`del`命令，AOF重写时已过期的 key**不会写入重写后AOF文件**
  - 主从复制：**过期时间由主库控制**；从库不会主动删除过期 key，主库 key 过期，主库发送 del 同步给从库
    \### 数据库通知
- 本质：类似MySQL触发器，键发生变更时自动触发，默认关闭，开启会消耗少量CPU
- 等同于自动执行两条PUBLISH命令，notifyKeyspaceEvent函数实现
- 打开通知：配置notify‑keyspace‑events参数

| 字符 | 含义 |
|----|----|
| K | 开启键空间通知（keyspace 前缀） |
| E | 开启键事件通知（keyevent 前缀） |
| g | 通用命令：del、rename、expire 等 |
| $|String 字符串命令|
|l|List 列表命令|
|s|Set 集合命令|
|h|Hash 哈希命令|
|z|Zset 有序集合命令|
|x|**过期事件 expired**（惰性 / 定期删除产生）|
|e|内存淘汰 evicted（maxmemory 删除 key）|
|A|别名 = `g$lshztxed\`，除 m、n 以外全部事件 |  |

```
# 只监听过期事件（最常用）
config set notify-keyspace-events Ex

# 开启全部事件
config set notify-keyspace-events KEA
```

- 通知类型
  - 键空间通知（K）`__keyspace@db__:<key>`：消息内容是事件名称（set/del/expired）
  - 键事件通知（E）`__keyevent@db__:<event>`：消息内容是发生的键名称
    \## RDB持久化
- 把内存全量数据二进制写入磁盘`.rdb`文件（一个经过压缩的二进制文件），重启自动直接加载快照恢复数据（没有主动调用的命令）
- AOF更新频率比RDB高，服务器优先选择AOF持久化
- BGSAVE、SAVE、BGREWRITEAOF不能同时进行，会阻塞（因为都在大量磁盘写入）
- 触发方式：会设置redisServer的saveparams、dirty、lastsave
  - 手动触发
    - `BGSAVE`：fork 子进程，子进程做快照，**主线程继续处理客户端命令**
    - `SAVE`：**主线程同步执行**，全部内存 dump，大数据量会阻塞，生产禁止用
  - 自动触发：配置服务器的save选项，serverCron定制检查save条件满足就执行BGSAVE命令

  ``` conf
  save 900 1     # 900秒内至少1次修改 → BGSAVE
  save 300 10    # 300秒内至少10次修改 → BGSAVE
  save 60 10000  # 60秒内至少10000次修改 → BGSAVE
  # save "" 关闭自动RDB
  ```
- BGSAVE 完整流程：
  - 调用`fork()`系统调用，**创建子进程**
  - 遍历全部内存数据，序列化二进制，写入临时`.rdb.tmp`文件
  - 把临时文件 rename 替换正式 rdb 文件
  - 子进程退出；主线程收到信号，更新最近一次快照时间记录
    \### RDB文件结构

  ``` txt
  [魔数5B][版本4B] → [AUX辅助字段] → [DB0块][DB1块]... → [EOF标记0xFF] → [CRC64校验8B]
  ```
- 魔数 Magic：5 字节 `REDIS`
- RDB 版本号：4 字节 ASCII 字符串
- AUX 辅助字段（可选，操作码 0xFA）：redis 版本、rdb 创建时间、占用内存、rdbcompression 标记等

<!-- -->

    0xFA  [aux‑key长度][aux‑key][aux‑value长度][aux‑value]

- 数据库块（可多个，db0、db1...）：以操作码 `0xFE` 开头
- **Opcode**（可选，没有就代表 key 永不过期）：`0xFD`：**秒级**后面 4 字节int，`0xFC`：**毫秒级**后面 8 字节 unsigned long
- value‑type（1 字节）：记录 value 底层编码类型：底层结构不是 robj 对象
- key：Redis 字符串变长编码存储
- value：根据 value‑type 使用对应序列化格式

<!-- -->

    [过期标记opcode(可选)]  value‑type(1字节)  key(字符串编码)  value(对应类型编码)

- EOF 标记 1 字节 `0xFF`
- CRC64 校验和 8 字节
- 分析RDB文件：od命令打印rdb文件，redis-check-dump自动检查
  \## AOF持久化
- 记录 Redis 执行过的**写命令和PUBSUB/SCRIPT LOAD命令**，以文本协议追加写入文件；重启时逐条重放命令恢复数据
- 开启：

``` conf
appendonly yes          # 开启AOF，默认no
appendfilename "appendonly.aof"
dir ./
```

- 命令追加：追加到redisServer的aof_buf缓冲区，服务器循环会调用flushAppendOnlyFIle按策略将缓冲区内容落盘
- 落盘策略：服务器选项appendfsync配置
  - always：每执行一条写命令就 fsync 刷盘
  - everysec：后台线程每秒 fsync 一次
  - no：交给操作系统自动刷盘，不主动调用 fsync
    \### AOF文件结构
- `*N`：参数总个数
- `$len`：后面字符串字节长度

<!-- -->

    *2
    $6
    SELECT
    $1
    0
    *3
    $3
    SET
    $3
    key
    $5
    hello
    *2
    $4
    SADD
    $2
    s1
    $1
    a

### AOF重写

- AOF 文件越滚越大，使用BGREWRITEAOF生成最小化的命令序列生成当前数据库
- 重写配置

<!-- -->

    auto‑aof‑rewrite‑percentage 100    # 文件比上次重写后增大100%触发
    auto‑aof‑rewrite‑min‑size 64mb     # 最小达到64MB才触发
    no‑appendfsync‑on‑rewrite yes      # 暂停 everysec 的 fsync

- BGREWRITEAOF 执行流程：
  - fork 子进程
  - 子进程遍历 Redis 内存，根据当前 key 真实状态生成最简 RESP 命令，写入临时文件`appendonly.aof.tmp`
  - 重写期间主线程继续接收新写命令：
    - 新命令正常追加到老 AOF 文件
    - 同时放到**aof_rewrite_buf 重写缓冲区**保存增量数据
  - 子进程重写完成，给主线程发信号
  - 主线程把`aof_rewrite_buf`缓冲区里新增命令追加到子进程
  - rename 把临时文件替换正式 appendonly.aof覆盖老AOF文件
- 与BGSAVE的冲突：两者不能同时执行，谁先执行就等他执行完再执行
  \### RDB与AOF对比
  \| 维度 \| RDB 快照 \| AOF 日志 \|
  \| ---- \| ---------------- \| ------------------------------------ \|
  \| 内容 \| 二进制快照，某一刻完整内存 \| RESP 写命令序列 \|
  \| 丢失数据 \| 两次快照间全部丢失 \| everysec 最多丢失 1 秒 \|
  \| 文件大小 \| 紧凑小 \| 通常更大 \|
  \| 恢复速度 \| 极快，直接加载二进制 \| 慢，逐条回放命令 \|
  \| fork \| BGSAVE 需要 fork \| BGREWRITEAOF 需要 fork；正常 AOF 不需要 fork \|
  \| 损坏风险 \| CRC 校验，局部损坏直接不可用 \| 尾部损坏可截断恢复 \|
  \## 事件驱动
- 整体基于事件驱动模型单Reactor单线程模型，两类事件源：socket、定时器（无序链表），支持select、epoll、kqueue等
- 主线程循环aeApiPoll：AOF刷盘、过期抽样、IO通信、命令执行，串行执行并发弱，IO阻塞或业务阻塞会影响整体性能
- aeApiPoll主循环、aeWaitIO多路复用、aeProcessEvents事件分派器（事件队列）
- 事件类型
  - 文件事件 FileEvent（socket）：
    - 可读事件（AE_READABLE）：回调readQueryFromClient
    - 可写事件（AE_WRITABLE）：回调sendReplyToClient
    - 优先处理可读事件，也就是读者-写者模型读优先策略
  - 时间事件 TimeEvent（定时器）：
    - 时间事件：（包括一次性的AE_NOMORE和周期性的）
      - `id`：事件 ID
      - `when`：下一次执行的时间戳（毫秒）
      - `proc`：回调函数
      - `next`：链表指针，所有时间事件串成单向链表
- 主要周期时间事件 serverCron：默认每秒执行 10 次（hz=10）
  - 主动抽样删除过期 key（定期删除）
  - 更新各种统计信息（内存、客户端统计）
  - 检查是否满足条件自动 BGSAVE、自动 BGREWRITEAOF
  - 清理超时客户端连接
  - 处理 AOF everysec 后台 fsync 调度
  - 调整内存碎片统计
  - 集群相关定时任务
- 周期执行逻辑：顺着链表遍历执行回调函数，执行完更新when，等待下次调用
- 事件执行流程aeProcessEvents：

<!-- -->

    1.计算最近要执行的时间事件还有多久到期 → 得到poll超时timeout
    2.调用epoll_wait(timeout)阻塞等待IO就绪（有超时时间不会无限阻塞）
    3.处理全部就绪文件事件（客户端读写、执行命令）
    4.遍历时间事件链表，执行到期的时间事件(serverCron等)
    5.回到循环开头

## 客户端

- 一个连接对应一个client对象，所以主线程维护全部客户端，在redisServer的clients链表中
- 客户端结构体client：
  - `mstate`：multi 事务状态，缓存入队命令，EXEC 一次性执行
  - 命令实现：argc、argv保存参数，argc\[0\]保存命令名，根据命令表cmd查找对应命令输入参数执行
  - 输入缓冲区 querybuf：从 socket 读到的字节全部追加到`sds querybuf`，直到完整解析出一条或多条 RESP 命令填充 `argc / argv`，剩余数据保留等待下一轮解析，输入缓冲区会有最大限制1GB
  - 输出缓冲区：
    - **buf（内联小缓冲区，sds，固定）**：优先使用，短响应直接写这里
    - **reply（链表 list，可变）**：返回数据量大
    - 输出流程：把结果写入 client 输出缓冲区（buf/reply）；**不会直接调用 send**，给 fd 注册`AE_WRITABLE`可写文件事件，等待epoll调度
    - 输出缓冲区溢出：大查询会直接把输出缓冲打满，超过阈值直接关闭客户端连接
  - 伪客户端：本地域socket，Lua脚本、AOF重写会使用
  - 阻塞客户端（假阻塞，停止处理而已）：设置 client 标记`CLIENT_BLOCKED`，挂到 redisDb 的`blocking_keys`字典对应 key 链表上，不再处理直到key 被 push 数据，放入`ready_keys`
  - 客户端超时：`serverCron`时间事件里面遍历全部 client，对比`lastinteraction`，空闲超过`timeout`直接关闭客户端

  ``` cpp
  typedef struct client {
    uint64_t id;            // 客户端唯一ID
    robj* name;             // 可以调用CLIENT setname为客户端命名
    int fd;                 // socket文件描述符，-1代表无网络客户端（Lua脚本、内部伪客户端）
    sds querybuf;           // 查询缓冲区：存放从socket读到的原始RESP协议数据
    int argc;               // 当前命令参数个数
    robj **argv;            // 当前命令参数对象数组 argv[0]命令名，argv[1...]参数
    struct redisCommand *cmd; // 命令实现函数表
    sds buf;                // 输出缓冲区1：固定小缓冲区，优先放短回复
    list *reply;            // 输出缓冲区2：大回复链表，buf放不下就挂这里

    int flags;              // 客户端标记：SLAVE、MONITOR、BLOCKED、UNBLOCKED等
    dict *watched_keys;     // WATCH命令监视的key，事务用
    multiState mstate;      // MULTI事务状态
    int btype;              // BLPOP等阻塞类型；阻塞时保存阻塞key
    list *blocking_keys;    // 当前客户端阻塞在哪些key

    time_t lastinteraction; // 上次交互时间，用于超时关闭空闲客户端
    ...
  } client;
  ```
- 常用flag定义：

``` cpp
#define CLIENT_SLAVE            (1<<0)   // 从节点客户端（replica）
#define CLIENT_MASTER           (1<<1)   // master客户端（主从复制，对方是master）
#define CLIENT_MONITOR          (1<<2)   // MONITOR监控客户端
#define CLIENT_BLOCKED          (1<<3)   // 客户端被BLPOP等阻塞
#define CLIENT_UNBLOCKED        (1<<4)   // 已经被唤醒，待解除阻塞
#define CLIENT_MULTI            (1<<5)   // 处于MULTI事务状态
#define CLIENT_DIRTY_CAS        (1<<6)   // WATCH的key被修改，EXEC要失败
#define CLIENT_CAS              (1<<7)   // 启用CAS(WATCH)
#define CLIENT_PUBLISH          (1<<8)   // Pub/Sub订阅客户端
#define CLIENT_PREVENT_AOF      (1<<9)   // 命令不写入AOF
#define CLIENT_PREVENT_REPLICA  (1<<10)  // 命令不向从库复制传播
#define CLIENT_READONLY         (1<<11)  // 从节点上开启readonly客户端
#define CLIENT_ASKING           (1<<12)  // 集群ASKING标记
#define CLIENT_NO_MULTI         (1<<13)  // 不允许MULTI事务
#define CLIENT_LUA              (1<<14)  // Lua脚本伪客户端
#define CLIENT_PROTECTED        (1<<15)  // 保护模式，未配置密码禁止访问
#define CLIENT_PENDING_READ     (1<<16)  // 等待读事件
#define CLIENT_PENDING_WRITE    (1<<17)  // 等待写事件输出缓冲区发送
```

- 客户端生命周期：
  - 新 TCP 连接到来 → accept，分配 fd，新建 client 结构体，初始化缓冲区，注册 AE_READABLE 可读事件
  - 数据读到`querybuf`，解析 RESP，得到`argc/argv`
  - 调用命令处理函数执行命令，结果写入输出缓冲区 buf/reply，注册 AE_WRITABLE
  - 等待epoll调度发送回复给服务器
  - 空闲超时 / 缓冲区溢出 / 客户端 close → client 销毁，释放内存，关闭 fd
- 关闭客户端的情况：
  - 进程退出
  - 发送不符合格式的命令
  - CLIENT KILL命令手动关闭
  - 最后一次互动的时间lastinteraction超过超时时间
  - 输入、输出缓冲区溢出
- 缓冲区限制：
  - 硬性限制：超过硬性设置直接关闭
  - 软性限制：超过软性设置，obuf_soft_limit_reached_time记录到达软性设置时间，继续监视，如果一直超出软性限制且持续时间超过阈值，就关闭
  - 可以为主客户端、从客户端、发布订阅客户端设置不同的软性和硬性限制
    \## 服务器
- 全局唯一实例，保存Redis全部配置、运行状态、共享资源，所有模块直接访问
  \### 服务器初始化
- main全流程：
- 加载默认配置：`initServerConfig()`
- 加载人工配置
- 创建运行时资源：`initServer()`
- 还原数据库状态：加载 RDB / AOF
- `aeMain(server.el)`进入事件循环
  \#### initServerConfig ()：
- 设置默认参数，只做内存赋值，不创建资源
- 网络配置
  - `server.port = 6379`：默认监听端口
  - `server.tcp_backlog = 511`：tcp backlog 默认值
  - `server.dbnum = 16`：默认 16 个逻辑 DB
- 持久化默认值
  - `server.saveparams` 初始化 RDB 默认 save 规则（900s 改 1 次、300s 改 10 次、60s 改 10000 次）
  - `server.appendonly = 0`：AOF 默认关闭
  - `server.dirty = 0`、`server.lastsave = 0`
- 内存淘汰
  - `server.maxmemory = 0`：0 代表不限制内存
  - `server.maxmemory_policy = MAXMEMORY_NO_EVICTION`：默认不淘汰
- 事件与时间
  - `server.hz = 10`：serverCron 默认每秒执行 10 次
  - 初始化 LRU 时钟 `server.lruclock`
- 命令表
  - 把内置命令数组填充到 `server.commands`（dict 字典），注册每个命令的处理函数、参数、flag
    \#### 加载配置
- `redis-server redis.conf`覆盖默认参数
  \#### initServer ()：真正创建运行时资源
- 事件循环初始化
  - `server.el = aeCreateEventLoop()` 创建 Reactor 事件循环
  - 注册**serverCron 时间事件**（唯一的周期性时间事件）
- 数据库初始化
  - 分配 `server.db` 数组，大小`server.dbnum`
  - 循环每一个 redisDb：
    - db-\>dict = dictCreate () 主 key 字典
    - db-\>expires = dictCreate () 过期字典
    - db-\>blocking_keys /ready_keys/watched_keys 全部新建空 dict
- 各种全局链表初始化
  - `server.clients`：存放所有正常 client
  - `server.clients_to_close`：延迟待释放 client 链表
  - `server.slaves`：从节点链表
- AOF 缓冲区初始化
  - `server.aof_buf = sdsempty()`
  - `aof_rewrite_buf_blocks` 空 list
- 网络 socket 初始化
  - 创建 TCP 监听 fd，设置非阻塞、TCP_NODELAY
  - 向 ae 事件循环注册**accept 可读回调**：`acceptTcpHandler`，新连接到来触发
- 信号处理注册
  - 注册 SIGINT、SIGTERM 信号处理器，收到信号执行安全关闭流程
- 共享对象池初始化
  - 创建小整数共享 robj（0‑9999），大量命令可以复用，减少 malloc
    \### redisServer核心字段
- 数据库存储
  - `redisDb *db`：redisDb 数组，默认 16 个逻辑数据库
  - `int dbnum`：逻辑数据库总数量
- 事件循环相关
  - `aeEventLoop *el`：主线程 Reactor 事件循环，管理文件事件、时间事件
  - `long long unixtime / mstime`：缓存系统时间，`serverCron`定时更新，减少系统调用
  - `unsigned lruclock:22`：LRU 时钟缓存，用于 robj 对象空闲时间计算
- 命令系统
  - `dict *commands`：命令字典；key 为命令名字符串，value 是`redisCommand`，存储命令函数指针、参数约束、命令标记
- 网络与客户端
  - `int port`：服务监听端口
  - `list *clients`：全部正常客户端链表
  - `list *clients_to_close`：延迟关闭客户端链表，事件循环末尾统一释放，避免回调内野指针
  - `client *current_client`：**当前正在执行命令的客户端**
  - `client *master`：副本模式下，连接 master 的客户端；主节点为 NULL
  - `list *slaves`：主节点下所有从节点 client 链表
- 持久化（RDB/AOF）
  - `struct saveparam *saveparams`：RDB 自动 save 配置数组（时间、修改次数阈值）
  - `long long dirty`：上一次 RDB 成功之后 key 修改总次数，用于判断自动快照条件
  - `time_t lastsave`：上一次 BGSAVE/SAVE 成功的时间戳
  - `char *aof_filename`：AOF 文件名
  - `sds aof_buf`：AOF 主缓冲区，写命令先追加到此缓冲区
  - `list *aof_rewrite_buf_blocks`：BGREWRITEAOF 重写增量缓冲区，保存重写期间新写入命令
- 内存淘汰配置
  - `size_t maxmemory`：Redis 最大内存上限
  - `int maxmemory_policy`：内存淘汰策略枚举（volatile‑lru /allkeys‑lru 等）
- 主从复制
  - `char replid[]`：全局复制 ID
  - `long long master_repl_offset`：本实例复制偏移量
  - `char *repl_backlog`：复制积压环形缓冲区，用于 PSYNC 部分重同步
- 集群与哨兵
  - `int cluster_enabled`：集群模式开关
  - `clusterState *cluster`：集群完整运行状态
  - `int sentinel_mode`：哨兵模式标记位
- 运行统计指标
  - `stat_keyspace_hits`：key 命中次数统计
  - `stat_keyspace_misses`：key 未命中次数统计
  - `stat_total_commands_processed`：累计执行命令总数
    \### 普通命令通信（服务器接收到返回）
- 网络 IO 接收（操作系统 → Redis 用户态）
  - epoll_wait 阻塞等待，当 socket 内核缓冲区有数据，触发**文件可读事件**
  - 回调`readQueryFromClient`执行`read()`系统调用，把内核缓冲区字节拷贝到用户态`client->querybuf`输入缓冲区
  - 流式接收，一次可能读到半条、多条命令，未处理字节保留等待下次解析
- 协议解析与命令分发
  - 循环解析 querybuf 内 RESP 协议，拆分参数，封装成`robj`对象数组`argc/argv`
  - 全局`server->current_client`指向当前正在处理的客户端上下文
  - 事务判断：如果是事务client，直接入队，不执行数据库逻辑，直接生成 QUEUED 应答写入输出缓冲区
  - `argv[0]`命令名字，查询全局`server->commands`命令字典，获取对应命令处理函数
- 命令执行，操作数据库与后台模块
  - 调用命令函数（如`setCommand`），修改`redisDb->dict`主字典；设置过期时间则写入`redisDb->expires`过期字典
  - `server->dirty++`，统计 key 修改次数
  - 命令追加写入`server->aof_buf`内存缓冲区，复制命令推送给从节点客户端输出缓冲区，没有刷磁盘，fsync根据AOF落盘策略实施
- 构造应答，输出缓冲区暂存
  - 命令返回结果写入客户端输出缓冲区，不直接send
- 异步网络发送，回传给客户端
  - 给 socket fd 注册`AE_WRITABLE`可写事件，交还事件循环
  - epoll 检测操作系统 TCP 发送缓冲区有空闲空间，触发回调`sendReplyToClient`
  - 执行`send()`系统调用，输出缓冲区数据拷贝给内核 TCP 发送缓冲区，数据经由 TCP 网络返回**客户端**
  - 完成清空输出缓冲区，注销可写事件
  - 善后：释放 argv 参数 robj 引用计数，`server->current_client`置空，回到事件循环等待下一轮 IO 事件
    \### BLPOP 阻塞通信
- 网络连接不断开，客户端 TCP 层面一直等待；
- 服务端只是打标记、挂链表，主线程继续处理别的客户端，**线程没有阻塞**；
- 只有 key 出现数据才走完整应答发送链路。
  \### MULTI‑EXEC 事务通信
- MULTI 之后的命令**只是入队列**，QUEUED 只是提示，数据没有真正修改；
- EXEC 才批量执行；
- WATCH 机制：别的客户端修改 key，给**发起 watch 的客户端**打上`CLIENT_DIRTY_CAS`，EXEC 直接返回 nil
  \### serverCron 定时任务执行流程
- 基础信息刷新
  - 更新服务器时间缓存
    - 刷新全局秒级、毫秒级系统时间
    - 减少高频 gettimeofday 系统调用
  - 更新 LRU 时钟缓存
    - 维护全局 lruclock 时钟
    - 用于计算对象空闲时长，支撑 LRU 内存淘汰
  - 更新服务器各项运行统计指标
- 过期 key 定期删除操作
  - 遍历所有逻辑数据库 redisDb
  - 执行抽样式过期清理
    - 每次随机抽取固定数量 key
    - 删除已经超时过期的键
    - 高过期率时循环抽样，保证清理力度
  - 仅抽样扫描，不全局遍历数据库，控制 CPU 开销
  - 配合惰性删除，构成 Redis 双过期删除策略
- 客户端资源管理与清理
  - 遍历全局所有客户端连接
    - 检测空闲超时客户端，标记待关闭
    - 检测输出缓冲区溢出超限客户端，标记待关闭
    - 检测阻塞命令（BLPOP 等）超时，唤醒客户端返回空值
  - 统一处理待关闭客户端链表
    - 不在 IO 回调内直接释放客户端
    - 定时统一释放资源，避免野指针异常
- RDB 持久化调度与维护
  - 校验自动 RDB 保存规则
    - 比对全局 dirty 数据修改次数
    - 比对上一次 RDB 保存时间
  - 满足阈值自动触发 BGSAVE 后台快照
  - 回收 RDB 子进程
    - 清理僵尸进程
    - 更新 lastsave 保存时间
    - 重置 dirty 修改计数器
- AOF 持久化与重写维护
  - 监控 BGREWRITEAOF 重写子进程状态
  - 回收 AOF 重写僵尸子进程
  - 配合后台线程处理 AOF 刷盘逻辑
  - 管控重写期间 fsync 暂停策略
- 主从复制定时维护
  - 从节点逻辑
    - 检测 Master 连接状态
    - 断连后定时重试建立主从连接
  - 主节点逻辑
    - 遍历检测所有从节点健康状态
  - 维护复制积压环形缓冲区
  - 校验、推进复制偏移量同步状态
- 内存资源管控
  - 实时检测当前实例内存占用
  - 配合 maxmemory 最大内存限制
  - 内存超限触发对应内存淘汰策略
- 集群与哨兵定时任务
  - 集群模式
    - 节点心跳通信、故障检测
    - 槽位校验、集群状态同步维护
  - 哨兵模式
    - 监控主从节点在线状态
    - 判定主观下线、客观下线
    - 触发故障转移调度逻辑
- 自身定时任务刷新
  - 更新当前时间事件的下次执行时间戳
  - 实现周期性无限循环执行
    \# 多机Redis
    \## 主从复制
- 读写分离、数据备份；主节点 (master) 写，从节点 (replica/slave) 读；**默认异步复制**
- 从服务器是主服务器的客户端
- 角色：
  - master：主节点，负责接收写命令，可以读；维护所有从节点状态
  - replica：从节点，完整复制 master 数据；默认只读，拒绝写命令
- 拓扑结构：
  - 一主多从：master 直接挂载多个 replica
  - 链式复制（不推荐）：A (master)→B (replica)→C (replica)，B 既是从也是主
- 基础设施：
- master 端：
  - `replid`：运行 ID，主从都有自己的，启动时自动生成，40个随机十六进制字符，初次复制时传给从
  - `repl_backlog`：复制积压环形缓冲区，固定长度，先入先出，默认1MB，保存最近一段已经执行过的写命令（offset+value）
  - `master_repl_offset`：复制偏移量，master 每传播一条命令增加字节数
  - `server.slaves`链表：保存所有从节点 client 对象
- replica 端：
  - `replid`：运行 ID，初次复制时保存主的
  - `master_repl_offset`：复制偏移量，每接收到一条指令增加字节数
  - `server.master`：指向 master 的 client 客户端上下文
- 全量同步和部分同步：
  - 全量同步：第一次连接、replid 不匹配、backlog 缓冲区数据已经被覆盖时，master BGSAVE 生成 RDB，网络传输 RDB，回放缓冲区积压命令
  - 部分同步：短暂网络抖动断开重连；replid 相同，接收到从发来的offset，判断offset 还在 环形缓冲区保存的命令范围内，master 直接补发 offset 之后的命令，不需要 RDB，回复从+CONTINUE指令
    \### PSYNC复制流程：
- 设置主服务器地址端口
  - 从机调用`SLAVEOF 主机ip 主机端口`复制主服务器
  - 将主服务器地址端口保存到redisServer的masterhost和masterport属性中
  - SLAVAEOF是异步命令，设置完成后即返回OK，实际复制在之后开始执行
- 建立套接字连接
  - serverCron 检测到需要复制，replica 主动向 master 发起 TCP 连接
  - master 接收连接，创建专门用于复制的 client 客户端对象
- 版本、权限握手
  - replica 发送 ping，校验网络连通
  - 密码认证；masterauth，从服务器向主服务器发送AUTH命令，校验主服务器的requirepass，不匹配直接断开
  - 端口信息：从服务器发送REPLCONF命令，主服务器记录在client对象的salve_listenling_port属性中
- 判断是全量同步还是部分同步
  - replica 携带`replid`、`repl_offset`发给 master 执行 PSYNC
  - 条件满足部分同步，不满足全量同步
  - 无论全量还是部分同步，执行同步操作后，主服务器也会成为从服务器的客户端
- 全量同步
  - master 执行 BGSAVE 子进程生成 RDB 快照
  - RDB 生成期间新写入命令，存入`repl_backlog`复制积压缓冲区
  - master 把 RDB 文件传输给 replica
  - replica 清空自身本地旧数据，加载接收的 RDB
  - RDB 加载完成，master 把 backlog 中快照期间增量命令发送给从节点回放
- 命令传播
  - 后续 master 每执行一条写命令，除执行 AOF、本地内存修改外，异步转发给所有 replica
  - replica 接收复制流，本地执行同样命令，保持数据一致
  - 心跳检测：从服务器每秒一次向主发送`REPLCONF ACK <offset>`命令
    - 协助从服务器检查连接状态（心跳间隔时间）
    - 辅助实现min-slaves配置：`min-slaves-to-write 3`和`min-slaves-max-lag 10`表示从服务器数量少于3个和所有延迟都大于10时，主服务器拒绝执行写命令
    - 检测命令丢失（offset不对）
- 主从复制缺陷：缺少故障转移机制，主节点宕机，需要手动把从节点提升为主
  \## 哨兵模式 Sentinel
- 本质：特殊模式运行的Redis进程，不存储业务数据，可以多个组成分布式哨兵集群，一般奇数个（3个为宜，防止脑裂和投票僵局），哨兵不要和业务实例部署在同一台机器上
- 功能：
  - 监控：持续监控 master、replica 从节点存活状态
  - 通知：通过发布订阅把事件通知客户端、其他哨兵
  - 故障转移：master 故障，自动挑选从节点升级新主，其余从节点跟随新主
- 客户端接入：客户端连接哨兵，向哨兵询问当前 master 地址
- 哨兵定时任务（每 100ms 执行）：
  - 向所有已知节点发送 PING 心跳包
  - 处理主观下线 SDOWN 判定：超时无有效响应标记 SDOWN
  - 处理客观下线 ODOWN 投票：master 主观下线后向其他哨兵发起问询
  - hello 频道广播自身元数据
  - 检测是否需要执行 Leader 选举、故障转移流程
  - 更新配置文件，持久化最新集群元数据
- 两条连接（对每个主从必须都有）：
  - 命令连接：发送 PING、INFO、SENTINEL is‑master‑down‑by‑addr 等交互指令
  - 订阅连接：专门订阅`__sentinel__:hello`，只收广播消息，不发普通命令，通过广播构建哨兵集群
- 节点下线判定机制：
  - 主观下线 SDOWN（Subjectively Down）
    - 每个哨兵每秒向 redis 节点发送 PING 心跳命令
    - 在 down‑after‑milliseconds 时间没有收到有效响应 (PONG/LOADING/MASTERDOWN)
    - **仅当前哨兵单方面认为故障，将sentinelRedisInstance中的flags属性设置SRI_S_DOWN标识，属于局部判断，网络抖动会误判，不能直接故障转移**
  - 客观下线 ODOWN（Objectively Down）
    - 哨兵发现 master 进入 SDOWN，发送`SENTINEL is‑master‑down‑by‑addr`向其余哨兵问询状态
    - 赞成 master 故障的哨兵数量 ≥ quorum，标记 master 客观下线，flags属性设置SRI_O_DOWN
      \### 启动初始化
- 启动：不能复用业务实例配置，要单独配置
  - `redis‑sentinel sentinel.conf` （官方推荐方式）
  - `redis‑server sentinel.conf --sentinel` 等价效果
- 加载配置文件
  - 读取`sentinel monitor`配置，获取要监控的 master 信息（ip、port、quorum）
  - 加载其他参数：主观下线超时、选主、故障转移相关参数
  - 初始状态只知道 master，不知道从节点、其他哨兵节点
- 基础内部结构初始化
  - 创建 sentinel 状态结构体（sentinelState），保存所有被监控主节点信息（sentinelRedisInstance）
  - 初始化事件循环 aeEventLoop，和 Redis 实例复用同一套事件驱动框架
  - 注册 sentinel 定时回调，每 100ms 执行哨兵定时任务
- 和被监控 master 建立连接
  - 哨兵主动向 master 建立两条 TCP 连接
    - 一条普通命令连接：发送 PING、INFO、SENTINEL 系列命令
    - 一条 Pub/Sub 订阅连接：订阅`__sentinel__:hello`频道，用于交换元数据
  - 发送 INFO 命令，从 master 返回结果解析出所有 replica 从节点信息，自动发现从库，并更新sentinelRedisInstance的slaves字典
- 创建从服务器连接
  - 发现了从库以后会创建到从服务器的连接，也是两类连接：命令连接和订阅连接
  - 更新从服务器的sentinelRedisInstance信息
- 自动发现其他哨兵
  - 无需手动把所有哨兵互相写进配置文件，每个哨兵定期在 hello 频道广播自身信息（ip、port、runid、监控的 master）
  - 收到其他哨兵广播消息，把对方加入哨兵集群列表，自动完成集群组网
  - 建立哨兵之间 TCP 连接，后续用于投票、消息交互
- 完成初始化，进入持续监控状态
  - 每秒向 master、所有 replica、其他哨兵发送 PING 心跳
  - 持续收集节点状态，执行主观下线判断
    \### 哨兵选举（主节点被判断下线）
- master 进入 ODOWN 之后，哨兵集群内部选举 1 个 Leader 哨兵，只有 Leader 执行故障转移
- 每个哨兵先给自己投一票
- 向其他哨兵发送`SENTINEL is‑master‑down‑by‑addr`请求投票
- 最先到达的获得本哨兵选票
- 获得**哨兵集群半数以上投票**，成功当选 Leader
- 一轮选举失败，等待随机延时开启下一轮选举
  \### 故障转移（leader哨兵执行）
- 筛选合格候选从节点
  - 过滤条件：排除失联、故障、与原主断开时间过长的从节点
  - 优先级顺序：
    - replica‑priority 数值越小优先级越高
    - 优先级相同，复制偏移量 master_repl_offset 越大，数据越新优先
    - 偏移量相同，取 run‑id 字典序最小节点兜底
- 选出新主服务器
  - Leader 发送`replicaof no one`，该从节点脱离复制，升级为主节点
  - Leader发送INFO，确认该从节点的role已经切换成master了
- 修改从服务器复制目标
  - 向其余从节点发送`replicaof 新主IP 端口`，跟随新 master 做复制
  - parallel‑syncs 控制并发同步从节点数量
- 旧主节点恢复后变成从服务器
  - 旧 master 网络恢复，哨兵自动下发 replicaof 命令，降级为新主的从节点
- 事件广播通知客户端
  - 通过发布订阅频道`+switch‑master`推送主节点变更事件；客户端收到更新连接地址
    \### 脑裂问题
- 因为网络分区，客户端还能写入旧主，但是旧主已经和整个集群隔离，导致写入了旧主和新主两份数据
- 解决方案：`min‑replicas‑to‑write N`，master 检测可用从节点数量小于 N，则拒绝接收写请求
  \## 集群
- Redis 官方分布式分片方案，实现数据水平拆分，多节点分担数据
- 核心：**哈希槽 slot**，一共 16384 个槽位，所有 key 映射到 0‑16383 slot
- 所有槽都有节点在处理时集群上线，否则认定为下线
- 节点只能使用0号数据库，节点还会用clusterState中的slots_to_keys来保存槽和键之间的关系，score是槽号，member是键名
- 至少需要 6 个节点：3 主 3 从，每个主节点分配一部分槽位，从节点做备份，不支持链式复制
- 主节点负责读写；从节点复制对应主节点数据，支持故障转移
- 集群数据结构：

``` cpp
// 集群全局状态，每个Redis Cluster实例一份
typedef struct clusterState {
    // 当前本机节点
    clusterNode *myself;

    // key: node-id(40字节字符串), value: clusterNode*
    dict *nodes;

    // ---------------- slot路由核心数组 0~16383 ----------------
    // slots[N]：槽N归属哪个主节点，O(1)路由查找
    clusterNode *slots[16384];

    // 槽迁移：源节点视角，slot正在迁出到哪个节点
    clusterNode *migrating_slots_to[16384];
    // 槽迁移：目标节点视角，slot正从哪个节点迁入
    clusterNode *importing_slots_from[16384];

    // 集群状态标记：OK / FAIL
    int flags;

} clusterState;


// 集群中单个节点描述（主/从都用这个结构体）
typedef struct clusterNode {
    // 40位16进制唯一node‑id
    char name[40];

    // ip + port
    char ip[32];
    int port;

    // 角色、状态位标记
    // CLUSTER_NODE_MASTER / CLUSTER_NODE_SLAVE / PFAIL / FAIL
    int flags;

    // 如果是从节点：指向自己的master；主节点为NULL
    struct clusterNode *slaveof;

    // 如果是主节点：链表保存自己所有从节点；从节点为NULL
    list *slaves;

    // ---------------- 位图：本节点拥有哪些slot ----------------
    // 16384 bit = 2048字节，bit置1代表本节点持有该slot
    // 用于gossip消息广播对外宣告自己的槽位
    unsigned char slots[16384 / 8];

    // PFAIL主观下线时间戳
    mstime_t pfail_time;

    // gossip网络时间戳：last_ping, last_pong
    mstime_t ping_sent;
    mstime_t pong_received;

} clusterNode;


// Gossip消息报文，节点之间传播拓扑
typedef struct clusterMsg {
    // 发送方节点信息快照
    clusterNode sender;

    // 附带一批其他节点状态（部分节点，不是全量）
    clusterNodeEntry *known_nodes;
    int known_nodes_count;

} clusterMsg;
```

- 寻找目标节点：

  - slot 计算公式：`CRC16(key) % 16384`
  - 是本节点就处理
  - 不然向客户端返回MOVED错误，携带根据clsterState的slots找到的对应节点
  - 客户端往正确节点重新发送

- MOVED / ASK 重定向：

  - MOVED 重定向
    - key 对应的 slot 不在当前访问节点，返回 MOVED 槽 ip:port
    - 客户端更新本地缓存拓扑，下次直接访问正确节点
    - 常态访问、slot 已经稳定分配时返回 MOVED
  - ASK 重定向
    - **槽正在迁移过程中**，部分 key 还在源节点，部分已经到目标节点
    - 客户端收到 ASK，访问目标节点，发送ASKING 临时标记命令，再发送操作 key的命令
    - ASK 只本次生效，**不更新客户端本地缓存拓扑**
      \### 启动初始化

- `cluster‑enabled yes`：开启集群模式，`cluster‑node‑timeout`：节点超时判定时间

- `cluster‑config‑file nodes‑xxx.conf`：节点自动维护该文件，保存集群拓扑、节点 id、slot 分配

- 手动使用`CLUSTER MEET <ip> <port>`节点握手分配集群，或使用`redis‑cli --cluster create ip1:port ip2:port ...`自动发起握手，使用gossip 流言协议，节点之间互相传播集群拓扑信息（所有节点 id、ip 端口、角色、持有 slot、从节点关系），节点之间定期 gossip 消息交换，拓扑可能不完整，MOVED协助完整

- 各节点生成 node‑id 唯一标识，持久化到 nodes.conf
  \### 消息类型
  \#### 基础 Gossip 消息

- PING（type=0）：心跳探测存活，携带 gossip 片段（自己看到的部分节点状态：PFAIL、角色、槽）

- PONG（type=1）：响应 PING、MEET；主动广播 PONG 快速同步拓扑

- MEET（type=2）：`CLUSTER MEET`命令触发，接收方**必须把发送者加入集群 nodes 字典**
  \#### 其他消息

- FAIL（type=3）： 当节点判定对方达到**客观下线 FAIL**，向集群广播 FAIL 消息

- PUBLISH（type=4）：集群模式下，客户端 publish 消息，节点收到执行并广播此消息

- FAILOVER_AUTH_REQUEST（type=5）：从节点发起故障转移选举，向其他主节点请求投票

- FAILOVER_AUTH_ACK（type=6）：主节点返回同意投票应答

- UPDATE（type=7）：发现别人有更新的槽配置，推送槽位更新通知
  \### 槽指派

- 通过CLUSTER ADDSLOTS可以手动槽指派

- clusterNode节点中的slots数组和numslots记录了节点负责处理哪些槽，位图形式表示

- 槽指派传播：将自己的slots数组发给别人，更新别人的clusterState中的自己的clusterNode信息
  \### 重新分片

- 以 slot 为单位迁移，不是单条 key

- 工具：redis‑cli --cluster 整套工具脚本，简化扩容缩容操作

- 迁移过程：

  - 源节点：持有该 slot，同时迁移 key 数据到目标节点
  - 迁移中间状态：slot 处于 migrate 状态
    - key 还在源节点 → 请求正常在源执行
    - key 已经搬走 → 检查clusterState的migrating_slots_to，返回 ASK 重定向
  - 全部 key 迁移完成，正式把 slot 归属修改为目标节点
    \### 故障转移

- 设置从节点：

  - 打开`cluster‑enabled yes`，此时是空白独立集群节点
  - 执行`CLUSTER MEET master_ip port`，将该节点加入集群拓扑
  - 从节点执行：`CLUSTER REPLICATE <master‑node‑id>`
    - 当前节点修改自身`clusterNode->flags`，标记为`CLUSTER_NODE_SLAVE`
    - 设置`clusterNode->slaveof`指针指向 master 的 clusterNode
    - master 节点的`clusterNode->slaves`链表把该从节点加入
    - 底层自动走 PSYNC2 复制流程，同步 master 全部数据
    - 通过 Gossip 协议，把主从关系扩散给集群所有节点

- 故障检测：

  - 主观下线PFAIL：单个节点PING探测对端超时，单方面标记疑似故障并通过PING传播
  - 客观下线FAIL：收到超过半数节点确认 PFAIL，标记客观下线并FAIL广播给集群

- 故障转移：

  - 主节点客观下线，如果有可用从节点：
    - 推举新主节点：集群中有投票权的主节点向这个从节点投票
    - 新主升级完成，撤销所有旧主的槽指派，将接管全部原主的 slot
    - 剩余从节点改为复制新主
    - 向集群广播一条PONG消息，声明自己为主节点，同步拓扑
    - 正常通信
  - 主节点客观下线，无可用从节点：对应 slot 全部不可用，集群进入`CLUSTER‑DOWN`整体停止服务
    \# 性能优化
    \## Linux 系统

- 关闭透明大页 `transparent_hugepage=never`

- `vm.overcommit_memory=1`，防止 fork 失败

- ulimit 文件句柄调大，tcp‑backlog 调大
  \## 业务架构

- 读写分离：主写从读，注意复制延迟

- Cluster 集群：单实例数据控制 10‑20G，主节点必须配从库

- 热点 key：本地缓存、拆分 key，避免单节点压力过载

- 缓存问题：穿透 / 击穿 / 雪崩做业务防护

- 监控重点指标：QPS、慢查询、内存碎片率、连接数、复制偏移、CPU 使用率
  \## Redis 配置

- lazy‑free 系列开启，大 key 删除 / 过期 / 淘汰后台释放，主线程不阻塞

- Redis6 + 开启`activedefrag yes`主动整理内存碎片

- AOF：`appendfsync everysec`；避免`always`

- RDB：避开业务高峰，从节点执行备份，主库少 bgsave

- 设置合理`maxclients`、空闲连接`timeout`
  \## 命令

- 禁用阻塞命令：`keys、hgetall、smembers`，改用`*scan`迭代

- 批量命令 /mget/mset + pipeline，减少网络往返

- 避免大 key、大 value；大集合拆分；key 设置过期时间

- Lua 脚本简短，禁止耗时脚本
  \## 客户端

- 使用连接池，不要频繁建销毁短连接

- 设置读写超时，清理僵死连接

- 尽量同机房部署，降低网络延迟
