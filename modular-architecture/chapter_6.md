# 🧩 一个结构回答三个问题

---

## 0. 先定一张「分层依赖图」

```text
infra → domain → features → ui → app/routes
（底层） （业务） （场景） （纯UI） （入口）
```

**规则：只能从左往右依赖。**

---

## 1. 模块如何划分？

- 按层 + 按领域划分：

  - `infra/`：HTTP、storage、config…
  - `domain/user | order | product`：模型 + 业务逻辑
  - `features/**`：登录表单、商品卡片、Dashboard 等“小场景块”
  - `ui/**`：Button / Table / Form
  - `app/routes/**`：页面入口

> 模块 = 这张图上的一个子块 / 子树。

---

## 2. 如何避免依赖循环？

- 只允许“向右依赖”：
  - domain 不能 import ui
  - features 不直接 import infra/http，而是走 domain.service
- 初始化逻辑放在最右边（`app/`）
  - 左边的层只定义结构，不做副作用

> **消除环 = 不允许往左指的箭头。**

---

## 3. 懒加载策略怎么定？

在同一张图上做两件事：

### 1. 选子图：

- 路由子图：`app/routes/dashboard` 这一坨 → lazy route
- 重组件子图：`features/DashboardCharts` → 二次 lazy

### 2. 给每个懒加载点绑定同一套状态机：

```text
idle → loading → success / error → retry
```

UI 上用 `fallback / skeleton / error + retry` 表达出来。

> **懒加载 = 在这张依赖图上圈出「延迟展开的子图」+ 给它们配状态机。**

---

## 所以你说得非常对：

这三个问题单独回答会显得零碎，  
**但用一张结构图回答，就会非常干净。**

以后如果有人问你：

- 模块怎么拆？
- 循环依赖怎么避免？
- 懒加载怎么设计？

你就把这张：

> **「分层依赖图 + 单向箭头 + Lazy 子图 + 状态机」**

拿出来——  
一句话概括就是：

> **先画图，再讨论实现。图画对了，细节怎么实现都不难。**
