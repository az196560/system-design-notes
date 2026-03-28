# CDN（Content Delivery Network，内容分发网络）

---

## 什么是 CDN

CDN 是由分布在全球各地的边缘节点（Edge Node / Point of Presence, PoP）组成的网络。它将内容缓存到离用户最近的节点，用户请求时直接从就近的边缘节点获取，而不需要每次都回源（Origin）到中心服务器。

**代表产品：** Cloudflare、AWS CloudFront、Akamai、Fastly、阿里云 CDN

---

## 核心工作原理

1. 用户请求 `cdn.example.com/image.jpg`
2. DNS 解析返回离用户最近的边缘节点 IP
3. 边缘节点检查是否有缓存（Cache Hit）：
   - **命中（Hit）：** 直接返回缓存内容，极低延迟
   - **未命中（Miss）：** 回源（Origin Pull）获取内容，缓存后再返回给用户
4. 后续同区域用户请求同一资源时直接命中缓存

---

## 主要优势

**降低延迟（Latency Reduction）：** 用户从就近节点获取内容，物理距离短，传输延迟小。

**减轻源服务器压力（Offloading）：** 大量请求在边缘节点被缓存命中，不需要打到源服务器。

**提升可用性（Availability）：** 即使源服务器临时故障，边缘节点仍可继续提供缓存内容。

**DDoS 防护：** 大量恶意流量被边缘节点吸收和过滤，保护源服务器。

**全球加速：** 跨洲际传输时，通过 CDN 骨干网络优化路由，避免公网长距离传输的延迟。

---

## 适合缓存的内容

**静态资源（Static Assets）：** 图片、CSS、JavaScript、字体文件、视频——CDN 最核心的使用场景。

**动态内容（Dynamic Content）：** 部分 CDN（如 Cloudflare）支持对动态 API 响应做缓存，需要配置缓存规则（Cache Rules）和缓存键（Cache Key）。

**流媒体（Streaming）：** HLS、DASH 格式的视频切片文件非常适合 CDN 分发。

---

## 缓存控制

通过 HTTP 响应头控制 CDN 和浏览器的缓存行为：

- `Cache-Control: public, max-age=86400`：允许 CDN 缓存，有效期 1 天
- `Cache-Control: no-cache`：每次需要验证（Revalidation）
- `Cache-Control: no-store`：不缓存（如敏感数据）
- `ETag`：内容指纹，用于验证缓存是否过期

**缓存失效（Cache Invalidation）：** 内容更新后主动清除 CDN 缓存（Purge），或通过文件名加版本号（如 `main.v2.css`）避免缓存问题。

---

## CDN 与源服务器的交互

```
用户 → 边缘节点（Cache Hit）→ 返回
用户 → 边缘节点（Cache Miss）→ 回源服务器 → 缓存 → 返回
```

**回源策略：**
- **拉取（Pull）：** 懒加载，首次请求时回源，适合大多数场景
- **推送（Push）：** 主动将内容预热（Pre-warm）到边缘节点，适合已知流量高峰（如大促前）

---

## 小结

> CDN 是提升用户体验和系统稳定性的标配组件。静态资源一定要走 CDN，动态内容可按需选择。部署 CDN 后，**缓存失效策略**是最需要仔细设计的部分。
