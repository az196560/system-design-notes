# 📚 System Design 学习笔记

系统设计 + AI 基础设施面试笔记，涵盖后端基础、ML/LLM 八股、核心组件、经典系统设计题和 AI Infra 专题。中文为主，重点英文概念括号标注。

---

## 📁 目录结构

```
system-design-notes/
├── 01-fundamentals/              # 基础概念
│   ├── backend/                  #   后端系统基础（8 篇）
│   └── ml-and-llm/              #   ML / LLM 基础八股（7 篇）
├── 02-components/                # 核心组件（15 篇）
│   ├── 01-storage/
│   ├── 02-networking/
│   ├── 03-messaging/
│   ├── 04-service-governance/
│   └── 05-observability/
├── 03-system-categories/         # 经典系统设计题
│   ├── 01-数据密集系统/
│   ├── 02-异步处理系统/
│   ├── 03-即时通信系统/
│   ├── 04-派生数据系统/
│   ├── 05-数据检索系统/
│   ├── 06-一致性系统/
│   ├── 07-分层架构/
│   ├── 08-高频补充/
│   └── 09-AI基础设施/            #   AI Infra 系统设计题（14 篇）
│       ├── 01-核心训练基础设施/
│       ├── 02-模型服务/
│       ├── 03-数据与特征/
│       ├── 04-MLOps与平台/
│       ├── 05-向量检索与RAG/
│       └── 06-高阶方向/
├── 04-case-studies/
└── 05-references/
```

---

## 01 基础概念（Fundamentals）

### 后端系统基础

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

### ML / LLM 基础八股

| 笔记 | 核心考点 |
|---|---|
| 机器学习基础 | 损失函数（CE/MSE）、优化器（SGD/Adam）、正则化（L1/L2/Dropout）、评估指标（P/R/F1/AUC） |
| 深度学习基础 | 激活函数（ReLU/GELU/SwiGLU）、反向传播与梯度消失、BN vs LN vs RMSNorm、残差连接、CNN/RNN/LSTM |
| Transformer 与注意力机制 | Self-Attention QKV 计算、Multi-Head Attention、位置编码（RoPE/ALiBi）、KV Cache、FlashAttention、GQA/MQA |
| LLM 基础 | Tokenization（BPE）、自回归预训练、Scaling Law、解码策略（Top-P/Temperature）、In-Context Learning、幻觉 |
| LLM 训练技术 | SFT、RLHF（PPO）、DPO、LoRA/QLoRA、Adapter、数据配比与课程学习 |
| LLM 推理优化 | 量化（GPTQ/AWQ/SmoothQuant）、Speculative Decoding、PagedAttention、FlashAttention、CUDA Graph、Kernel Fusion |
| Embedding 与向量检索基础 | 余弦相似度 vs L2、HNSW 多层图搜索、IVF 聚类加速、Product Quantization 压缩 |

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
- **Design Google Flights** — 预计算+实时混合搜索、中转路径图搜索、价格缓存 TTL、灵活搜索
- **Design Expedia** — 多供应商聚合适配层、Saga 跨供应商预订补偿、两阶段验价、打包定价
- **Design Ticketmaster** — 虚拟排队削峰、座位锁定+超时释放、Redis 原子防超卖、黄牛防控
- **Design Recommendation System** — News Feed / 短视频 / 好友三场景，多路召回 + 多层排序 + 多目标建模
- **Design Harmful Content Detection** — 同步快检+异步深检、哈希库防复发、多模态检测、HITL 反馈闭环
- **Design Bot Detection** — 网络/浏览器/行为/业务多信号融合、GBDT+图聚类、静默降级对抗

### 09 AI 基础设施

#### 核心训练基础设施
- **Design Distributed Training System** — 数据/模型/流水线并行、AllReduce vs Parameter Server、3D Parallelism
- **Design GPU Cluster Scheduler** — Gang Scheduling、拓扑感知调度、抢占与碎片管理
- **Design Model Checkpoint System** — 异步 Checkpoint（GPU→CPU→SSD→S3）、分布式 Resharding

#### 模型服务
- **Design LLM Serving System** — Prefill/Decode 分离、PagedAttention、Continuous Batching、KV Cache 管理
- **Design Inference Gateway** — Token-Aware 负载均衡、多级降级、灰度发布、Token 限流计费
- **Design Embedding Service** — 请求聚合与动态 Batching、Sorted Batching、Embedding 缓存

#### 数据与特征
- **Design Feature Store** — Online/Offline 双存储、Point-in-Time Join、Training-Serving Skew 消除
- **Design Training Data Pipeline** — 多级预取、分片 Shuffle、数据去重与质量过滤、Data Mixing
- **Design Data Labeling Platform** — Golden Set QA、Inter-Annotator Agreement、主动学习

#### MLOps 与平台
- **Design ML Pipeline** — DAG 编排、容器化步骤、Artifact 血缘追踪、缓存与断点续跑
- **Design Model Registry and Deployment** — ML CI/CD、金丝雀发布、Shadow Traffic、秒级回滚
- **Design Experiment Tracking System** — 高频指标采集、时序数据库存储、实验对比与可复现性

#### 向量检索与 RAG
- **Design Vector Database** — HNSW/IVF-PQ 索引、分布式分片、混合查询（Metadata + Vector）
- **Design RAG System** — Chunking 策略、混合检索（向量 + BM25）、Reranker 精排、幻觉控制

#### 高阶方向
- **Design GPU Memory Manager** — 显存池化、PagedAttention 分页、多租户隔离（MIG）、Offloading 分层
- **Design Multimodal Serving Platform** — 统一多模态 API、异构硬件调度、多模型 DAG 编排
- **Design AI Agent Orchestration** — Agent Loop、Tool Calling 安全控制、多 Agent 协作、状态与记忆管理

---

## 🛠️ 使用建议

1. **后端方向：** `01-fundamentals/backend` → `02-components` → `03-system-categories/01~08`
2. **AI Infra 方向：** `01-fundamentals/ml-and-llm` → `03-system-categories/09-AI基础设施`
3. **全面备考：** 两条路线都走一遍，AI Infra 面试通常也会考传统系统设计
4. **Mermaid 图渲染：** 推荐用 [Obsidian](https://obsidian.md/)、VS Code（Markdown Preview Mermaid Support 插件）或 Typora 查看架构图

---

## 📖 参考资料

- [System Design Foundation by s09g](https://maven.com/s09g/system-design-foundation)
- [Designing Data-Intensive Applications - Martin Kleppmann](https://dataintensive.net/)
- [System Design Interview - Alex Xu](https://www.amazon.com/System-Design-Interview-insiders-Second/dp/B08CMF2CQF)
- [ByteByteGo Newsletter](https://blog.bytebytego.com/)
