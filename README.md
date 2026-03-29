# 📚 System Design 学习笔记

基于 [s09g System Design Foundation](https://maven.com/s09g/system-design-foundation) 课程体系整理的系统设计学习笔记，中文为主，重点英文概念括号标注。

---

## 📁 目录结构

```
system-design-notes/
├── 01-fundamentals/          # 基础概念
├── 02-components/            # 核心组件
│   ├── 01-storage/
│   ├── 02-networking/
│   ├── 03-messaging/
│   ├── 04-service-governance/
│   └── 05-observability/
└── 03-system-categories/     # 经典系统设计题（按系统类型分类）
    ├── 01-数据密集系统/
    ├── 02-异步处理系统/
    ├── 03-即时通信系统/
    ├── 04-派生数据系统/
    ├── 05-数据检索系统/
    ├── 06-一致性系统/
    ├── 07-分层架构/
    └── 08-高频补充/
```

---

## 01 基础概念（Fundamentals）

| 笔记 | 核心考点 |
|---|---|
| 网络与通信 | HTTP/HTTPS、REST vs gRPC、WebSocket、DNS |
| CAP 定理与一致性模型 | CAP 三角、强一致 vs 最终一致 |
| ACID 与 BASE | 事务四大属性、隔离级别、NoSQL 设计哲学 |
| 分布式事务 | 2PC、Saga 模式、本地消息表 |
| 扩展性与负载均衡 | 垂直/水平扩展、一致性哈希、Auto Scaling |
| 缓存策略 | Cache-Aside/Write-Through、LRU、缓存三大问题 |
| 高可用与容错 | 限流算法、熔断器三态、重试退避 |
| 数据存储基础 | SQL vs NoSQL 选型、索引、分片、读写分离 |

---

## 02 核心组件（Components）

### 存储
| 组件 | 要点 |
|---|---|
| 关系型数据库 | MySQL/PostgreSQL、ACID、索引优化 |
| NoSQL 数据库 | Redis、Cassandra、MongoDB 选型对比 |
| 对象存储 | S3、存储类分级、预签名 URL、生命周期管理 |
| 搜索引擎 | Elasticsearch、倒排索引、相关性评分、ELK Stack |

### 网络
| 组件 | 要点 |
|---|---|
| CDN | 边缘节点、回源策略、缓存失效 |
| API 网关 | 路由、认证限流、BFF 模式 |
| 反向代理 | Nginx、负载均衡、SSL 终止 |

### 消息
| 组件 | 要点 |
|---|---|
| 消息队列 & Pub/Sub | Kafka vs RabbitMQ、投递语义、幂等消费 |
| 任务队列 | Celery、重试机制、定时任务、工作流 |
| 流处理 | Flink、Kafka Streams、窗口、Watermark |

### 服务治理
| 组件 | 要点 |
|---|---|
| 服务发现与注册 | Consul/Eureka、客户端 vs 服务端发现 |
| 配置中心 | Apollo/Nacos、动态推送、Feature Flag |
| 分布式锁 | Redis SETNX、Redlock、ZooKeeper |

### 可观测性
| 组件 | 要点 |
|---|---|
| 日志系统 | ELK Stack、结构化日志、TraceID 关联 |
| 监控与告警 | Prometheus + Grafana、四个黄金信号、SLO/SLA |
| 链路追踪 | Jaeger、Span/Trace、OpenTelemetry、采样策略 |

---

## 03 经典系统设计题（System Categories）

每篇包含：**问题定义 → High-Level 架构图（Mermaid）→ 核心组件详解 → 关键 Trade-off → 小结**

### 01 数据密集系统
- **Design TikTok** — 视频上传/转码、CDN 分发、推荐 Feed

### 02 异步处理系统
- **Design Webhook System** — 可靠投递、指数退避重试、熔断
- **Design Job Scheduler** — DAG 调度、分布式锁防重、时间轮
- **Design Notification System** — 多渠道路由、去重防骚扰

### 03 即时通信系统
- **Design 1-1 Chat (WhatsApp)** — WebSocket、消息有序性、离线兜底
- **Design Group Chat (Slack)** — 读扩散、Pub/Sub 大群广播、未读计数
- **Design Live Comment** — 百万连接、流量采样、内容安全

### 04 派生数据系统
- **Design Monitoring & Alert System** — TSDB、降采样、告警管理器
- **Design Ad Event Aggregation** — Flink 流处理、Lambda/Kappa 架构、计费去重
- **Design Driver Heatmap** — GeoHash、流处理聚合、供需调度

### 05 数据检索系统
- **Design Game Scoreboard** — Redis Sorted Set、亿级排名、多维度榜单
- **Design POI / Yelp** — GeoHash/QuadTree、两阶段距离过滤
- **Design URL Shortener** — Base62 编码、301 vs 302、缓存设计
- **Design Autocomplete** — Trie 树、节点 Top N 预缓存、离线热度更新

### 06 一致性系统
- **Design Payment (Stripe)** — 幂等键、支付状态机、PSP 超时处理、对账
- **Design Cloud Storage (Dropbox)** — 分块上传、增量同步、冲突解决
- **Design Auction System** — 乐观锁 CAS、实时价格推送、拍卖定时结束
- **Design Flash Sale** — 多层流量过滤、Redis 原子库存预扣、超时释放
- **Design Charity Event** — 高并发计数、分桶计数、资金托管

### 07 分层架构
- **Design CI/CD (GitHub Actions)** — Pipeline DAG、容器隔离、日志实时流
- **Design CDN (Cloudflare)** — 三层拓扑、Anycast、缓存失效传播
- **Design Web Crawler** — URL Frontier、Bloom Filter 去重、robots.txt

### 08 高频补充
- **Design Online IDE** — MicroVM 安全隔离、CRDT 协作、WebSocket 终端
- **Design Model Distribution** — GPU 推理优化、Continuous Batching、弹性扩缩容

---

## 🛠️ 使用建议

1. **初学者：** 先读完 `01-fundamentals` 打好基础，再进入 `02-components`
2. **备考面试：** 直接按 `03-system-categories` 的系统类型分类刷题，同类题对比复习
3. **Mermaid 图渲染：** 推荐用 [Obsidian](https://obsidian.md/)、VS Code（Markdown Preview Mermaid Support 插件）或 Typora 查看架构图

---

## 📖 参考资料

- [System Design Foundation by s09g](https://maven.com/s09g/system-design-foundation)
- [Designing Data-Intensive Applications - Martin Kleppmann](https://dataintensive.net/)
- [System Design Interview - Alex Xu](https://www.amazon.com/System-Design-Interview-insiders-Second/dp/B08CMF2CQF)
- [ByteByteGo Newsletter](https://blog.bytebytego.com/)
