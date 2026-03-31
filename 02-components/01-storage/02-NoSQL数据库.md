# NoSQL 数据库

---

## 为什么需要 NoSQL

关系型数据库在以下场景遇到瓶颈：
- 数据结构多变，schema 频繁调整
- 超大规模数据需要水平扩展
- 极高的写入吞吐量（Throughput）需求
- 数据模型天然不是表格结构（如图、文档、时序）

NoSQL（Not Only SQL）通过放宽一致性要求（BASE），换取更高的可扩展性和灵活性。

---

## Redis

**类型：** 键值存储（Key-Value Store）/ 内存数据库（In-Memory Database）

**核心特点：**
- 数据存在内存中，读写速度极快（微秒级延迟）
- 支持丰富的数据结构：String、Hash、List、Set、Sorted Set、Bitmap、HyperLogLog
- 支持持久化（Persistence）：RDB（快照）和 AOF（追加日志）两种方式
- 支持发布订阅（Pub/Sub）、Lua 脚本、事务（有限）

**典型使用场景：**
- 缓存（Cache）：最主流用途
- 会话存储（Session Store）
- 排行榜（Leaderboard）：Sorted Set 天然支持
- 分布式锁（Distributed Lock）：SETNX 命令
- 限流（Rate Limiting）：滑动窗口计数
- 消息队列：Stream 数据类型

**集群模式：**
- 主从复制（Master-Replica）：读写分离
- Redis Sentinel：自动故障转移（Failover）
- Redis Cluster：数据分片，支持水平扩展

---

## Cassandra

**类型：** 宽列存储（Wide-Column Store）

**核心特点：**
- 分布式、去中心化（Peer-to-Peer），无单点故障
- 写入性能极强，适合海量数据写入
- 可调节一致性级别（Tunable Consistency）：ONE、QUORUM、ALL
- 数据模型以分区键（Partition Key）和聚集键（Clustering Key）组织

**典型使用场景：**
- 时序数据（Time-Series Data）：IoT、传感器数据
- 消息历史、日志存储
- 需要跨地域多活（Multi-Region）的系统

**注意：** Cassandra 不支持 JOIN，查询模式（Access Pattern）需要在建表时就设计好。

---

## MongoDB

**类型：** 文档数据库（Document Database）

**核心特点：**
- 以 BSON（Binary JSON）文档存储，结构灵活，无固定 Schema
- 支持嵌套文档（Nested Document）和数组
- 支持丰富的查询语法和聚合管道（Aggregation Pipeline）
- 支持副本集（Replica Set）和分片（Sharding）

**典型使用场景：**
- 内容管理系统（CMS）
- 用户画像、产品目录（字段多变）
- 快速迭代的业务（schema 经常变化）

---

## 各 NoSQL 对比速查

| 数据库 | 类型 | 一致性 | 适用场景 |
|---|---|---|---|
| Redis | 键值/内存 | 强（单节点）/ 最终（集群） | 缓存、排行榜、分布式锁 |
| Cassandra | 宽列 | 可调节（最终为主） | 海量写入、时序、多活 |
| MongoDB | 文档 | 强（副本集默认） | 灵活 schema、内容管理 |
| DynamoDB | 键值/文档 | 最终（可选强） | AWS 生态、Serverless |
| Neo4j | 图 | 强 | 社交关系、推荐、知识图谱 |

---

## 小结

> NoSQL 不是关系型数据库的替代品，而是补充。选型关键看数据模型和访问模式：需要灵活 schema 用 MongoDB，需要极速读写用 Redis，需要海量写入用 Cassandra。
