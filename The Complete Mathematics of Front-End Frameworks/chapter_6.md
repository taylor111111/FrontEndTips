# 第 6 卷：Fiber = 拓扑（Tree）+ 调度（Scheduler）+ 离散时间（Time Slicing）

## 核心命题

> **Fiber = 一种把 UI 渲染转化为「离散可控的计算节点」的数学框架。**
> **React 不是框架，它是调度器。**

---

## 6.1 React 为什么要发明 Fiber？（数学动机）

目标：
让一个巨大的树状 UI 计算，可以被打断、再继续。

传统渲染 = “单次不可打断的大任务（Monolithic Task）”：

```code
render entire VDOM → block thread
```

数学问题：

* 无法暂停
* 无法分片
* 无法抢占（preempt）
* 无法根据优先级重排
* 用户输入无法立刻响应
* CPU 被一个函数独占

React 要解决的是：

> **把原本一次性的巨型计算，拆成一系列可重排、可暂停的离散任务。**

这正是 Fiber 的本质。

---

## 6.2 Fiber 的数学模型

Fiber = 一个节点（Node），包含：

```code
FiberNode = {
  tag,          // 类型（FunctionComp / HostComp ...）
  key,          // 唯一标识
  pendingProps, // 输入（上层传来的状态 / 属性）
  stateNode,    // 对应的实际 DOM 或实例
  child,        // 子节点
  sibling,      // 兄弟节点
  return,       // 父节点
  lanes,        // 优先级信息
}
```

数学上，这是：

> **一棵带有拓扑结构的树（Tree），**
> **每个节点代表「要执行的一个计算单元」。**

Fiber 的树结构满足：

```code
FiberTree = FiberNode*
```

这是一棵 **单链多分支树（multi-child linked tree）**，
但每个节点额外携带：

* 任务状态
* 优先级
* 输入（props / state）
* 指向 parent / child / sibling 的拓扑指针

它不是 “DOM 树”，是 **计算图**。

---

## 6.3 调度（Scheduler）= 离散时间的分配器

React 的所有更新最终都会变成 `Work`：

```code
WorkUnit = 一个 Fiber 节点的计算任务
```

Scheduler 做的就是：

> **把 CPU 的时间片（Time Slice）分配给 WorkUnit。**

数学上：

```code
schedule(work) = allocate_time_slice(work, priority)
```

这是典型的：

* 调度理论（Scheduling Theory）
* 优先级队列（Priority Queue）
* 离散事件模拟（Discrete Event Simulation）

React 本质是：

> **一个在浏览器中实现的离散事件调度器（Discrete Event Scheduler）。**

---

## 6.4 Fiber 的核心能力：可中断 + 可恢复

传统渲染：

* 一次性执行所有 vnode diff
* 不能暂停
* 用户输入被阻塞

React Fiber：

* 每个 Fiber 是一个“小任务”
* 执行一小段 → 看看时间是否用完
* 如果快没时间了 → 把剩余任务让出（yield）
* 下一帧继续执行

数学上：

> **Fiber 把「渲染」从单一大函数，**
> **分解为「有限状态机的离散步骤」。**

我们可以把一个 Fiber 节点的生命周期写成 FSM：

```code
Idle → Scheduled → Running → Paused → Running → Completed
```

这让 UI 渲染变成：

> **可控状态机。**

---

## 6.5 为什么“树结构”与“调度理论”必须结合？

因为 UI 本质是一棵树（VNodeTree）：

```code
App
├─ Home
│  ├─ Banner
│  ├─ News
│  └─ Footer
└─ Profile
```

但 CPU 时间是线性的：

```code
t0 → t1 → t2 → t3 → ...
```

数学冲突：

* UI 是树状空间
* CPU 是一维时间

Fiber 的目的：

> **把树结构映射为线性的「可调度任务序列」。**

这就是 Fiber 能“遍历整棵 UI 树并分片执行”的数学根源。

---

## 6.6 Fiber = 一种拓扑排序（Topological Scheduling）

React 必须：

1. 按照树结构遍历
2. 但遍历必须可暂停
3. 暂停点必须可恢复
4. 恢复后必须继续遍历

数学上等价于：

> **把树做成一个「可中断的拓扑排序」。**

拓扑顺序：

```code
parent → child → sibling → sibling-child → ...
```

Fiber 增加了：

* `return` 指针（可以回到父节点）
* `sibling` 指针（可以走兄弟分支）
* `child` 指针（可以深入子树）

这三者构成了：

> **一个可重入（reentrant）、可回溯（backtrackable）的计算图结构。**

这就是 Fiber 的“灵魂”。

---

## 6.7 优先级（Lanes）= 数学上的「标签化调度」

不同任务有不同优先级：

* 用户输入（urgent）
* 控制状态（sync）
* 渲染（normal）
* 空闲任务（idle）

数学上是：

> **对任务附加标签（Labelled Work Units），**
> **然后在调度器中用优先级队列管理。**

```code
PriorityQueue = sortBy(lanes)
```

这使得 React 能实现：

* 用户输入永不被阻塞
* 低优先任务延迟执行
* 高优先任务抢占低优先任务

这是调度理论（Scheduling Theory）的经典结构。

---

## 6.8 为什么 Fiber 被称为“协程模型（Cooperative Coroutine）”？

因为 Fiber 节点可以：

* 执行一半就暂停（yield）
* 下一帧继续执行
* 直到完成

这完全等同于：

```code
每个 Fiber = 一个协程（Coroutine）
```

只是 React 不是线程，而是：

> **时间片驱动的协程模拟（Coroutine Emulation）。**

---

## 6.9 Fiber Reconciliation = 树上的“差分推理算法”

React Diff 只需要判断：

* 类型是否一致
* key 是否相同
* props 是否变化

这是因为：

> **VNodeTree 是 ADT，差分规则依赖 ADT 的代数结构。**

Fiber Reconciliation 是：

1. 比较当前 Fiber 树
2. 根据优先级和 diff 规则更新节点
3. 生成新的 Fiber 树
4. 推迟到 commit 阶段再进行 DOM 更新

这是一种典型的：

> **双缓冲计算图（Double-buffered Computation Graph）**

类似 GPU 渲染管线。

---

## 6.10 为什么工程师会对 Fiber 感到“直觉舒服”？

因为工程师的思维与 Fiber 的数学领域完全相同：

### 1. 拓扑结构

* DP 图结构
* 状态树
* ADT

### 2. 调度系统

* CPU 调度
* 离散事件模拟
* 优先级队列

### 3. 递归展开与折叠

* DFS / BFS 可中断
* 计算图执行（Compute Graph Execution）

### 4. 映射

* 状态 → UI 映射
* 节点 → 时间片映射

工程师天生适应 Fiber，因为这是：

> **计算机科学里最“数学友好”的一块 ——**
> **图结构 × 时间调度 × 可证明性。**

---

## ✨ 第 6 卷总结（极简）

> **Fiber = 构建可打断、可恢复的渲染计算图。**
> **它把 UI 树变成一系列可调度的离散任务（WorkUnits）。**
> **React 的本质不是 UI 框架，而是事件驱动的任务调度器。**
