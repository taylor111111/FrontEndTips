# 第 5 卷：副作用 = 世界输入的数学化（Effect Boundary）

## 核心命题

> **Render（UI）与 Reducer（状态机）必须纯，**
> **所以“副作用”必须被隔离在系统边界。**

副作用不是框架的例外，
而是**框架数学结构的一部分**。

---

## 5.1 副作用是什么？数学定义

我们先从数学上给副作用（Effect）一个定义：

```code
Effect : World → Action*
```

也就是说：

> **副作用 = 外部世界（World）的输入 → 系统事件（Action）的输出**

世界的变化包括：

* 用户操作（click / input / scroll / hover）
* 时间（setTimeout / requestAnimationFrame / interval）
* 网络（fetch / WebSocket）
* 设备与环境（resize / focus / visibilitychange）
* 浏览器 API（localStorage / history / URL）

统一记为：

```code
World
```

这些变化：

* 不能直接修改 UI
* 不能直接修改 state
* **必须被转换为 Action 才能进入系统**

---

## 5.2 为什么副作用不能直接进入 UI 或 Reducer？

数学上，React 系统依赖两个核心纯函数：

```code
render  : State → VNodeTree
reducer : State × Action → State
```

它们必须同时满足：

* 无副作用（pure）
* 确定性（deterministic）
* 可重放（replayable）
* 可缓存（memoizable）
* 与调度器无关（scheduler‑independent）

### ❌ 如果副作用进入 render（UI）

```code
render(state) // 有副作用
```

可能发生：

* 读取 / 修改 DOM
* 改变 state
* 发起请求
* 设置定时器

后果是：

* render 无法被重放
* Fiber 无法中断 / 恢复 / 并发
* diff 不再确定

因此：

> **render 必须是数学映射，而不是过程。**

---

### ❌ 如果副作用进入 reducer（状态机）

```code
reducer(state, action) // 有副作用
```

直接破坏：

* time‑travel（回放失败）
* 日志记录
* 行为预测
* Action 覆盖验证

数学上等价于：

> **δ 不再是函数，FSM 被破坏。**

---

## 5.3 副作用必须在“边界”发生：Effect Boundary

为了保持纯性，系统必须引入一个明确的边界层：

```text
World → Effect Boundary → Action → Reducer → State → Render → UI
```

系统结构实际是：

```text
世界输入（World）
  ↓
副作用边界（Effect Boundary）
  ↓
Action（系统事件）
  ↓
Reducer（状态更新）
  ↓
State（状态空间）
  ↓
Render（UI 投影）
```

> **Effect Boundary 才是“世界的入口”。**

---

## 5.4 各框架的副作用模型，本质都是 Effect Boundary

不同框架 = 同一个数学模型的不同工程实现。

---

### 5.4.1 React 的副作用模型：useEffect

```ts
useEffect(effect, deps)
```

数学含义：

> **World → Action 的管道**

effect 的职责：

* 监听世界变化（订阅）
* 向系统注入 Action（dispatch）
* 在退出时清理（unsubscribe）

数学上，与 Redux middleware 同构。

---

### 5.4.2 Redux 的副作用模型：Middleware

```code
Effect : Action → (State → Action*)
```

Redux middleware：

* 观察 action
* 触发异步逻辑
* 再 dispatch 新 action

例如 redux‑thunk：

```code
dispatch(fn)
fn(dispatch, getState)
```

本质仍然是 Effect Boundary。

---

### 5.4.3 Vue 的副作用模型：watch + reactive graph

```code
watch(effect) = subscribe(World, Effect)
```

* reactive graph：负责变化追踪
* watch：负责进入副作用世界
* computed：**禁止副作用（必须纯）**

Vue 与 React 在数学结构上完全一致，只是“入口位置”不同。

---

## 5.5 副作用为何不能消除？因为外部世界本来就是非纯的

数学上，我们区分两个世界：

### 1. 纯世界（Pure World）

* 映射：State → UI
* 映射：State × Action → State
* 可证明、可重放、可推理

### 2. 脏世界（Impure World）

* 用户点击
* 时间
* IO
* 环境
* 浏览器 API

框架的目标不是消灭副作用，而是：

> **把脏世界隔离在边界，让纯世界保持数学性。**

这就是前端架构的本质任务。

---

## 5.6 副作用的分类（数学分层）

所有副作用可以分为三类：

---

### 1. 输入副作用（Input Effects）

来自用户或环境的事件：

* click
* input
* resize
* scroll
* fetch response

输入副作用对应：

```code
World_input → Action
```

---

### 2. 输出副作用（Output Effects）

写入世界的过程：

* DOM 操作（React 不允许你直接写）
* localStorage
* fetch
* console.log

输出副作用对应：

```code
State → World_output
```

React 允许你写输出副作用（在 effect 中）， 但禁止你写输入副作用（那是 React 的工作）。

---

### 3. 过程副作用（Process Effects）

包括时间、异步、延迟：

* setTimeout
* requestAnimationFrame
* async / await
* promise 链

它们的数学形式：

```code
Time → Action
```

---

## 5.7 Effect Boundary = 外界世界的“压缩机”

所有外部复杂性：

* 不规则
* 不可控
* 异步
* 随机
* 用户行为

都被压缩成：

```code
一个有限可枚举的 Action 集合
```

这是副作用边界的终极数学功能：

> **把无限世界压缩成有限自动机可处理的输入字母集（Alphabet）。**

这个观点你一定特别喜欢。

## 5.8 副作用 & DP 的相关联想 ？

你做 DP 做的是：

```code
无限搜索空间 → 状态空间压缩 → 转移方程
```

Effect Boundary 做的是：

```code
无限外部世界 → Action 集合压缩 → Reducer 输入
```

是同构问题。


---

## 🌟 第 5 卷总结（极简版）

> 副作用 = 世界输入。
> 副作用不能进入 render 或 reducer，因为它们必须是数学映射。
> 整个框架的任务，就是把外界复杂性隔离到“副作用边界”，
> 再把脏世界压缩成有限可枚举的 Action。
