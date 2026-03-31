# 消息队列与 Pub/Sub（Message Queue & Publish-Subscribe）

---

## 为什么需要消息队列

同步调用（Synchronous Call）的问题：
- 调用方必须等待被调用方响应，耦合紧密
- 被调用方故障时，调用方直接失败
- 瞬时流量高峰会压垮下游服务

消息队列（Message Queue）通过引入中间件，将生产者（Producer）和消费者（Consumer）解耦，实现异步通信（Asynchronous Communication）。

**核心优势：**
- **解耦（Decoupling）：** 生产者不需要知道谁在消费消息
- **削峰填谷（Traffic Shaping）：** 高峰期消息堆积在队列，消费者按自身能力处理
- **异步处理（Async Processing）：** 耗时任务放入队列，立即返回响应给用户
- **可靠投递（Reliable Delivery）：** 消息持久化，消费者故障重启后仍可消费

---

## 两种核心通信模型

### 点对点（Point-to-Point）
一条消息只被一个消费者消费，消费后从队列删除。适合任务分发，如发邮件、图片压缩任务。

### 发布订阅（Publish-Subscribe，Pub/Sub）
生产者将消息发布到主题（Topic），所有订阅该主题的消费者都能收到。适合事件广播，如订单创建后同时通知库存、物流、通知服务。

```
Point-to-Point:  Producer → Queue → Consumer（只有一个消费）

Pub/Sub:         Producer → Topic → Consumer A
                                  → Consumer B
                                  → Consumer C
```

---

## Kafka

Apache Kafka 是目前最主流的分布式消息流平台，兼具消息队列和流处理能力。

### 核心概念

- **Topic（主题）：** 消息的分类，类似数据库的表
- **Partition（分区）：** Topic 被拆分为多个分区，分布在不同 Broker 上，实现并行处理
- **Offset：** 消息在分区内的序号，消费者通过 Offset 追踪消费进度
- **Consumer Group（消费者组）：** 同一组内的消费者共同消费一个 Topic，每个分区只被组内一个消费者消费；不同消费者组之间独立消费（Pub/Sub 模式）
- **Broker：** Kafka 服务节点
- **Replication Factor：** 每个分区的副本数，保证高可用

### Kafka 的核心特性

**高吞吐量：** 顺序磁盘写入（Sequential Disk I/O）+ 零拷贝（Zero Copy），单机可达百万 QPS。

**消息持久化：** 消息写入磁盘，可配置保留时间（Retention Period），支持消费者回溯（Replay）历史消息。

**强顺序保证：** 同一分区内的消息严格有序；跨分区不保证顺序。

**消费者主动拉取（Pull）：** 消费者自主控制消费速率，不会被推送压垮。

### 消息投递语义（Delivery Semantics）

| 语义 | 说明 | 风险 |
|---|---|---|
| At Most Once（最多一次） | 消息可能丢失，不重复 | 丢消息 |
| At Least Once（至少一次） | 消息不丢失，可能重复 | 重复消费 |
| Exactly Once（恰好一次） | 不丢不重复，实现复杂 | 性能开销 |

实际生产中最常用 **At Least Once**，配合幂等性（Idempotency）设计消费者，处理重复消息。

---

## RabbitMQ

传统消息队列，基于 AMQP（Advanced Message Queuing Protocol）协议，适合复杂路由场景。

**与 Kafka 的核心区别：**

| 对比项 | Kafka | RabbitMQ |
|---|---|---|
| 消息模型 | 日志（Log），消费后不删除 | 队列（Queue），消费后删除 |
| 吞吐量 | 极高（百万级） | 较高（万级） |
| 顺序保证 | 分区内有序 | 队列内有序 |
| 消息回溯 | 支持 | 不支持 |
| 路由灵活性 | 简单 | 复杂路由（Exchange）|
| 适用场景 | 大数据流、日志、事件溯源 | 业务消息、任务队列、复杂路由 |

---

## 常见使用场景

- **异步处理：** 用户注册后异步发送欢迎邮件/短信
- **订单流程：** 下单成功后广播事件，库存、物流、积分各自消费
- **日志收集：** 各服务将日志发送到 Kafka，集中存储和分析
- **削峰：** 秒杀活动中将请求写入队列，消费者匀速处理

---

## 小结

> 消息队列是解耦和异步处理的利器。Kafka 适合高吞吐、需要消息回溯的场景；RabbitMQ 适合业务消息和复杂路由。设计时注意**幂等性**，保证消息重复消费不会造成数据错误。
