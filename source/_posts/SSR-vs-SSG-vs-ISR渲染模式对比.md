---
title: SSR_vs_SSG_vs_ISR渲染模式对比
date: 2026-10-22 15:28:30
categories: 前端
tags: SSR
---

# SSR vs SSG vs ISR 渲染模式对比

## 概述

Next.js 提供了三种主要的渲染模式，每种都有其特定的使用场景和优缺点。本文档详细对比 SSR、SSG 和 ISR 的区别和适用场景。

---

## 一、三种渲染模式概览

### 核心定义

| 渲染模式 | 全称                            | 渲染时机                |
| -------- | ------------------------------- | ----------------------- |
| **SSG**  | Static Site Generation          | 构建时预渲染所有页面    |
| **SSR**  | Server-Side Rendering           | 每次请求时服务端渲染    |
| **ISR**  | Incremental Static Regeneration | 构建时预渲染 + 定期更新 |

### 快速对比表

| 特性           | SSG（静态生成） | SSR（服务端渲染） | ISR（增量静态再生成） |
| -------------- | --------------- | ----------------- | --------------------- |
| **渲染时机**   | 构建时          | 每次请求时        | 构建时 + 定期更新     |
| **性能**       | 最快            | 中等              | 快（缓存）+ 定期更新  |
| **数据新鲜度** | 过期            | 实时              | 定期更新（可配置）    |
| **服务器负载** | 无              | 高                | 低（缓存命中）        |
| **TTFB**       | 最快            | 慢                | 快（缓存命中）        |
| **适用场景**   | 固定内容        | 实时数据          | 定期更新的内容        |

---

## 二、SSG（Static Site Generation）

### 工作流程

```
构建阶段：
  所有页面预渲染 → 生成静态 HTML 文件

运行时：
  用户请求 → 直接返回静态文件
           ↓
      无需服务器渲染
           ↓
      响应速度最快
```

### 代码实现

```tsx
// app/blog/page.tsx

// 构建时获取数据并生成静态页面
export default async function BlogPage() {
  const res = await fetch('https://api.example.com/posts')
  const posts = await res.json()
  
  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  )
}
```

### 特点

| 优势              | 劣势                     |
| ----------------- | ------------------------ |
| ✅ 响应速度最快    | ❌ 数据可能过期           |
| ✅ 无服务器压力    | ❌ 构建时间长（页面多时） |
| ✅ 易于部署（CDN） | ❌ 无法动态更新内容       |
| ✅ SEO 友好        | ❌ 内容更新需重新构建     |

### 适用场景

- ✅ 博客文章
- ✅ 文档网站
- ✅ 营销落地页
- ✅ 公司官网
- ✅ 产品介绍页
- ❌ 实时数据页面
- ❌ 用户个性化内容

---

## 三、SSR（Server-Side Rendering）

### 工作流程

```
用户请求 → 服务端执行代码 → 实时获取数据 → 渲染 HTML → 返回给客户端
         ↓
    每次请求都重新渲染
         ↓
    数据总是最新的
         ↓
    服务器压力大
         ↓
    TTFB 较慢
```

### 代码实现

```tsx
// app/dashboard/page.tsx

// 强制每次请求都服务端渲染
export const dynamic = 'force-dynamic'

export default async function DashboardPage() {
  const res = await fetch('https://api.example.com/dashboard', {
    cache: 'no-store',  // 不缓存，每次请求都重新获取
  })
  const data = await res.json()
  
  return (
    <div>
      <h1>实时数据看板</h1>
      <p>活跃用户: {data.activeUsers}</p>
      <p>订单数量: {data.orders}</p>
      <p>更新时间: {new Date().toLocaleString()}</p>
    </div>
  )
}
```

### 特点

| 优势           | 劣势                        |
| -------------- | --------------------------- |
| ✅ 数据实时更新 | ❌ TTFB 慢（每次请求都渲染） |
| ✅ 内容个性化   | ❌ 服务器压力大              |
| ✅ 适合动态内容 | ❌ 高并发性能差              |
| ✅ SEO 友好     | ❌ 服务器成本高              |

### 适用场景

- ✅ 实时数据看板
- ✅ 股票行情
- ✅ 社交媒体动态
- ✅ 用户仪表盘
- ✅ 个性化推荐页面
- ❌ 固定内容页面
- ❌ 高流量内容页面

---

## 四、ISR（Incremental Static Regeneration）

### 工作流程

```
构建阶段：
  预渲染部分或全部页面 → 生成静态文件 + 缓存

运行时：
  用户请求 → 返回缓存内容
           ↓
      检查缓存是否过期
           ↓
      未过期：直接返回缓存（快）
      已过期：返回旧缓存 + 后台重新渲染
           ↓
      下次请求获得新内容
```

### Revalidate（重新验证）机制

```
时间轴示例（revalidate = 60 秒）：

T0 (构建时):
  生成页面，缓存到服务器

T1 (30s 后):
  用户请求 → 缓存未过期 → 返回 T0 的缓存（新鲜）

T2 (60s 后):
  用户请求 → 缓存过期 → 返回 T0 的缓存（旧但可用）
            同时后台触发重新渲染

T3 (61s 后):
  后台渲染完成 → 缓存更新为新内容

T4 (90s 后):
  用户请求 → 缓存未过期 → 返回 T3 的缓存（新鲜）
```

### Stale-While-Revalidate 策略

```
核心原则：
1. 永远不让用户等待
2. 立即返回旧缓存（即使过期）
3. 后台异步更新
4. 下次请求获得新内容
```

### 代码实现

#### 基础 ISR

```tsx
// app/products/page.tsx

// 每 60 秒重新验证一次
export const revalidate = 60

export default async function ProductsPage() {
  const res = await fetch('https://api.example.com/products')
  const products = await res.json()
  
  return (
    <div>
      <h1>商品列表</h1>
      {products.map(product => (
        <div key={product.id}>
          <h2>{product.name}</h2>
          <p>价格: ¥{product.price}</p>
        </div>
      ))}
    </div>
  )
}
```

#### 单个请求级别的 Revalidate

```tsx
// app/news/page.tsx

export default async function NewsPage() {
  // 新闻列表：5 分钟更新一次
  const newsRes = await fetch('https://api.example.com/news', {
    next: { revalidate: 300 },  // 5 分钟
  })
  const news = await newsRes.json()
  
  // 天气信息：10 分钟更新一次
  const weatherRes = await fetch('https://api.example.com/weather', {
    next: { revalidate: 600 },  // 10 分钟
  })
  const weather = await weatherRes.json()
  
  return (
    <div>
      <section>
        <h1>新闻</h1>
        {news.map(item => <p key={item.id}>{item.title}</p>)}
      </section>
      <section>
        <h1>天气</h1>
        <p>温度: {weather.temp}°C</p>
      </section>
    </div>
  )
}
```

#### ISR + 动态路由

```tsx
// app/products/[id]/page.tsx

// 每小时更新一次
export const revalidate = 3600

// 预生成热门商品页面
export async function generateStaticParams() {
  const res = await fetch('https://api.example.com/products')
  const products = await res.json()
  
  // 只预生成前 100 个热门商品
  return products.slice(0, 100).map(product => ({
    id: product.id.toString(),
  }))
}

export default async function ProductPage({ params }: { params: { id: string } }) {
  const res = await fetch(`https://api.example.com/products/${params.id}`)
  const product = await res.json()
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>价格: ¥{product.price}</p>
      <p>库存: {product.stock}</p>
    </div>
  )
}
```

**效果**：

- 前 100 个热门商品：构建时预生成
- 其他商品：首次访问时按需生成 + 缓存
- 所有商品页面：每小时自动更新一次

#### 按需重新验证（On-Demand Revalidation）

```tsx
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache'
import { NextRequest } from 'next/server'

export async function POST(request: NextRequest) {
  const path = request.nextUrl.searchParams.get('path')
  const tag = request.nextUrl.searchParams.get('tag')
  
  if (path) {
    // 重新验证指定路径
    revalidatePath(path)
    return Response.json({ revalidated: true, path, now: Date.now() })
  }
  
  if (tag) {
    // 重新验证指定标签
    revalidateTag(tag)
    return Response.json({ revalidated: true, tag, now: Date.now() })
  }
  
  return Response.json({ revalidated: false })
}
```

**使用场景**：

- CMS 内容更新后立即刷新页面
- 商品价格变动后立即更新商品页
- 无需等待 revalidate 时间

### 特点

| 优势                     | 劣势                               |
| ------------------------ | ---------------------------------- |
| ✅ 响应速度快（缓存命中） | ❌ 数据有延迟（最多 revalidate 秒） |
| ✅ 定期自动更新           | ❌ 需要服务器运行时                 |
| ✅ 服务器压力小           | ❌ 不支持静态导出                   |
| ✅ 按需手动更新           | ❌ 首次访问非预生成页面可能慢       |
| ✅ SEO 友好               | ❌ 分布式环境需共享缓存             |

### 适用场景

- ✅ 博客文章
- ✅ 新闻网站
- ✅ 电商商品页
- ✅ 文档网站（定期更新）
- ✅ 分类信息网站
- ❌ 实时数据（股票、社交）
- ❌ 高度个性化内容

---

## 五、详细对比分析

### 1. 性能对比

```
SSG:
  构建时: 渲染所有页面
  运行时: 直接返回静态文件（~5ms）
  TTFB: 最快

ISR:
  构建时: 预生成部分页面
  运行时: 返回缓存（~5ms）或按需生成（~200ms）
  TTFB: 快（缓存命中） / 中（缓存未命中）

SSR:
  构建时: 无渲染
  运行时: 每次请求都渲染（~200ms）
  TTFB: 慢
```

### 2. 数据新鲜度对比

```
SSG:
  数据新鲜度: 构建时的数据
  更新方式: 重新构建部署
  数据延迟: 数小时到数天

ISR:
  数据新鲜度: revalidate 秒前的数据
  更新方式: 自动更新 + 手动触发
  数据延迟: 最多 revalidate 秒

SSR:
  数据新鲜度: 实时
  更新方式: 每次请求都获取最新数据
  数据延迟: 0
```

### 3. 服务器负载对比

```
假设：每秒 1000 个请求，页面渲染耗时 200ms

SSG:
  服务器压力: 0（所有请求都返回静态文件）
  计算次数: 0 次/秒

ISR (revalidate = 60s):
  服务器压力: 极低
  计算次数: 最多 1 次/60 秒（后台渲染）
  999+ 请求都返回缓存

SSR:
  服务器压力: 高
  计算次数: 1000 次/秒
  需要强大的服务器支撑
```

### 4. 适用场景决策表

| 场景       | SSG  | ISR  | SSR  | 原因                 |
| ---------- | ---- | ---- | ---- | -------------------- |
| 博客文章   | ✅    | ✅    | ❌    | 内容固定，SEO 重要   |
| 新闻网站   | ❌    | ✅    | ❌    | 内容定期更新，流量大 |
| 电商商品页 | ❌    | ✅    | ❌    | 商品信息定期更新     |
| 文档网站   | ✅    | ✅    | ❌    | 内容相对固定         |
| 股票行情   | ❌    | ❌    | ✅    | 数据必须实时         |
| 社交媒体   | ❌    | ❌    | ✅    | 动态实时更新         |
| 用户仪表盘 | ❌    | ❌    | ✅    | 个性化实时数据       |
| 营销落地页 | ✅    | ✅    | ❌    | 固定内容，追求性能   |
| 公司官网   | ✅    | ✅    | ❌    | 内容固定             |
| 实时监控   | ❌    | ❌    | ✅    | 数据实时变化         |

---

## 六、决策流程图

```
开始
  ↓
是否需要 SEO？
├── 否 → CSR（客户端渲染）
│        例如：管理后台、登录后应用
│
└── 是
    ↓
    数据实时性要求？
    ├── 实时（秒级） → SSR
    │   例如：股票行情、社交媒体、实时监控
    │
    └── 可延迟（分钟/小时级）
        ↓
        流量大小？
        ├── 低流量 → SSR
        │   （简单，无需考虑缓存）
        │
        └── 高流量 → ISR
            ↓
            内容更新频率？
            ├── 几乎不变 → SSG
            │   例如：文档、营销页
            │
            └── 定期更新 → ISR
                例如：博客、新闻、电商
```

---

## 七、混合使用策略

在实际项目中，不同页面可以使用不同的渲染策略：

```tsx
// 项目结构示例

// app/
// ├── layout.tsx           (SSR - 布局)
// ├── page.tsx             (ISR - 首页，60s revalidate)
// ├── blog/
// │   ├── page.tsx         (ISR - 博客列表，300s revalidate)
// │   └── [slug]/
// │       └── page.tsx     (SSG - 博客文章，构建时生成)
// ├── products/
// │   ├── page.tsx         (ISR - 商品列表，60s revalidate)
// │   └── [id]/
// │       └── page.tsx     (ISR - 商品详情，3600s revalidate)
// ├── dashboard/
// │   └── page.tsx         (SSR - 用户仪表盘，实时数据)
// └── admin/
//     └── page.tsx         (CSR - 管理后台，无需 SEO)
```

### 示例配置

```tsx
// 首页 - ISR（60 秒更新）
export const revalidate = 60

// 博客列表 - ISR（5 分钟更新）
export const revalidate = 300

// 博客文章 - SSG（构建时生成）
export async function generateStaticParams() {
  const posts = await getAllPosts()
  return posts.map(post => ({ slug: post.slug }))
}

// 商品详情 - ISR（1 小时更新）
export const revalidate = 3600

// 用户仪表盘 - SSR（实时数据）
export const dynamic = 'force-dynamic'
```

---

## 八、最佳实践建议

### 选择建议

```
1. 优先考虑 ISR
   - 兼顾性能和数据新鲜度
   - 适合大多数内容网站
   - 减轻服务器压力

2. 实时数据用 SSR
   - 数据必须实时更新
   - 个性化内容
   - 不怕服务器压力

3. 固定内容用 SSG
   - 内容几乎不变
   - 追求极致性能
   - 简单部署

4. 无 SEO 需求用 CSR
   - 管理后台
   - 登录后应用
   - 高度交互应用
```

### Revalidate 时间建议

```
根据内容更新频率选择：

- 新闻快讯: 60-300 秒（1-5 分钟）
- 博客文章: 3600-7200 秒（1-2 小时）
- 商品信息: 3600 秒（1 小时）
- 文档网站: 86400 秒（24 小时）
- 营销页面: 3600-86400 秒（1-24 小时）
```

---

## 九、总结

### 核心区别

```
SSG = 静态生成，最快，数据可能过期
SSR = 实时渲染，最新，性能差
ISR = 静态 + 定期更新，平衡性能和数据新鲜度
```

### 核心价值

| 渲染模式 | 核心价值                |
| -------- | ----------------------- |
| **SSG**  | 极致性能 + 低成本       |
| **SSR**  | 实时数据 + 个性化       |
| **ISR**  | 性能 + 数据新鲜度的平衡 |

### 最终建议

```
如果不确定用什么 → 先用 ISR

ISR 的灵活性：
- revalidate 小（如 60s）→ 接近 SSR 的实时性
- revalidate 大（如 86400s）→ 接近 SSG 的性能
- 根据实际需求调整即可
```

