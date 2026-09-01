---
title: "数据库基础系列-MySQL"
slug: "mysql"
date: 2026-09-01T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["MySQL", "数据库"]
categories: ["技术笔记"]
summary: "MySQL基础内容，包括数据库、表、表模式、主键、外键、DML、快照读、当前读、事务、MVCC等。"
toc: true
comments: true
description: "MySQL"
---

# 数据库基础

- 数据库：物理集合，包含文件、多套 schema、用户权限
- 关系数据库三大范式：
  - 第一范式 1NF 原子性：列不可再分
  - 第二范式 2NF 消除部分函数依赖：非主键字段完全依赖主键
  - 第三范式 3NF 消除传递函数依赖：非主键字段不能依赖其他非主键字段，必须直接完全依赖主键
- 数据库类型：
  - 关系型：强表模式，支持事务
    - MySQL
    - pgsql
    - Oracle
    - sql server
    - SQLite
  - 非关系型：灵活结构，弱事务
    - 键值对：Redis
    - 文档：MongoDB
    - 图：Neo4j
- 模式：对象的容器，逻辑命名空间，存放表、索引、视图、函数
  - 数据库模式：数据库的元数据：存储引擎、字符集、排序规则、表注释、字段信息
  - 表模式：**一张表的结构元数据**：字段名、类型、约束、索引、默认值、注释，不包含表里的业务数据
- 主键：标识每一行，必须有且唯一，不能为NULL，不能更新不能重用，不能使用可能更改的值
- 外键：**保证参照完整性**，让一张表的字段引用另一张表的主键 / 唯一键，约束数据关系
- DML：insert/update/delete操作
- 快照读`select`：读取数据的快照版本，不加行锁，读写互不阻塞
- 当前读`select … for update`：读取**数据库最新物理版本的数据**，会加锁（行锁、间隙锁、临键锁），读取最新已提交数据
  - select ... for update;
  - select ... lock in share mode;
  - DML操作
    \# MySQL原理
- MySQL架构图
  - Connectors 客户端连接层
  - MySQL‑Server 服务层（引擎共用）
    - **连接池 Connection Pool**：连接管理、线程分配、身份认证、权限校验、连接数限制
    - **SQL Interface SQL 接口**：接收 SQL（DML/DDL/ 存储过程 / 视图触发器），返回结果
    - **Parser 解析器**：词法语法分析，生成解析树，校验 SQL 语法
    - **Optimizer 查询优化器**：成本模型，选择索引、join 顺序，生成**执行计划**
    - **Executor 执行器**：按照执行计划调用存储引擎 API 读取数据
    - **Caches & Buffers**：表缓存、权限缓存；**8.0 已移除 Query Cache 查询缓存**
    - **Management Service**：备份恢复、复制集群、元数据管理；**binlog 二进制日志就在 Server 层**
  - Pluggable Storage Engines 可插拔存储引擎层
  - File System 文件系统磁盘层
    ![640](../mysql/f194e0fa22259340e3ef9308e21f32c6fbb87904.png)
- 引擎：InnoDB、MyISAM、Memory，生产项目一律使用InnoDB
- 外键不能跨引擎
- `\G：`{=tex}内容横向列表格式输出，可以代替分号
- 引擎对比

| 对比项 | InnoDB | MyISAM | Memory(Heap) |
|----|----|----|----|
| 事务 | ✅支持，完整 ACID 事务 | ❌不支持事务 | ❌不支持事务 |
| 外键 | ✅支持外键约束 | ❌语法兼容，但约束无效 | ❌不支持外键 |
| 锁粒度 | **行锁 + 表锁**；默认行级锁；有表级意向锁 | **只有表锁**，DML 全表锁定 | **表锁** |
| 日志 | redo log、undo log、binlog；崩溃安全恢复 | 只有 binlog；崩溃可能损坏数据，无崩溃恢复机制 | 无持久化日志，重启全部数据丢失 |
| 表结构文件 | *.frm（表定义）<br><br>*.ibd（独立表空间，数据 + 索引） | *.frm（表定义）<br><br>*.MYD（数据）<br><br>\*.MYI（索引） | 仅 \*.frm，没有数据磁盘文件 |
| 索引类型 | B + 树聚簇索引；主键和数据在一起；二级索引存主键值 | B + 树非聚簇；数据与索引分开存储 | B‑Tree / Hash 索引可选，全部数据内存存放 |
| COUNT (\*) 无 where | 不会缓存总行数，需要扫描统计 | ✅缓存表总行数，`COUNT(*)`直接返回缓存值 | 内存维护计数，速度很快 |
| 重启自增 ID 记录 | ✅持久化，重启后自增 ID 不变 | ❌内存保存，重启自增 ID 重置为 max (id)+1 | ❌表清空重启，自增重置为 1 |
| 数据持久化 | 磁盘持久化，崩溃不丢数据 | 磁盘存储，但崩溃易损坏 | 全部在内存，**重启数据清空** |
| 适合场景 | 默认引擎；业务表、高并发读写、需要事务 | 只读 / 静态历史数据，极少更新；现在基本淘汰 | 临时表、高速缓存，不能存重要业务数据 |

## 正则表达式

- LIKE是简单模糊查询，优先使用LIKE
- `RLIKE`等价`REGEXP`；**不是完整 PCRE 正则，语法有裁剪**
- MySQL 字符串里`\`是转义，正则里`\`要写**双反斜杠`\\`**
- 元字符表：

| 符号    | 含义                     | 示例                         |
|---------|--------------------------|------------------------------|
| `^`     | 开头                     | `^a` 以 a 开头               |
| `$`     | 结尾                     | `z$` 以 z 结尾               |
| `.`     | 任意单个字符（不含换行） | `a.b`                        |
| `[]`    | 字符集合                 | `[0‑9]`数字，`[a‑z]`小写字母 |
| `[^]`   | 取反集合                 | `[^0‑9]`非数字               |
| `*`     | 前面单元 0 次或多次      | `ab*` a 后面 b 任意个        |
| `+`     | 前面单元 1 次或多次      | `ab+` a 后面至少 1 个 b      |
| `?`     | 前面单元 0 或 1 次       | `ab?` b 可选                 |
| `{m}`   | 恰好 m 次                | `[0‑9]{11}`11 位数字         |
| `{m,n}` | m\~n 次                  | `[0‑9]{6,8}`6‑8 位数字       |
| `\|`    | 或                       | `a\|b`匹配 a 或 b            |
| `()`    | 分组                     | `(ab)+`ab 重复               |

# 数据库模式

## 字符集

- 类型：
  - latin1：单字节，西欧字符，不支持中文
  - gbk：中文，双字节；只支持简体繁体，不支持 emoji
  - utf8：MySQL 内部阉割版本，**最多 3 字节，不能完整存 emoji 表情**，不要用
  - utf8mb4（首选）：真正完整 UTF‑8，1‑4 字节，支持中文、emoji、所有 Unicode 字符；
- 排序规则 collation
  - `_ci`：case‑insensitive **大小写不敏感**
  - `_cs`：case‑sensitive 大小写敏感
  - `_bin`：二进制比较，直接按字节对比，区分大小写
- 查看系统字符集参数

``` sql
SHOW VARIABLES LIKE '%character%';
SHOW VARIABLES LIKE '%collation%';
```

- 工程推荐：
  - utf8mb4_unicode_ci：通用，推荐默认
  - utf8mb4_general_ci：速度略快，少数语言排序精度略差
  - utf8mb4_bin：二进制，区分大小写
    \## 数据类型
    \### 整数
- TINYINT：1 字节，范围‑128\~127；无符号 0\~255，适合状态、开关字段
- SMALLINT：2 字节，范围‑32768\~32767
- MEDIUMINT：3 字节
- INT：4 字节，常用 id、普通数字；范围‑2¹³¹ \~ 2¹³¹‑1
- BIGINT：8 字节，业务主键 id**推荐优先使用**，防止溢出
- UNSIGNED：无符号修饰，只存非负数，最大值翻倍
  \### 浮点
- FLOAT：4 字节浮点数，存在精度丢失，不要存金额
- DOUBLE：8 字节浮点数，依然有精度误差
- DECIMAL (M,D)：定点数，**金额必须用这个**，M 总有效位数，D 小数位数
  \### 字符串
- CHAR (N)：定长字符串，不足 N 位自动补空格；查询去掉尾部空格；适合固定长度编码、手机号
- VARCHAR (N)：变长字符串，**业务最常用**；N 代表==字符数==，utf8mb4 中文 1 字符占 3‑4 字节；会额外 1‑2 字节记录长度；节省空间
- TEXT：大文本，最大 65535 字符；不能有默认值；索引只支持前缀索引，适合文章内容
- MEDIUMTEXT / LONGTEXT：超大文本，日志、长篇内容，尽量少用
  \### 日期时间
- DATE：日期，格式`yyyy‑mm‑dd`，只存年月日，3 字节
- TIME：时间，`hh:mm:ss`，时分秒
- DATETIME：日期 + 时间 `yyyy‑mm‑dd hh:mm:ss`；8 字节，**业务首选**，不受时区影响，范围 1000‑9999
- TIMESTAMP：4 字节；受时区影响；时间上限 2038‑01‑19，新项目尽量不用
- YEAR：年份，只存年
- ON UPDATE CURRENT_TIMESTAMP：只有 DATETIME/TIMESTAMP 支持，修改行自动刷新时间
  \### 二进制
- BINARY / VARBINARY：二进制字节流，存字节数据，区分大小写，不做字符集转换
- BIT (M)：位类型，存 0/1 比特位，较少业务使用
  \### 枚举集合
- ENUM ('a','b','c')：枚举，只能取列表中一个值；内部存数字，节省空间
- SET ('x','y','z')：集合，可以多选多个值；业务极少用
  \### 选型
- 主键 id：BIGINT AUTO_INCREMENT
- 状态标记：TINYINT UNSIGNED
- 金额：DECIMAL (10,2)
- 短文本、名称：VARCHAR (合理长度)
- 长文本文章：TEXT
- 时间戳：DATETIME DEFAULT CURRENT_TIMESTAMP
- 更新时间：DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
  \## 表创建 CREATE TABLE
- 字段定义：字段名 数据类型 \[约束\] \[COMMENT 注释\]
  - 约束：
    - PRIMARY KEY：主键，非空且唯一，一张表只能一个主键
    - AUTO_INCREMENT：自增属性，仅整数主键 / 唯一键可用
    - NOT NULL：字段不允许存 NULL
    - DEFAULT：设置字段默认值
    - UNIQUE KEY：唯一索引，字段不能重复，可以允许 NULL
    - KEY：普通 B‑Tree 索引
- 表选项：存储引擎ENGINE、字符集DEFAULT CHARSET、排序规则COLLATE、表注释COMMENT

``` sql
CREATE TABLE user (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '主键ID',
    name VARCHAR(50) NOT NULL COMMENT '姓名',
    age TINYINT DEFAULT NULL COMMENT '年龄',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    UNIQUE KEY uk_name (name) COMMENT '姓名唯一索引',
    KEY idx_age (age) COMMENT '年龄普通索引'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户表';
```

- 避免表已存在报错

``` sql
CREATE TABLE IF NOT EXISTS user(
    id INT PRIMARY KEY
)ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

- 复制表结构不复制数据

``` sql
-- 只复制结构，不带数据
CREATE TABLE user_bak LIKE user;
```

- 复制表结构和数据

``` sql
CREATE TABLE user_bak AS SELECT * FROM user;
```

## 表更新 ALTER TABLE

- 重命名表 RENAME

``` sql
ALTER TABLE user RENAME TO user_info;
--简写
ALTER TABLE user RENAME user_info;
```

- 修改表选项（引擎、字符集、注释）

``` sql
-- 修改表注释
ALTER TABLE user COMMENT = '用户信息表';

-- 修改字符集和排序规则
ALTER TABLE user CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

- 添加外键（==现代工程不建议用外键==）
  - CASCADE（少用）：级联
    - ON DELETE CASCADE：父表记录删除，子表关联记录同步删除
    - ON UPDATE CASCADE：父表主键更新，子表外键自动跟着更新
  - SET NULL：置空，父表删除 / 更新，子表外键字段设为 NULL
  - RESTRICT（默认）：限制拒绝，父表有子表关联数据时，不允许删除 / 更新父表记录
  - NO ACTION：和 RESTRICT 几乎一样，阻止风险操作

  ``` sql
  ALTER TABLE `order`
  ADD CONSTRAINT fk_order_user FOREIGN KEY(user_id) REFERENCES user(id)
  ON DELETE CASCADE
  ON UPDATE CASCADE;
  ```
- 删除外键

``` sql
--查看约束名称
SHOW CREATE TABLE `order`;

ALTER TABLE `order` DROP FOREIGN KEY fk_order_user;

--外键配套索引，手动删除
ALTER TABLE `order` DROP INDEX fk_order_user;
```

- 添加字段 ADD

``` sql
-- 在表末尾新增字段
ALTER TABLE user ADD COLUMN age TINYINT NOT NULL DEFAULT 0 COMMENT '年龄';

-- 指定位置：FIRST 放在第一列
ALTER TABLE user ADD COLUMN nick VARCHAR(20) FIRST;

-- AFTER col_name：放在某个字段后面
ALTER TABLE user ADD COLUMN phone VARCHAR(11) AFTER name;
```

- 修改字段 MODIFY

``` sql
ALTER TABLE user MODIFY COLUMN age SMALLINT NOT NULL DEFAULT 1 COMMENT '用户年龄';
```

- 重命名字段 CHANGE

``` sql
ALTER TABLE user CHANGE COLUMN old_name new_name VARCHAR(50) NOT NULL COMMENT '新名称';
```

- 删除字段 DROP COLUMN

``` sql
ALTER TABLE user DROP COLUMN phone;
```

- 添加普通索引

``` sql
ALTER TABLE user ADD INDEX idx_age(age);
```

- 添加唯一索引

``` sql
ALTER TABLE user ADD UNIQUE KEY uk_phone(phone);
```

- 删除索引

``` sql
ALTER TABLE user DROP INDEX idx_age;
```

- 添加主键

``` sql
ALTER TABLE user ADD PRIMARY KEY(id);
```

- 删除主键

``` sql
ALTER TABLE user DROP PRIMARY KEY;
```

- 一次性多个修改（逗号分隔）

``` sql
ALTER TABLE user
ADD COLUMN email VARCHAR(100),
MODIFY COLUMN age TINYINT DEFAULT 0,
DROP COLUMN old_col;
```

## 表删除 DROP TABLE

- 单表删除

``` sql
DROP TABLE user;
```

- 多表删除

``` sql
DROP TABLE IF EXISTS table_a, table_b, table_c;
```

- 安全删除

``` sql
DROP TABLE IF EXISTS user;
```

# 增删改查

## 登录

``` sql
mysql -u root -p pwd -h localhost -p 3306
help show;
exit;
```

## 元信息查看

``` sql
# 登录后切换数据库
USE test_db;

-- 会话级：当前这一个数据库连接的状态
SHOW SESSION STATUS;
SHOW STATUS; -- 默认SESSION
-- 列出所有数据库
SHOW DATABASES;
SHOW CREATE TABLE t;        -- 看表完整定义、引擎、外键
-- 当前库所有表
SHOW TABLES;
-- 指定库.表
DESC test_db.user;
-- 查看用户权限
SHOW GRANTS FOR 'dev'@'%';
-- 查看警告（Warning + Note + Error）
SHOW WARNINGS;
-- 只看错误
SHOW ERRORS;

SHOW INDEX FROM t;          -- 看索引
SHOW FULL PROCESSLIST;      -- 看正在执行SQL
SHOW ENGINE INNODB STATUS;  -- innodb详细状态，锁、事务
SHOW VARIABLES LIKE '%xxx%' -- 查配置参数
SHOW TABLE STATUS;          -- 看表磁盘大小、行数
```

## 查询

``` sql
SELECT  [DISTINCT] 列表达式
FROM 表名
[JOIN 关联表 ON 关联条件]
[WHERE 行过滤条件]
[GROUP BY 分组列]
[HAVING 分组后过滤]
[ORDER BY 排序字段]
[LIMIT 偏移,行数];
```

- 实际执行顺序：FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT
  \### SELECT 数据列
- 完全限定名：库名.表名.列名
- DISTINCT：去重
- AS：别名
- 表达式：`+-*/`
- 函数：
- 字符串拼接函数：
  - 聚合函数：与GROUP BY联合使用
  - 文本函数：`CONCAT(name,':',age)`、`SUBSTRING(name,1,2)`、`TRIM(name)`、`REPLACE(name,'a','A')`、`UPPER(name)`、`LENGTH(name)`、`LEFT(name,2)`
  - 日期函数：`NOW()`、`DATE_FORMAT(create_time,'%Y‑%m‑%d %H:%i:%s')`、`CURDATE()`、`CURTIME()`、`DATE(create_time)`、`TIME(create_time)`、`YEAR(create_time),MONTH(create_time),DAY(create_time)`
  - 数学函数：`ABS(-10)`、`ROUND(3.1415,2)`、`MOD(10,3)`、`CEIL(3.2)`、`RAND()`
    \`\`\` sql
    -- 查询全部列（生产禁止select *）
    SELECT * FROM user;

-- 指定列
SELECT id,name,age FROM user;

-- 列别名 AS，AS可以省略
SELECT id, name AS username, age user_age FROM user;

-- DISTINCT 去重
SELECT DISTINCT age FROM user;

-- 表达式、函数
SELECT id, age+10, UPPER(name) FROM user;

    ### FROM 来源
    - 数据源可以是表、视图、子查询
    ``` sql
    SELECT * FROM user;
    SELECT * FROM test_db.user; -- 指定库.表

    -- 子查询作为数据源（派生表）
    SELECT * FROM (SELECT id,name FROM user) AS t;

### JOIN 多表连接

- INNER JOIN：内连接，只返回两边匹配上的数据
- LEFT JOIN：左连接，左边表全部保留，右表匹配不到填 NULL
- RIGHT JOIN（工程禁用）：右连接，右表全部保留，左表匹配不到填 NULL

``` sql
SELECT u.name,o.amount
FROM user u
INNER JOIN `order` o 
ON u.id = o.user_id;
```

### ON 关联过滤

- join 关联条件，连接时过滤，where是join 完成之后，对结果集过滤
- ==LEFT JOIN/RIGHT JOIN必须用ON==

``` sql
SELECT u.name,o.amount
FROM user u
LEFT JOIN `order` o ON u.id = o.user_id;
```

### WHERE 行过滤

- 支持运算符：`> < >= <= = != <> AND OR IN NOT IN LIKE IS NULL REGXP`
- LIKE支持元字符：
  - `%`：匹配任意 0 个或多个字符
  - `_`：匹配**恰好 1 个**任意字符
    \`\`\` sql
    -- \> 大于
    SELECT \* FROM user WHERE age \> 22;

-- \< 小于
SELECT \* FROM user WHERE age \< 28;

-- \>= 大于等于
SELECT \* FROM user WHERE age \>= 22;

-- \<= 小于等于
SELECT \* FROM user WHERE age \<= 28;

-- = 等于
SELECT \* FROM user WHERE name = 'Alice';

-- != 不等于
SELECT \* FROM user WHERE age != 22;

-- \<\> 不等于（和!=等价，SQL标准写法）
SELECT \* FROM user WHERE age \<\> 22;

-- AND 并且，多个条件同时满足
SELECT \* FROM user WHERE age\>20 AND age\<30;

-- OR 或者，满足其一即可
SELECT \* FROM user WHERE age\<20 OR age\>30;

-- IN 在集合内
SELECT \* FROM user WHERE id IN (1,2,3);

-- NOT IN 不在集合内
SELECT \* FROM user WHERE id NOT IN (1,2);

-- LIKE 模糊匹配 %任意字符 \_单个字符
SELECT \* FROM user WHERE name LIKE 'A%'; --A开头
SELECT \* FROM user WHERE name LIKE '\_ac%'; --第二位是a

-- IS NULL 判断为空（⚠️不能用 = NULL）
SELECT \* FROM user WHERE phone IS NULL;

-- IS NOT NULL 判断非空
SELECT \* FROM user WHERE phone IS NOT NULL;

-- REGEXP 正则匹配
SELECT \* FROM user WHERE name REGEXP '\^J'; --J开头

    ### GROUP BY 分组
    - 聚合函数：`COUNT() SUM() AVG() MAX() MIN()`
    - 可以在SELECT语句中使用聚合函数，对`*`使用计算包括NULL，对列名使用计算不包括NULL
    ``` sql
    -- 按年龄分组，统计每组人数
    SELECT age, COUNT(*) AS cnt 
    FROM user 
    GROUP BY age;

### HAVING 分组后过滤

- GROUP BY之后过滤聚合结果，可以使用别名

``` sql
SELECT age, COUNT(*) cnt
FROM user
GROUP BY age
HAVING cnt >= 2;
```

### ORDER BY 排序

- 默认升序：ASC，降序：DESC

``` sql
-- ASC升序（默认），DESC降序
SELECT * FROM user ORDER BY id DESC;

--多字段排序，先按age，age相同按id
SELECT * FROM user ORDER BY age DESC, id ASC;
```

### LIMIT 分页

- `n,m` 等价于 `LIMIT n OFFSET m` 表示跳过m后取n条
  \## 结果集合并
- `UNION ALL`：直接拼接，不去重，**推荐优先用**
- `UNION`：合并并自动去重，性能差

``` sql
SELECT id,name FROM user
UNION ALL
SELECT id,name FROM user_bak;
```

## 子查询

- 必须使用完全限定名
- WHERE 后子查询
  - 标量子查询：返回单个值，`WHERE id = (SELECT MAX(id) FROM user);`
  - IN 子查询：返回**单列多行**，`` WHERE id IN (SELECT DISTINCT user_id FROM `order`); ``
  - NOT IN子查询：返回单列多行，`` WHERE id NOT IN (SELECT user_id FROM `order`); ``
  - ANY/ALL子查询：ANY满足子查询任意一个值、ALL满足子查询全部值，`WHERE age > ANY(SELECT age FROM user WHERE id<3); WHERE age > ALL(SELECT age FROM user WHERE id<3);`
  - EXISTS / NOT EXISTS：子查询有返回行则条件成立，**不关心返回什么字段，一般写 SELECT 1**，`WHERE EXISTS (SELECT 1 FROM`order`o WHERE o.user_id = u.id);`
- FROM 后子查询：子查询返回结果集作为临时表使用，必须给临时表取别名

``` sql
SELECT t.name FROM (SELECT id,name FROM user WHERE age>18) AS t;
```

- SELECT 后标量子查询：不推荐使用，嵌套查询次数极多，优先用JOIN

``` sql
SELECT 
  id,
  name,
  (SELECT COUNT(*) FROM `order` o WHERE o.user_id = u.id) AS order_cnt
FROM user u;
```

## 表联结

- 旧写法：where 外键 = 主键，等价 INNER JOIN，没有 ON 关键字，全部条件写 WHERE
- INNER JOIN 内连接：只返回两边匹配成功的数据，两边条件同时满足才输出

``` sql
SELECT u.name,o.amount
FROM user u
INNER JOIN `order` o
ON u.id = o.user_id;
```

- LEFT JOIN 左外连接：**左边表全部保留**，右表匹配不到的字段填充 NULL

``` sql
SELECT u.name,o.amount
FROM user u
LEFT JOIN `order` o
ON u.id = o.user_id;
```

- RIGHT JOIN 右外连接：**右边表全部保留**，左表匹配不到填充 NULL

``` sql
SELECT u.name,o.amount
FROM user u
RIGHT JOIN `order` o
ON u.id = o.user_id;
```

- 交叉连接 CROSS JOIN：笛卡尔积，A 表 m 行 \* B 表 n 行，返回 m \* n 行

``` sql
SELECT * FROM user CROSS JOIN `order`;
```

- 多表联结：可以连续 JOIN，每次联结从左到右生成中间结果集

``` sql
SELECT u.name,o.amount,g.goods_name
FROM user u
LEFT JOIN `order` o ON u.id=o.user_id
LEFT JOIN goods g ON o.goods_id=g.id;
```

- 模拟 FULL JOIN（无原生）：左右两表全部数据都保留，匹配不上填 NULL

``` sql
SELECT u.name,o.amount FROM user u LEFT JOIN `order` o ON u.id=o.user_id
UNION
SELECT u.name,o.amount FROM user u RIGHT JOIN `order` o ON u.id=o.user_id;
```

## 插入

- 单行插入 INSERT VALUES

``` sql
INSERT INTO user(id,name,age) VALUES(1,'julian',29);
```

- 多行插入 INSERT VALUES

``` sql
INSERT INTO user(name,age)
VALUES
('bob',25),
('jack',31),
('lily',20);
```

- 少量插入 INSERT SET

``` sql
INSERT INTO user SET name='tom',age=27;
```

- 查询结果批量插入 INSERT SELECT

``` sql
INSERT INTO user_bak(id,name,age)
SELECT id,name,age FROM user WHERE age>18;
```

- 冲突处理
  - INSERT IGNORE：忽略当前记录
  - ON DUPLICATE KEY UPDATE：更新当前记录
    \`\`\` sql
    INSERT IGNORE INTO user(id,name) VALUES(1,'julian');

INSERT INTO user(id,name,age) VALUES(1,'julian',30)
ON DUPLICATE KEY UPDATE name='julian',age=30;

    ## 更新
    - 严禁不带WHERE的全表更新，小心更新
    - 单表更新
    ``` sql
    UPDATE 表名
    SET 列1 = 值1, 列2 = 值2
    [WHERE 条件];

- 限制更新行数

``` sql
UPDATE user
SET age=22
WHERE age<18
ORDER BY id
LIMIT 5;
```

- 多表更新

``` sql
UPDATE user u
INNER JOIN `order` o ON u.id = o.user_id
SET u.age = 25
WHERE o.amount > 100;
```

- 忽略行错误（严禁）

``` sql
UPDATE IGNORE user SET name=null WHERE id>10;
```

## 删除

- 严禁不带WHERE的删除，小心删除
- 不允许删除有外键关联的行
- 单值删除：UPDATE将值设置为NULL
- 单表删除

``` sql
DELETE FROM user WHERE id = 1;
```

- 限制删除行数

``` sql
DELETE FROM user
WHERE age < 18
ORDER BY id
LIMIT 10;
```

- 多表删除

``` sql
DELETE u,o
FROM user u
INNER JOIN `order` o ON u.id = o.user_id
WHERE u.age < 16;
```

- 忽略行错误（严禁）

``` sql
DELETE IGNORE FROM user WHERE id>100;
```

- 删除全表

``` sql
TRUNCATE TABLE user;
```

# 索引

- 本质：基于[B+树](../3.数据结构/常用数据结构.md#B+树)组织行，加速查找（随机IO变成了顺序IO），降低DML性能，占用额外空间
- InnoDB一张表只有一个聚簇索引（主键索引），其余都是非聚簇索引，所以表结构文件只有.idb又保存数据又保存索引
- MyISAM全部都是非聚簇索引，所以表结构文件有.MYD保存数据，.MYI保存索引
- 聚簇索引与非聚簇索引：
  - 聚簇索引：B+树叶子节点有整行数据，索引数据一体
  - 非聚簇索引：B+树叶子节点是指针（主键）和本列的值，不保存整行的值
  - 回表查询：非聚簇索引索引到主键然后根据主键到聚簇索引查找返回完整行
  - 索引覆盖：查询所需要的全部字段，都直接能从二级索引里面拿到，不需要去聚簇索引回表读取行数据
- 分类：
  - 主键索引 PRIMARY：唯一、非空；一张表只能一个聚簇索引
  - 普通索引 INDEX：插入索引本字段可以重复
  - 唯一索引 UNIQUE：插入索引本字段不可以重复，但允许 NULL
  - 联合索引 (复合索引)：`INDEX(a,b,c)`，多字段组合索引，遵循**最左前缀原则**
  - 全文索引 FULLTEXT：文本检索
  - Hash 索引：InnoDB 自适应哈希索引 AHI，内部自动维护，**不能手动创建 hash 索引**
- 最左前缀匹配规则：
  - 从左到右依次匹配，比如索引 `INDEX(a,b,c)`，`where b=? and c=?`索引就会失效，变成全局遍历
  - 联合索引中遇到范围条件（\> \< \>= \<= between），范围右边所有字段不再使用索引，只做结果过滤，所以等值字段放联合索引最左边
- 索引失效：
  - 索引字段使用函数运算或计算
  - 索引字段查询发生隐式类型转换
  - `like '%关键词'` 前缀百分号
  - or 一边条件没有索引
  - 联合索引违反最左前缀匹配
  - is not null / != / \<\> 大概率失效
  - 优化器主动放弃索引
    \## 普通/联合索引
    \`\`\` sql
    -- 创建单列普通索引
    CREATE INDEX idx_name ON t(name);

-- 创建联合普通索引（多字段）
CREATE INDEX idx_a_b ON t(a,b);

-- 删除索引
DROP INDEX idx_name ON t;

    ## 唯一/联合索引
    - 会比普通索引多一个唯一性检查，写入速度慢一点，但增加了唯一性约束防止脏数据（比如手机号不能重复）
    ``` sql
    --单列唯一索引
    CREATE UNIQUE INDEX uk_phone ON t(phone);

    --联合唯一索引：a+b组合不能重复，单独a或b可以重复
    CREATE UNIQUE INDEX uk_a_b ON t(a,b);

    DROP INDEX uk_phone ON t;

- 唯一索引允许多条记录字段为 NULL

``` sql
CREATE UNIQUE INDEX uk_email ON user(email);
--下面两条可以同时存在，不会冲突
insert into user(email) values(null);
insert into user(email) values(null);
```

## 全文索引

- 支持引擎：InnoDB、MyISAM
- 作用：针对大文本字段做词语检索，替代低效`LIKE '%xxx%'`
- 由于基于列索引，所以数据更新时会自动更新，在大量插入时不要建立FULLTEXT索引
- 最小词长：由于基于列索引，少于n个字符的词不会被索引
- 公共词忽略：自然模式下词语在超过50%的行存在会被当做公共词直接忽略
- 搜索模式：
  - IN NATURAL LANGUAGE MODE：自然语言模式，计算相关性得分，多个词语空格分隔，自动根据相关性大小降序
  - IN BOOLEAN MODE（推荐）：布尔模式，不计算相关性，精确控制词语规则，不排序
    - 无符号：该词可选，命中优先
    - +：必须包含该词
    - -：必须排除该词
    - \<：降低该词权重
    - `*`：词前缀通配，放在词尾部
    - ""：完整短语精确匹配

    ``` sql
    SELECT * FROM article
    WHERE MATCH(title,content) AGAINST ('+mysql -oracle' IN BOOLEAN MODE);
    ```
- 创建索引

``` sql
CREATE TABLE article(
  id INT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(200),
  content TEXT,
  FULLTEXT ft_idx_title_content(title,content)
)ENGINE=InnoDB;
```

- 已有表追加全文索引

``` sql
ALTER TABLE article ADD FULLTEXT INDEX ft_idx_title_content(title,content);
```

- MATCH AGAINST语法使用（WHERE后）

``` sql
-- 自然语言模式，按相关性分数排序，分数越高越匹配
SELECT id,title,content,MATCH(title,content) AGAINST ('java' IN NATURAL LANGUAGE MODE) AS score
FROM article
WHERE MATCH(title,content) AGAINST ('java' IN NATURAL LANGUAGE MODE)
ORDER BY score DESC;
```

- 查询扩展（自然语言模式）
  - 两次搜索，第一轮原始关键词搜索，将搜索的结果提取高频词汇追加到搜索词，再次搜索
    \`\`\` sql
    -- 简写
    SELECT \* FROM article
    WHERE MATCH(title,content) AGAINST('database' WITH QUERY EXPANSION);

--完整写法
SELECT \* FROM article
WHERE MATCH(title,content) AGAINST('database' IN NATURAL LANGUAGE MODE WITH QUERY EXPANSION);

    # 视图
    - 本质：**存储的 SELECT 查询语句**，虚拟临时表，没有物理存储，方便重用sql语句
    - 大量视图会导致性能下降，视图本身**不能建立索引**
    - 底层表结构变更（增减字段），视图可能失效
    - 更新视图实际修改原始表，==视图应当尽量用于检索==
    - 应用：生成统计字段、复杂表联结、过滤干扰行
    - 创建视图
    ``` sql
    CREATE VIEW v_user_order AS
    SELECT u.id, u.name, o.amount, o.create_time
    FROM user u
    LEFT JOIN `order` o ON u.id = o.user_id;

- 覆盖视图

``` sql
CREATE OR REPLACE VIEW v_user_order AS
SELECT u.id,u.name FROM user u;
```

- 查看视图

``` sql
-- 查看视图定义SQL
SHOW CREATE VIEW v_user_order;

-- 使用视图，和普通表一样查询
SELECT * FROM v_user_order WHERE amount>100;
```

- 修改视图内容

``` sql
ALTER VIEW v_user_order AS
SELECT u.id,u.name,u.age FROM user u;
```

- 删除视图

``` sql
DROP VIEW IF EXISTS v_user_order;
```

- 更新视图（不建议使用，单表、所有 NOT NULL 字段可见、非复杂查询）

``` sql
--单表简单视图，可以更新
CREATE VIEW v_simple_user AS
SELECT id,name,age FROM user;

UPDATE v_simple_user SET age=30 WHERE id=1;
```

- 视图更新保护 WITH CHECK OPTION
  - 限制通过视图修改 / 插入的数据，必须满足视图 WHERE 条件
    - WITH CASCADED CHECK OPTION：级联检查（包含底层视图条件，默认）
    - WITH LOCAL CHECK OPTION：只检查当前视图条件

    ``` sql
    CREATE VIEW v_adult_user AS
    SELECT id,name,age FROM user WHERE age >=18
    WITH CHECK OPTION;
    ```

    # 存储过程（高权）
- 本质：一组预编译好的 SQL 语句集合，调用时直接执行，类似脚本可以带参数、变量、分支、循环逻辑
- 创建
  - 参数类型：
    - IN：输入参数，调用传入值，过程内部可读，修改不影响外部（默认）
    - OUT：输出参数，过程内部赋值，调用方拿到结果
    - INOUT：输入输出，既可以传入，内部修改后带回外部

    ``` sql
    DELIMITER //
    CREATE PROCEDURE proc_get_count(IN p_age INT, OUT total INT)
    BEGIN
      SELECT COUNT(*) INTO total FROM user WHERE age >= p_age;
    END //
    DELIMITER ;
    ```
- 查看

``` sql
SHOW CREATE PROCEDURE proc_query_user;
```

- 调用

``` sql
--调用in参数
CALL proc_query_user(20);

--调用out参数，使用用户变量接收
CALL proc_get_count(18,@cnt);
SELECT @cnt;
```

- 删除

``` sql
DROP PROCEDURE IF EXISTS proc_query_user;
```

## 语法

- 注释：
  - 单行注释：双横杠 +**空格**，后面写注释
  - 多行注释：/\* \*/ 块注释
  - \#内容：单行，mysql 专属
  - COMMENT 'xxx'：表 / 字段元注释，持久化存储在数据字典
- 业务变量：@x，用户会话变量，会话全局有效，存储过程内外都能用
- 局部变量 DECLARE：必须写在BEGIN 最开头

``` sql
DELIMITER //
CREATE PROCEDURE proc_demo()
BEGIN
    DECLARE v_name VARCHAR(32);
    DECLARE v_cnt INT DEFAULT 0;
    
END //
DELIMITER ;
```

- 赋值

``` sql
SELECT name INTO v_name FROM user WHERE id=1;
```

- IF 判断

``` sql
IF v_age >=18 THEN
    --逻辑
ELSEIF v_age >=12 THEN
    --逻辑
ELSE
    --逻辑
END IF;
```

- CASE 分支

``` sql
CASE v_status
WHEN 1 THEN 
ELSE 
END CASE;
```

- WHILE 循环

``` sql
WHILE v_i <10 DO
    SET v_i = v_i +1;
END WHILE;
```

- REPEAT 循环

``` sql
REPEAT
    SET v_i = v_i +1;
UNTIL v_i >=10 END REPEAT;
```

- LEAVE / ITERATE
  - LEAVE：跳出循环，类似 break
  - ITERATE：直接下一轮，类似 continue
- LOOP while1循环

``` sql
标签名:LOOP
    --循环体SQL
    IF 退出条件 THEN
        LEAVE 标签名;
    END IF;
END LOOP 标签名;
```

## 游标

- **在存储过程 / 函数内逐行读取 SELECT 结果集**，用来循环处理每一行数据
- 只读前序遍历迭代器
- DECLARE 声明游标，DECLARE handler 声明异常处理器
- OPEN 打开游标
- FETCH 抓取一行数据到局部变量
- CLOSE 关闭游标

``` sql
DELIMITER //
CREATE PROCEDURE proc_cursor_demo()
BEGIN
    -- 1.先声明局部变量
    DECLARE v_id BIGINT;
    DECLARE v_name VARCHAR(32);
    DECLARE v_done INT DEFAULT 0;

    -- 2.声明游标，绑定查询结果集
    DECLARE cur_user CURSOR FOR
        SELECT id,name FROM user WHERE age >=18;

    -- 3.结束处理器：抓取不到数据时设置v_done=1
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_done = 1;

    -- 打开游标
    OPEN cur_user;

    -- 循环读取
    read_loop:LOOP
        FETCH cur_user INTO v_id, v_name;

        IF v_done = 1 THEN
            LEAVE read_loop; -- 退出循环
        END IF;

        -- 这里写单行业务逻辑
        SELECT v_id, v_name;
    END LOOP read_loop;

    -- 关闭游标
    CLOSE cur_user;
END //
DELIMITER ;

--调用
CALL proc_cursor_demo();
```

# 触发器（高权）

- 本质：绑定在表上的特殊存储程序，**表发生 INSERT / UPDATE / DELETE 时自动触发执行**，就是个回调函数呗
- 一张表**同一时机 + 同一事件只能有一个触发器**
- 内部不能insert/update/delete本表
- 触发器失败会导致原 SQL 整体回滚，每行都会触发，性能有巨大损耗，==工程实践很少用==
- 触发时机：BEFORE（操作之前）、AFTER（操作之后）
- 触发事件：INSERT、UPDATE、DELETE
- NEW / OLD 伪记录：变更前后的数据
- 粒度：MySQL 只有行级触发器FOR EACH ROW，没有语句级触发器
- 创建

``` sql
CREATE TRIGGER 触发器名
BEFORE/AFTER  INSERT/UPDATE/DELETE
ON 表名 FOR EACH ROW
BEGIN
    --触发器执行逻辑
END;
```

- 查看

``` sql
SHOW TRIGGERS;
SHOW CREATE TRIGGER tr_user_after_insert;
```

- 删除

``` sql
DROP TRIGGER IF EXISTS tr_user_after_insert;
```

# 事务处理

- 事务：一组操作作为一个原子单元，要么全部成功提交，要么全部回滚撤销
- InnoDB 支持事务；MyISAM 不支持事务，事务不能混用引擎
- 事务四大特性：
  - A 原子性：全部成功或全部失败
  - C 一致性：业务数据状态合法
  - I 隔离性：多个事务之间互相隔离
  - D 持久性：commit 之后修改永久保存
- 自动提交（每条 SQL 自动单独事务）

``` sql
--关闭自动提交，后续SQL需要手动commit
SET autocommit = 0;
--恢复默认自动提交
SET autocommit =1;
```

- 建立

``` sql
START TRANSACTION;
--等价 BEGIN;
```

- 保存点（事务内局部回滚，不用回滚整个事务）

``` sql
BEGIN;
UPDATE user SET age=20 WHERE id=1;
SAVEPOINT sp1;
UPDATE user SET age=30 WHERE id=2;

ROLLBACK TO sp1; --回滚到保存点sp1，id=2修改撤销，id=1修改还在事务内

COMMIT;
```

- 提交

``` sql
COMMIT;
```

- 回滚

``` sql
ROLLBACK;
```

## 事务隔离级别

- 脏读：A事务读到B事务还没提交的数据，B回滚A就读到脏数据
- 不可重复读：同一个事务A两次读同一行，中间被B事务修改提交，导致读到的结果不一样
- 幻读：同一个事务A范围查询，中间被B事务插入删除并提交，导致读到的前后行数不一样
- 事务隔离级别：

| 隔离级别 | 脏读 | 不可重复读 | 幻读 | 锁 / 实现机制 |
|----|----|----|----|----|
| **READ UNCOMMITTED<br>读未提交** | ✅允许脏读<br><br> | ✅允许 | ✅允许 | 无锁、无MVCC |
| **READ COMMITTED<br>读已提交 RC** | ❌禁止脏读<br><br> | ✅允许不可重复读<br><br> | ✅允许幻读 | MVCC，每次 select 生成新快照，行锁 |
| **REPEATABLE READ<br>可重复读 RR（默认）** | ❌禁止脏读 | ❌禁止不可重复读<br><br> | ❌InnoDB 通过**间隙锁 + 临键锁**解决幻读现象（SQL 标准仍然允许幻读） | MVCC，事务第一次 select 生成唯一快照，临键锁 <br><br> |
| **SERIALIZABLE<br><br>串行化** | ❌禁止脏读 | ❌禁止不可重复读 | ❌禁止幻读 | 全部变成当前读，临键锁 |

## MVCC

- 本质：同一行数据保存多个历史版本，不同事务读到自己快照对应的版本
- 行隐藏字段：
  - `DB_TRX_ID`：最近修改这条记录的事务 ID
  - `DB_ROLL_PTR`：回滚指针，指向 undo log 里旧版本数据，形成版本链
  - `DB_ROW_ID`：隐藏主键，表无主键时使用
- undo log（回滚日志）：
  - 修改数据时，把**修改前旧数据保存到 undo log**
  - 通过`DB_ROLL_PTR`把不同版本串联成**版本链**
  - undo log 不是永久保存，旧版本没有事务需要会被 purge 线程清理
- read‑view 读视图（快照）：
  - `m_ids`：当前活跃未提交事务 ID 集合
  - `min_trx_id`：m_ids 最小事务 id
  - `max_trx_id`：下一个将要分配的事务 id
  - `creator_trx_id`：当前事务自己 ID
- 可见性判断：`DB_TRX_ID` 和 Read‑View 对比
  - `trx_id < min_trx_id`：事务已经提交 → **可见**
  - `trx_id >= max_trx_id`：这个版本是快照之后才产生 → **不可见**
  - `min_trx_id ≤ trx_id < max_trx_id`
    - trx_id 在`m_ids`集合：事务还没提交 → **不可见**
    - 不在 m_ids：已经提交 → **可见**
  - 不可见的话顺着DB_ROLL_PTR找上一个历史版本
- 解决脏读：执行SELECT生成快照，没有提交的事务还在m_ids池子中，直接回溯到旧版本数据，避免脏读

| 时间 | 事务 A (RC) | 事务 B (trx_id=200) | 生成 Read‑View 与判断逻辑 |
|----|----|----|----|
| T1 | `begin;` | `begin;` | 无快照 |
| T2 |  | `update account set money=200 where id=1;`<br><br>**B 没有 commit** | 物理行变成 money=200，`DB_TRX_ID=200`；undo 保存旧版本 100 (trx_id=0) |
| T3 | `select money from account where id=1;` |  | RC 执行 select，立刻新建 RV，`m_ids={200}`<br><br>行`DB_TRX_ID=200`在`m_ids`活跃集合内 →**判定不可见**<br><br>顺着`roll_ptr`回溯 undo‑log，读到旧版本`100`返回✅不会读到 200（无脏读） |

- RC：每次执行SELECT都生成快照，第二次SELECT判定可见，直接读到数据，导致不可重复读

| 时间 | 事务 A (trx_id=100) RC | 事务 B (trx_id=200) | Read‑View 内容 (m_ids) | 底层行为 |
|----|----|----|----|----|
| T1 | `begin;` | `begin;` | 无 RV，未生成快照 | 仅开启事务，不创建 Read‑View |
| T2 | `select * from user where id=1;`<br><br>读到 Alice |  | RV1：`m_ids={200}`<br><br>min=200 max=201 creator=100 | 此时 B 还活跃；行`DB_TRX_ID=0`\<0RV1.min，可见，返回 Alice |
| T3 |  | `update ... set name='Bob'; commit;` |  | B 提交；物理行`DB_TRX_ID=200`；undo 保存 Alice 旧版本 |
| T4 | `select * from user where id=1;`<br><br>读到 Bob（不可重复读） |  | RV2：`m_ids={}`<br><br>min 无 max=201 creator=100 | **RC 每次 select 新建 RV**；B 已经提交不再活跃，`m_ids`为空<br><br>行`DB_TRX_ID=200`不在 m_ids，判定可见，直接读到 Bob |
| T5 | `commit;` |  | 销毁 RV2 | 事务结束 |

- RR：只有第一次的时候生成快照，第二次SELECT判定不可见，直接回溯到旧版本数据，避免不可重复读

| 时间 | 事务 A (trx_id=100) RR | 事务 B (trx_id=200) | Read‑View 内容 (m_ids) | 底层行为 |
|----|----|----|----|----|
| T1 | `begin;` | `begin;` | 无 RV | begin 不会生成快照 |
| T2 | `select * from user where id=1;`<br><br>读到 Alice |  | RV：`m_ids={200}`<br><br>min=200 max=201 creator=100 | **第一条 select 生成 RV，之后不再更换**<br><br>行`DB_TRX_ID=0`小于 min，可见，返回 Alice |
| T3 |  | `update ... set name='Bob'; commit;` | RV 保持不变：`m_ids={200}` | B 提交，但 A 的 RV**不会更新**，m_ids 依然保留当时快照时刻的活跃集合<br><br>物理行`DB_TRX_ID=200` |
| T4 | `select * from user where id=1;`<br><br>读到 Alice |  | RV 不变：`m_ids={200}` | 拿最新行`DB_TRX_ID=200`和 RV 对比：200 属于快照记录的活跃集合 m_ids<br><br>→判定**不可见**；顺着 roll_ptr 回溯 undo‑log 取旧版本 Alice |
| T5 | `commit;` |  | 销毁 RV | 事务结束 |

- MVCC不能阻止幻读！但是因为RR拿的是事务启动瞬间的快照，第二次SELECT判定不可见，直接回溯到旧版本数据，看不到这个插入，依旧返回旧数据进行操作，直接避开了幻读问题，但是这个插入是真实存在的并没有被MVCC阻止。
  \## 锁
- MyISAM 只有表锁；**InnoDB 支持表锁、行锁、间隙锁、临键锁**
- 行锁基于索引实现，没有用到索引，行锁会**升级为表锁**
- 表锁：
  - 表读锁（S‑lock）：其他事务可读，不可写；本事务也只能读
  - 表写锁（X‑lock）：本事务可写，其他事务读写全部阻塞
- 意向锁（表级）：
  - IS 意向共享：事务准备给某些行加 S 行锁，就先在全表加这个IS锁，便于其他表锁提前检查
  - IX 意向排他：事务准备给某些行加 X 行锁，就先在全表加这个IX锁，便于其他表锁提前检查
  - IX/IS 只和表锁冲突，同类锁不冲突，因为有可能改的不是同一行
- 行锁（Record Lock，记录锁）：加在**索引项**上，不是物理数据行，如果不走索引会每一行加行锁等于表锁了
  - 行共享锁 S：`select ... lock in share mode`
  - 行排他锁 X：`select ... for update` / update / delete
- 间隙锁 Gap Lock：锁住索引记录**之间的空隙**，防止其他事务在间隙插入新数据，解决幻读
  - 例如：索引存在 id=1，id=5，锁住间隙`(1,5)`：别人不能插入 2、3、4，但可以修改已有 1、5 行
- 临键锁 Next‑Key Lock = Record Lock + Gap Lock（RR默认）：既锁住现有行，又锁住前面间隙，**物理阻止幻读**
- 悲观锁实现：当前读

``` sql
-- 排他悲观锁，其他事务阻塞
select * from account where id=1 for update; 

-- 共享悲观锁
select * from account where id=1 lock in share mode;
```

- 乐观锁实现：版本号或时间戳，没变才成功更新，出现冲突需要重试，乐观觉得冲突的次数不多

``` sql
-- 查询，不加任何锁（快照读）
select id,balance,version from account where id=1;

-- 更新时带上版本条件：只有版本没变才更新成功
update account 
set balance = balance‑100, version = version + 1 
where id=1 and version = #{old_version};

update account set balance=balance‑100,update_time=now() 
where id=1 and update_time=#{old_update_time};
```

# 访问控制

- 用户账号：`'用户名'@'主机'`
- 账号最小权限原则：只给业务必须的`select/insert/update/delete`，不给`alter/drop`
- root 账号只允许`localhost`本地登录，禁止远程 root
- 创建用户
  - 主机部分不写默认为 `@'%'`，允许任意地址登录

  ``` sql
  CREATE USER 'app_user'@'192.168.1.%' IDENTIFIED BY 'MyPass@123';
  ```
- 查看用户

``` sql
SELECT user,host FROM mysql.user;
```

- 修改用户名

``` sql
RENAME USER '旧用户名'@'主机' TO '新用户名'@'主机';
```

- 删除用户

``` sql
DROP USER IF EXISTS 'app_user'@'192.168.1.%';
```

- 修改密码

``` sql
ALTER USER 'app_user'@'192.168.1.%' IDENTIFIED BY 'NewPass456!';
```

- 修改密码过期策略

``` sql
ALTER USER 'app_user'@'192.168.1.%' PASSWORD EXPIRE NEVER;
```

- 授权 GRANT
  - 权限：
    - SELECT：查询
    - INSERT：插入
    - UPDATE：更新
    - DELETE：删除
    - CREATE / ALTER / DROP：建表、改表、删表（DDL）
    - INDEX：索引操作
    - ALL PRIVILEGES：全部权限

    ``` sql
    GRANT 权限列表 ON 库.表 TO 用户@主机
    ```
- 查看权限

``` sql
--查看当前用户权限
SHOW GRANTS;

--查看指定用户权限
SHOW GRANTS FOR 'app_user'@'192.168.1.%';
```

- 刷新权限

``` sql
FLUSH PRIVILEGES;
```

- 删除权限

``` sql
REVOKE DELETE ON test.* FROM 'app_user'@'192.168.1.%';
```

# 主从复制

- 8.0后支持多线程重放提高回放速度
- 核心线程：
  - **主库：Binlog Dump Thread（dump 线程）**：事件写入binlog，从库连接过来后，把 binlog 发送给从库
  - 从库：IO 线程（IO‑thread）：连接主库，接收 binlog，写入从库本地**relay‑log（中继日志**
  - 从库：SQL 线程（SQL‑thread）：读取本地 relay‑log，重放里面的 SQL / 行变更，执行到从库数据库
- binlog格式：
  - STATEMENT（语句模式）：记录原始 SQL 语句
  - ROW（行模式，生产推荐）：不记 SQL，记录**修改前后每一行数据**，体积大
  - MIXED（混合）：安全 SQL 用 statement，危险 SQL 自动切换 row
- 同步策略：
  - 异步复制（默认）：主库写完 binlog 直接返回，**不等待从库接收**
  - 半同步复制 Semi‑Sync（rpl_semi_sync）：至少等待**一个从库收到 binlog（写到 relay‑log）**才返回
  - 全同步复制（NDB 集群才有，普通 InnoDB 不支持）：等待所有从库**完整执行完事务**主库才返回
    \# 维护
    \## 日志（增量备份）
- error_log 错误日志：记录启动、关闭、崩溃、严重错误

``` sql
SHOW VARIABLES LIKE 'log_error';
```

- slow_query_log 慢查询日志：记录执行超过`long_query_time`的 SQL

``` sql
SET GLOBAL slow_query_log=ON;
SET GLOBAL long_query_time=1; --秒

SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';
```

- binlog 二进制日志（工程常用）：记录所有操作，事务日志，用于恢复，配合mysqldump使用

``` sql
SHOW BINARY LOGS;
--查看binlog事件
SHOW BINLOG EVENTS IN 'mysql-bin.000001';
```

- general_log 通用查询日志：记录全部 SQL，开销极大，**只临时调试打开，平时关闭**
- redo log（InnoDB 引擎层）：记录数据页物理修改，断电重启后自动重放 redo 把数据恢复，不能直接看的
- undo log（InnoDB 引擎层）：MVCC基础，独立 undo 表空间，purge 线程后台回收，更是不能直接看的
  \## 表维护
- OPTIMIZE TABLE：整理碎片、回收删除释放的空间，重建索引

``` sql
OPTIMIZE TABLE user;
```

- ANALYZE TABLE：更新表统计信息，给优化器生成正确执行计划，速度快

``` sql
ANALYZE TABLE user;
```

- CHECK TABLE：检查表数据与索引完整性，检测损坏

``` sql
CHECK TABLE user;
```

- REPAIR TABLE：仅 MyISAM 修复损坏；**InnoDB 不要用**，InnoDB 靠崩溃恢复
  \## 备份恢复（全量备份）
  \### 逻辑备份恢复
- 逻辑备份 mysqldump
  - `--single‑transaction` InnoDB 一致性快照，不锁表

  ``` sql
  #全库
  mysqldump -u root -p --all‑databases > all.sql
  #单库
  mysqldump -u root -p testdb > testdb.sql
  #单表
  mysqldump -u root -p testdb user > user.sql
  ```
- 逻辑恢复

``` sql
mysql -u root -p testdb < testdb.sql
```

### 物理备份恢复

- xtrabackup：热备份，不需要锁表，适合大数据量；备份 ibd 物理文件，恢复速度远快 mysqldump
  \## 系统状态监测

``` sql
--当前会话
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST; --完整SQL

--InnoDB运行状态
SHOW ENGINE INNODB STATUS;

--缓冲池、连接数等全局状态
SHOW GLOBAL STATUS;
SHOW GLOBAL VARIABLES;
```

- 重点指标
  - Connections：总连接
  - Threads_connected：当前连接数
  - Innodb_buffer_pool_reads：物理读，过高说明缓冲池不够
  - Slow_queries：慢查询累计数量
    \## 死锁、长事务排查
- 长事务危害：undo 膨胀、锁持有、复制延迟
- 查看事务、锁等待：

``` sql
SHOW ENGINE INNODB STATUS
```

## 索引维护

- 查看当前索引

``` sql
SELECT * FROM sys.schema_unused_indexes;
```

# 性能改善

## 架构业务

- 应用连接池：控制连接数量
- 缓存：划分冷热数据，Redis/memcache 缓存热点数据
- 读写分离：读请求走从库，主库只负责写
- 分库分表：
  - 垂直分库：不同业务划分到不同数据库
  - 垂直分表：把大字段长文本拆出去成新表
  - 水平分库：多个库多张子表
  - 水平分表：一张过大的表拆成几个子表，子表结构完全一致仅名字略微不同，设置分片键
    - 分片规则：
      - 哈希分片：哈希函数分散
      - 范围分片：根据时间或ID区间
      - 复合分片：哈希+范围
    - 分布式难题：
      - 分布式主键不能依赖自增主键，雪花算法替代
      - 无法跨库Join：内存join、es做检索代替复杂join
      - 分页、排序复杂：游标分页
      - 分布式事务：最大努力通知 / 本地消息表
      - 数据扩容迁移：双写迁移
        \## 参数配置
- innodb_buffer_pool_size：最重要参数；物理内存 50%‑70%，缓存热数据与索引
- innodb_log_file_size：redo 日志，不要太小；一般 256M‑2G
- max_connections：不要盲目调很大
- long_query_time：开启慢查询日志
- innodb_flush_log_at_trx_commit：每次提交刷磁盘，最安全
- sync_binlog：binlog 刷盘策略
  \## 锁/事务优化
- 避免长事务
- 更新尽量按相同顺序，降低死锁
- 普通查询使用快照读，少用`select ... for update`排他锁
- 隔离级别：大部分业务 RC 也可以，最多 RR
  \## 表结构设计
- 字段尽量小
- 尽量NOT NULL
- 时间统一使用 DATETIME
- 字符集统一utf8mb4_unicode_ci
- 大文本 TEXT/BLOB 尽量拆分
- 主键优先 BIGINT 自增
  \## SQL语句优化
- 大查询分解为小查询
- 避免 join 大量表
- 不要返回\*，只返回需要的字段
- UNION ALL \> UNION
- LIMIT分页减少行数
- WHERE优化：
  - 少用!=
  - 少用 is null
  - 少用 or， 多用 union
  - 少用 in
  - 少用函数和计算
    \## 索引优化
- where 条件字段建立合适索引，避免全表扫描
- 联合索引最左前缀原则
- 尽量复合索引索引覆盖
- 定期清理从未使用索引
