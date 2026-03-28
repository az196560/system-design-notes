# API 网关（API Gateway）

---

## 什么是 API 网关

API 网关是系统对外的统一入口（Single Entry Point），所有客户端请求都先经过网关，再由网关路由到对应的后端服务。在微服务（Microservices）架构中，API 网关是不可或缺的基础组件。

**代表产品：** AWS API Gateway、Kong、Nginx（兼做）、Traefik、Apigee、阿里云 API 网关

---

## 核心功能

### 请求路由（Request Routing）
根据请求路径、请求头、方法等将请求转发到对应的微服务：
- `/api/users/*` → 用户服务
- `/api/orders/*` → 订单服务
- `/api/products/*` → 商品服务

支持蓝绿部署（Blue-Green Deployment）和金丝雀发布（Canary Release）：按比例将流量分配到不同版本的服务。

### 认证与授权（Authentication & Authorization）
在网关统一处理身份验证，后端服务无需重复实现：
- JWT（JSON Web Token）验证
- OAuth 2.0 授权流程
- API Key 校验

### 限流（Rate Limiting）
对不同客户端、不同接口设置请求频率上限，防止滥用和保护后端服务。

### 负载均衡（Load Balancing）
将同一服务的请求分发到多个实例（Instance），与反向代理功能重叠。

### SSL 终止（SSL Termination）
在网关层处理 HTTPS 加密解密，后端服务之间可以用 HTTP 通信，简化证书管理。

### 请求/响应转换（Transformation）
修改请求头、请求体格式，做协议转换（如 REST → gRPC）。

### 日志与监控（Logging & Monitoring）
统一收集所有请求的访问日志、延迟、错误率，便于观测和排查问题。

---

## API 网关 vs 反向代理 vs 负载均衡

| 组件 | 主要职责 | 工作层级 |
|---|---|---|
| 负载均衡器 | 分发流量到多个实例 | L4/L7 |
| 反向代理 | 代理请求，隐藏后端 | L7 |
| API 网关 | 上述所有 + 认证限流等业务逻辑 | L7 |

API 网关是功能最丰富的一层，通常包含了反向代理和负载均衡的能力。

---

## BFF（Backend for Frontend）模式

当不同客户端（Web、iOS、Android）需要不同格式的数据时，可以为每类客户端建立专属的 BFF 网关层，做数据聚合和裁剪，避免前端反复拼接多个服务的数据。

```
Web App   → BFF-Web   →
iOS App   → BFF-iOS   → 各微服务
Android   → BFF-Android →
```

---

## 常见问题

**单点故障（Single Point of Failure）：** 网关本身需要高可用部署（多实例 + 负载均衡）。

**性能瓶颈：** 所有流量过网关，网关本身的吞吐量上限决定系统极限，需要水平扩展。

**过度集中逻辑：** 避免把太多业务逻辑放在网关层，网关应保持轻量，复杂业务逻辑留在服务内部。

---

## 小结

> API 网关是微服务架构的"门卫"，统一处理横切关注点（Cross-Cutting Concerns），让后端服务专注于业务逻辑。关键是保持网关轻量，避免成为系统瓶颈。
