# 《模块依赖图 → 懒加载子图 → 用户体验状态机 → chunk 结构》

---

## 1. 模块依赖图（Module Dependency Graph）

先画层级依赖，只看大块目录：

```text
[ infra ]        底层：HTTP、storage、config、logger ...
    ↓
[ domain ]       业务领域：user / order / product ...
    ↓
[ features ]     场景模块：LoginForm / ProductCard / DashboardCharts ...
    ↓
[ ui ]           纯 UI 组件：Button / Table / Form ...
    ↓
[ app ]          App.tsx / routes / providers（路由、布局、全局状态）
```

再细一点，举个具体领域：

```text
infra/http/client.ts
        ↑
domain/user/service.ts
        ↑
features/LoginForm/LoginForm.tsx
        ↑
app/routes/user/index.tsx
```

这就是完整依赖链：

> **外部世界 → infra → domain → features → app(ui)**

---

## 2. 懒加载子图（Lazy Subgraph）

假设我们做两个懒加载点：

- 路由级懒加载：`/dashboard`
- 重组件懒加载：`DashboardCharts`（用到大型图表库）

先画完整模块图：

```text
App (routes)
├─ Route "/"
│   └─ HomePage
└─ Route "/dashboard"
    └─ DashboardPage
        ├─ features/DashboardHeader
        ├─ features/DashboardCharts   ← 重组件
        └─ domain/order / product ...
```

在这个图上，把“懒加载子图”框出来：

```text
                (lazy route chunk)
App (routes) ────────────────┐
                              │
                              │  Route "/dashboard"
                              │   └─ DashboardPage
                              │       ├─ DashboardHeader
                              │       ├─ DashboardCharts ──┐  (二级 lazy)
                              │       └─ domain/order/product
                              │
                              └───────────────────────────┘

                     (lazy heavy-ui chunk)
                     ┌──────────────────────┐
                     │ features/DashboardCharts
                     │   └─ ui/Chart + chart-lib
                     └──────────────────────┘
```

解释：

- 第一层：`/dashboard` 整个 route 是一个 **lazy subgraph**
  - `React.lazy(() => import('app/routes/dashboard'))`
- 第二层：`DashboardCharts` 内部再 lazy
  - `lazy(() => import('./DashboardCharts'))`
- 把图表库单独拆成更小的 lazy subgraph

你脑子里的抽象应当是：

> **大图（整个项目依赖图）上，圈出“某些子树”标记为 Lazy。**

---

## 3. 用户体验状态机（UX State Machine）

对任意一个懒加载点（例如 `/dashboard` 或 `DashboardCharts`），  
我们都对应一套相同的状态机：

```text
[ idle ]        （用户还没触发这个懒加载点）
    ↓ 用户导航到 /dashboard
[ loading ]     （import() 开始下载 chunk）
   ↙        ↘
[ success ]   [ error ]
               ↓ 用户点击“重试”
            （回到 loading）
```

和 React / Suspense 对应：

- `idle`：组件还没被渲染 / 用户没进入这个 route
- `loading`：React.lazy 触发，fallback 在页面上
- `success`：chunk 加载成功，组件正常渲染
- `error`：chunk 加载失败 / import reject → 显示错误 UI + 重试按钮

你可以给自己定一个“硬标准”：

> **每一个懒加载点，都必须画出这四个状态和箭头。**  
> **不能只是 import()，必须有完整 UX 状态机。**

---

## 4. chunk 结构（Bundle / Chunk Graph）

结合上面目录 + 懒加载点，我们画一个打包后的 chunk 拓扑图（大致形态）：

```text
┌──────────────── main chunk ────────────────┐
│ - app/App.tsx                               │
│ - app/routes/index.tsx                     │
│ - providers                                │
│ - 核心 ui/Button / Table                  │
└────────────────────────────────────────────┘
                    │
                    │ dynamic import()
                    ↓
┌──────────── route-dashboard chunk ─────────┐   ← 懒加载路由
│ - app/routes/dashboard                     │
│ - features/DashboardHeader                 │
│ - domain/order / product                   │
└────────────────────────────────────────────┘
                    │
                    │ dynamic import()
                    ↓
┌──────────── dashboard-charts chunk ────────┐   ← 重组件再次拆分
│ - features/DashboardCharts                 │
│ - ui/Chart                                 │
│ - chart-lib（e.g. echarts）                │
└────────────────────────────────────────────┘

（同时还有）

┌──────────── vendor chunk ──────────────────┐
│ - react / react-dom                         │
│ - axios / dayjs ...                         │
└────────────────────────────────────────────┘
```

核心信息：

- **main chunk**：首屏必须的东西（App、首页路由、基础 UI）
- **route-dashboard chunk**：用户进入 `/dashboard` 时才加载
- **dashboard-charts chunk**：用户看到图表区域时才加载
- **vendor chunk**：通用第三方库，可能被 main / route chunk 共享

你可以直接把这个当成「项目的 bundle 规划图」。

之后看 bundle analyzer 时，就能对照这个抽象去问：

> **“现在真实打出来的 chunk，有没有偏离我设计的结构？”**

---

## 总结

这样四张图串起来，你脑子里就很清楚了：

> **依赖图（Graph） → 子图延迟（Lazy Subgraph） →  
> 加载状态机（idle / loading / success / error） →  
> 工程策略（分 chunk、fallback、监控）。**
