# ACID 与 BASE

---

## ACID

ACID 是关系型数据库（Relational Database）事务（Transaction）的四个保证：

### 原子性（Atomicity）
事务中的所有操作要么全部成功，要么全部回滚（Rollback）。不存在"做了一半"的状态。

> 例：银行转账，扣款和入账必须同时成功，否则全部撤销。

### 一致性（Consistency）
事务执行前后，数据库始终处于合法状态，满足所有约束（Constraint）和规则。

> 例：账户余额不能变成负数。

### 隔离性（Isolation）
并发执行的事务互不干扰，每个事务感觉像是独立运行的。

**隔离级别从低到高：**

| 隔离级别 | 脏读（Dirty Read） | 不可重复读（Non-repeatable Read） | 幻读（Phantom Read） |
|---|---|---|---|
| 读未提交（Read Uncommitted） | 可能 | 可能 | 可能 |
| 读已提交（Read Committed） | 不会 | 可能 | 可能 |
| 可重复读（Repeatable Read） | 不会 | 不会 | 可能 |
| 串行化（Serializable） | 不会 | 不会 | 不会 |

隔离级别越高，性能越低。MySQL InnoDB 默认是可重复读（Repeatable Read）。

### 持久性（Durability）
事务一旦提交（Commit），数据就永久保存，即使系统崩溃也不会丢失。通常通过预写日志（WAL, Write-Ahead Log）实现。

---

## BASE

BASE 是 NoSQL 数据库和分布式系统的设计哲学，是对 ACID 的放宽，换取更高的可用性和可扩展性：

### 基本可用（Basically Available）
系统保证可用性，但允许部分降级（Degradation）。即使某些节点故障，系统仍能响应请求（可能返回旧数据或部分数据）。

### 软状态（Soft State）
系统的状态可能随时间变化，即使没有新的输入。中间状态是被允许的，数据不要求实时一致。

### 最终一致性（Eventually Consistent）
系统保证：在没有新写入的情况下，所有副本（Replica）最终会收敛到一致状态。

---

## ACID vs BASE 对比

| 对比项 | ACID | BASE |
|---|---|---|
| 一致性要求 | 强一致 | 最终一致 |
| 可用性 | 较低（加锁等待） | 高 |
| 适用系统 | 关系型数据库（MySQL、PostgreSQL） | NoSQL（Cassandra、DynamoDB、MongoDB） |
| 适用场景 | 金融、支付、订单 | 社交、推荐、日志、购物车 |

---

## 小结

> ACID 保证了数据绝对可靠，但限制了扩展性。BASE 牺牲了部分一致性，换来了高并发和高可用。选型时核心问题是：**业务能不能接受短暂的数据不一致？**
