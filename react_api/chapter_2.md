# useRef：逃生口，而不是状态系统

在 React 的 Hooks 体系中，`useRef` 是一个极其容易被误解、也极其容易被滥用的 API。

很多问题并不是“不会用 useRef”，而是**在不该用的地方用了它**。

本文从 **设计目的 → 推荐使用场景 → 不推荐使用场景 → 明确禁止的使用边界** 四个层面，重新把 `useRef` 放回它在工程中的真实位置。

---

## 一、useRef 的设计目的

### 1. 一句话定义

> **useRef 用来在 React 渲染模型之外，保存一个“稳定、可变、且不触发渲染”的引用容器。**

关键词拆解：

- **稳定（identity stable）**：ref 本身在组件整个生命周期内保持同一个引用
- **可变（mutable）**：可以自由修改 `ref.current`
- **不触发渲染**：修改 `ref.current` 不会引起 re-render

这是 React 明确给出的一个信号：

> **React 不会、也不打算追踪 ref 内部的变化。**

---

### 2. 源码层面的真实语义（简化）

从源码角度看，`useRef` 的本质几乎可以等价为：

```ts
const ref = { current: initialValue }
```

- mount 阶段创建一次
- update 阶段原样返回
- React 不关心 `.current` 的变化

从 React 的视角看：

> **useRef = “我不关心这个值的变化”**

这正是它与 `useState` 的根本区别。

---

### 3. 官方真正的设计动机

React 并不是为了“更方便写逻辑”而设计 `useRef`，
而是为了在一个**严格的声明式渲染模型**中，给开发者留出一个：

> **用于对接命令式世界的逃生口。**

它的存在，是为了处理那些：

- 不适合进入 state
- 不应触发渲染
- 却需要跨 render 保存

的对象或信息。

---

## 二、推荐使用 useRef 的场景（干净用法）

下面这些场景，使用 `useRef` 在工程上是**完全正当且推荐的**。

---

### 1. 对接命令式 / 非 React 系统（最正宗用途）

#### 典型场景

- DOM 节点
- 第三方实例（Map / Chart / Editor 等）
- WebSocket / RTC / Media 实例
- Observer / Subscription handle

```ts
const socketRef = useRef<WebSocket | null>(null)

useEffect(() => {
  socketRef.current = new WebSocket(url)
  return () => socketRef.current?.close()
}, [])
```

#### 为什么必须用？

- 实例不参与渲染
- 不应触发 re-render
- identity 必须稳定

这是 `useRef` 的**本职工作**。

---

### 2. 保存“跨 render 的临时状态”（但不影响 UI）

#### 典型例子

- 上一次的值
- timer / interval id
- debounce / throttle 内部状态
- 是否已经初始化过

```ts
const prevValueRef = useRef<number>()

useEffect(() => {
  prevValueRef.current = value
})
```

这些信息的共同特征是：

- 不需要展示在 UI 上
- 不参与 diff 或渲染决策
- 但需要在多次 render 之间保存

---

### 3. 解决“闭包过期”问题（谨慎、但合理的用法）

```ts
const latestStateRef = useRef(state)

useEffect(() => {
  latestStateRef.current = state
}, [state])
```

然后在：

- `setInterval`
- 事件监听器
- subscription callback

中读取最新值。

> **前提**：你必须非常清楚，这是在“逃离 React 的响应模型”。

---

## 三、不推荐使用 useRef 的场景（危险信号）

下面这些情况并非“立刻错误”，但通常意味着：

> **组件设计正在偏离 React 的建模方式。**

---

### 1. 用 useRef 代替 useState

```ts
const countRef = useRef(0)
countRef.current++
```

风险包括：

- UI 不更新
- 状态变化不可追踪
- debug 成本极高
- 与 React 心智模型脱节

> **如果变化需要反映在 UI 上，就应该使用 state。**

---

### 2. 用 useRef 绕开 effect 依赖数组

```ts
const handler = () => {
  doSomething(ref.current)
}
```

这种写法的本质是：

- 使用可变共享状态
- 绕开依赖声明
- 放弃数据一致性保证

短期省事，长期极难维护。

---

### 3. 让业务逻辑大量依赖 ref.current

如果你发现：

- 多个 effect 写同一个 ref
- if / else 判断大量读取 ref
- ref 成为核心数据源

那么你很可能在：

> **用 useRef 搭建一个“影子状态系统”。**

这是一个非常危险的信号。

---

## 四、一定别用 useRef 的场景（工程红线）

下面这些情况，可以明确视为**工程错误**。

---

### 1. 用 useRef 存储业务状态

```ts
const userRef = useRef(user)
```

并在逻辑中当作 state 使用。

问题在于：

- React 无法感知变化
- 状态不可预测
- 渲染与数据彻底脱节

> **这是在 React 中编写非 React 程序。**

---

### 2. 用 useRef 作为跨组件通信手段

```ts
// 父组件
const ref = useRef()

// 子组件直接修改 ref.current
```

这等同于：

- 全局可变状态
- 无边界数据流
- 无调试入口

如果需要跨组件通信，应使用：

- state 提升
- context
- store

而不是 ref。

---

### 3. 用 useRef 逃避 effect 依赖建模

```ts
const fnRef = useRef(fn)
fnRef.current = fn

useEffect(() => {
  doSomething(fnRef.current)
}, [])
```

这类写法的真实含义是：

> **“我拒绝参与 React 的一致性模型。”**

这不是优化，而是设计层面的逃避。

---

## 五、一个通用的判断法则

在考虑是否使用 `useRef` 时，可以问自己一句：

> **“如果这个值发生变化，React 应不应该知道？”**

- 如果 **应该** → ❌ 不要使用 `useRef`
- 如果 **不应该** → ✅ `useRef` 很合适

这个判断，比任何“最佳实践列表”都更可靠。

---

## 六、最终总结

> **useRef 是一个声明式系统中的命令式逃生口。**

因此，它的使用原则是：

- 可以用
- 但要克制
- 用在系统边界
- 不要用来建模核心业务逻辑

当一个工具被用来**绕开系统约束**，
而不是**解决边界问题**时，
它就已经越界了。
