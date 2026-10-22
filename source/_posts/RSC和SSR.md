---
title: SSR和RSC
date: 2026-10-22 14:00:17
categories: 前端
tags: 
 - SSR
 - React
---

# SSR和RSC

## 概述

本文档详细介绍 React 19 中的服务端组件（RSC）和服务端渲染（SSR）的概念、联系、区别以及实际应用。

---

## 一、SSR（传统服务端渲染）

### 定义

在服务端将整个页面渲染成 HTML，发送到客户端。

### 核心特征

| 特性          | 说明                           |
| ------------- | ------------------------------ |
| **渲染时机**  | 服务端渲染成完整 HTML          |
| **Hydration** | 客户端重新执行 JS 来"激活"交互 |
| **组件性质**  | 所有组件都是客户端组件         |
| **数据获取**  | 路由级别进行数据预取           |
| **代码传输**  | 所有 JS 都要发送到客户端       |

### 工作流程

```
服务端 SSR:
  React 组件 → renderToString() → HTML 字符串
                    ↓
              发送 HTML 到浏览器（快速显示）
                    ↓
客户端 Hydration:
  下载 JS bundle → 执行 React 代码 → 重建虚拟 DOM
                    ↓
              绑定事件监听器 → 页面可交互
```

### 示例代码

```tsx
// 传统 SSR (Next.js Pages Router)
export async function getServerSideProps() {
  const data = await fetch('/api/data')
  return { props: { data } }
}
```

---

## 二、RSC（React Server Components）

### 定义

组件在服务端执行，结果序列化为特殊格式流式传输到客户端。

### 核心特征

| 特性          | 说明                               |
| ------------- | ---------------------------------- |
| **渲染时机**  | 服务端执行，结果流式传输           |
| **Hydration** | 无需 hydration（服务端组件）       |
| **组件性质**  | 分为服务端组件（默认）和客户端组件 |
| **数据获取**  | 组件级别直接访问服务端资源         |
| **代码传输**  | 只发送客户端组件的 JS              |

### 工作流程

```
服务端组件 → 序列化流 → 客户端只接收结果
客户端组件 → 正常 hydration
```

### 示例代码

```tsx
// RSC (Next.js App Router)
async function Page() {
  const data = await db.query()  // 直接访问数据库
  return <div>{data}</div>
}
```

---

## 三、SSR 与 RSC 的联系与演进

### 演进关系

```
SSR → RSC 是 React 渲染模型的演进

传统 SSR：
  服务端渲染 HTML → 客户端 hydration（全部重新执行）

RSC 模型：
  服务端组件 → 序列化流 → 客户端只接收结果
  客户端组件 → 正常 hydration
```

### 类比理解

| 特性      | 传统 SSR           | RSC                   |
| --------- | ------------------ | --------------------- |
| 类比      | 整部电影一次性下载 | 流媒体按需加载        |
| 代码传输  | 所有 JS 都发送     | 只发送客户端组件的 JS |
| 数据获取  | 页面级别预取       | 组件级别直接访问      |
| Hydration | 全量 hydration     | 按需 hydration        |
| 交互      | 一次性激活         | 部分激活              |

### 关键联系

1. **都发生在服务端**：SSR 和 RSC 都利用服务端进行渲染
2. **RSC 可以在 SSR 中使用**：在 Next.js App Router 中，RSC 和 SSR 协同工作
3. **混合模型**：现代框架（如 Next.js 13+）将两者结合：
   - RSC 用于组件级别的服务端逻辑
   - SSR 用于页面级别的首屏渲染

### 总结对比

- **SSR** 关注的是**页面级别的渲染优化**（首屏性能、SEO）
- **RSC** 关注的是**组件级别的执行模型**（减少客户端 JS、更好的数据获取）
- **RSC ≠ SSR 的替代品**，而是对渲染模型的细化，两者可以协同工作
- React 19 中 RSC 更加成熟，但两者依然共存、互补

---

## 四、组件嵌套规则

### 核心规则

**客户端组件不能直接嵌套服务端组件作为子组件。**

```tsx
// ❌ 错误示例
'use client'

function ClientComponent() {
  return (
    <div>
      <ServerComponent />  {/* ❌ 不允许 */}
    </div>
  )
}
```

### 原因分析

服务端组件在服务端执行后序列化，结果流式传输到客户端。客户端组件在浏览器中执行，此时服务端组件已经"消失"了，只剩下渲染结果。

**客户端组件不知道服务端组件的存在，无法实例化它。**

### 解决方案：通过 props 传递

客户端组件可以通过 `children` 或其他 props 接收服务端组件的**渲染结果**：

```tsx
// ✅ 正确方式：通过 children

// 服务端组件
function ServerComponent() {
  return <p>我是服务端组件</p>
}

// 客户端组件
'use client'
function ClientComponent({ children }) {
  return (
    <div className="client-wrapper">
      {children}  {/* ✅ 接收已渲染的结果 */}
    </div>
  )
}

// 父组件（服务端组件）
function Parent() {
  return (
    <ClientComponent>
      <ServerComponent />  {/* ✅ 这里可以，因为是在服务端组合的 */}
    </ClientComponent>
  )
}
```

### 图示理解

```
服务端渲染阶段：
  Parent (Server) → 创建 ServerComponent
                  → 传给 ClientComponent 的 children
                  → 序列化为特殊格式

客户端接收：
  ClientComponent 接收的是已经渲染好的"children 节点"
  （不是 ServerComponent 组件本身，而是它的结果）
```

### 嵌套规则总结表

| 情况                                                   | 是否允许 |
| ------------------------------------------------------ | -------- |
| Server Component 嵌套 Client Component                 | ✅        |
| Server Component 嵌套 Server Component                 | ✅        |
| Client Component 嵌套 Server Component（直接导入）     | ❌        |
| Client Component 通过 props 接收 Server Component 结果 | ✅        |

### 实际应用示例

```tsx
// ✅ 常见模式：服务端获取数据，客户端负责交互

async function ServerParent() {
  const data = await db.query()  // 服务端获取数据
  
  return (
    <ClientList items={data}>
      <ServerHeader />  {/* 作为 children 传入 */}
    </ClientList>
  )
}

'use client'
function ClientList({ items, children }) {
  const [selected, setSelected] = useState(null)
  
  return (
    <div>
      {children}
      <ul>
        {items.map(item => (
          <li key={item.id} onClick={() => setSelected(item)}>
            {item.name}
          </li>
        ))}
      </ul>
    </div>
  )
}
```

---

## 五、Hydration 深度解析

### Hydration 的本质

Hydration 是 **让静态 HTML"活过来"的过程** —— 将服务端渲染的静态 DOM 与客户端 React 组件树"对齐"并绑定事件处理。

### Hydration 具体做了什么

#### 1. 重建组件树

```tsx
// 服务端已经渲染出 HTML
<div id="root">
  <button class="btn">点击我</button>
</div>

// 客户端 hydration 过程：
// - 重新执行 React 组件代码
// - 在内存中重建虚拟 DOM 树
// - 不重新创建真实 DOM（已存在）
```

#### 2. 对齐与校验

```tsx
function App() {
  return <button className="btn">点击我</button>
}

// Hydration 时 React 会：
// 1. 对比虚拟 DOM 和真实 DOM
// 2. 检查结构、属性、文本是否匹配
// 3. 如果不匹配 → 控制台警告（hydration mismatch）
```

#### 3. 绑定事件

```tsx
function App() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(c => c + 1)}>点击 {count}</button>
}

// Hydration 做的事：
// ✅ 为 <button> 添加 click 事件监听器
// ✅ 初始化 React 状态（count = 0）
// ✅ 建立状态更新 → DOM 更新的响应式连接
```

### 完整流程图

```
┌─────────────────────────────────────────────────────┐
│                    服务端 SSR                        │
│  React 组件 → renderToString() → HTML 字符串        │
└─────────────────────────────────────────────────────┘
                        ↓
              发送 HTML 到浏览器（快速显示）
                        ↓
┌─────────────────────────────────────────────────────┐
│                    客户端 Hydration                  │
│  1. 下载 JS bundle                                  │
│  2. 执行 React 代码，重建虚拟 DOM 树                 │
│  3. 遍历真实 DOM，与虚拟 DOM 对比                    │
│  4. 绑定事件监听器                                   │
│  5. 初始化状态、副作用                               │
└─────────────────────────────────────────────────────┘
                        ↓
                页面变得可交互
```

### 为什么需要 Hydration？

| 问题                    | 解决方案                    |
| ----------------------- | --------------------------- |
| HTML 是静态的，没有事件 | Hydration 绑定事件          |
| 没有状态管理            | Hydration 初始化 React 状态 |
| 用户无法交互            | Hydration 让页面"活"起来    |

### Hydration 的问题

#### 1. 重复工作

```tsx
// 服务端已经渲染过一次
// 客户端 hydration 又要重新执行一遍组件代码
// 相当于做了两次渲染
```

#### 2. 必须下载所有 JS

```tsx
// 传统 SSR：即使某个组件不需要交互
// 它的代码也要发送到客户端参与 hydration

function Page() {
  return (
    <>
      <StaticContent />      {/* 不需要交互 */}
      <InteractiveButton />  {/* 需要交互 */}
    </>
  )
}

// 两个组件的代码都要下载！
```

#### 3. Hydration Mismatch 错误

```tsx
function App() {
  // 服务端和客户端时间不同 → 错误！
  return <div>{new Date().toLocaleString()}</div>
}

// 控制台警告：
// Warning: Text content did not match.
// Server: "2024-01-01 10:00:00"
// Client: "2024-01-01 10:00:01"
```

**Hydration Mismatch 的常见原因**

```tsx
// 1. 时间相关
<div>{new Date().toLocaleString()}</div>

// 2. 随机数
<div>{Math.random()}</div>

// 3. 浏览器 API
<div>{window.innerWidth}</div>

// 4. 用户特定数据
<div>{localStorage.getItem('user')}</div>

// 5. 条件渲染不一致
function App() {
	const isServer = typeof window === 'undefined'
	return <div>{isServer ? 'Server' : 'Client'}</div>  // 不一致！
}
```

#### 4. 性能开销

```tsx
// Hydration 必须在客户端完成
// 大型应用 → 大量组件 → hydration 耗时
// 用户可能看到内容但无法交互（"hydration 延迟"）
```

### Hydration 做的事与问题总结表

| Hydration 做的事 | 问题                 |
| ---------------- | -------------------- |
| 重建组件树       | 重复工作             |
| 对齐 DOM         | 必须下载全部 JS      |
| 绑定事件         | 可能有 mismatch 错误 |
| 初始化状态       | 性能开销             |

---

## 六、RSC 如何改进 Hydration

### 传统 SSR vs RSC 对比

```tsx
// 传统 SSR + Hydration
// 所有组件都要 hydration

function Page() {
  return (
    <>
      <StaticHeader />        {/* 不需要交互 → 也要 hydration */}
      <InteractiveForm />     {/* 需要交互 → hydration */}
    </>
  )
}

// RSC 模型
// 只有客户端组件需要 hydration

function Page() {
  return (
    <>
      <StaticHeader />              {/* 服务端组件 → 不发送 JS */}
      <ClientInteractiveForm />     {/* 客户端组件 → 才 hydration */}
    </>
  )
}
```

### 流程对比

```
传统 SSR：
  [服务端渲染] → HTML
  [客户端] 下载全部 JS → 全量 Hydration

RSC：
  [服务端渲染] → HTML + 服务端组件结果流
  [客户端] 只下载客户端组件 JS → 部分 Hydration
```

### 代码层面的体现

```tsx
// RSC 下的 hydration 更轻量

// 服务端组件（不会 hydration）
async function ServerComponent() {
  const data = await db.query()  // 服务端执行
  return <div>{data}</div>       // 结果流式传输到客户端
}
// ↑ 这个组件的 JS 不会发送到浏览器

// 客户端组件（需要 hydration）
'use client'
function ClientComponent() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}
// ↑ 只有这个组件的代码会下载并 hydration
```

### 核心改进

**RSC 不是优化 hydration，而是从根本上减少需要 hydration 的代码量。**

---

## 七、总结

### 关键要点

1. **SSR** 和 **RSC** 是不同层面的概念：
   - SSR 关注页面级别的渲染优化
   - RSC 关注组件级别的执行模型

2. **Hydration** 是传统 SSR 必不可少的一步，但也带来了性能开销

3. **RSC 的革命性** 在于减少需要 hydration 的组件，而不是优化 hydration 过程

4. **组件嵌套规则** 是 RSC 的核心边界，需要正确理解和使用

5. **现代 React 应用** 可以将 SSR 和 RSC 结合使用，获得最佳性能

### 最佳实践建议

1. 默认使用服务端组件，只在需要交互时使用客户端组件
2. 遵循组件嵌套规则，通过 props 传递服务端组件结果
3. 合理拆分服务端和客户端逻辑，最大化 RSC 的优势
4. 关注 hydration 性能指标，优化用户体验

