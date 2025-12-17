# 《A Complete Guide to useEffect》最佳实践总纲

> **核心总原则**：
>
> **Effect 不是 lifecycle，它是“把某一次渲染的结果，同步到外部世界”的描述。**

---

## 1. 基础心智模型：Effect 不是生命周期，而是“同步描述”

### 核心思想

- 不要把 `useEffect` 想成 `componentDidMount / componentDidUpdate / componentWillUnmount` 的平替。
- **React 每一次渲染都是一张“时间快照（snapshot）”**：
  - 当前的 `props`、`state`、闭包里的变量，**统属于这一次快照**。
- Effect 的作用是：**在这张快照渲染完成后，做与外部世界的“同步动作”**，比如：
  - 订阅 / 取消订阅
  - 发送请求
  - 操作 DOM
  - 写入 `localStorage`

### 最佳实践

- 写 Effect 前先问自己：

  > “这是在**描述某次渲染之后**需要做的同步动作吗？”

- 如果不是在同步 UI 的结果，而是**在 Effect 里直接驱动业务流程 / 接管状态**，那大概率是设计有问题。

### 典型误用

- 把 Effect 当成：
  - 组件初始化逻辑
  - 数据流的总控制器
  - 生命周期回调集合

---

## 2. 每次渲染都有自己的“一切”：Each Render Has Its Own… Everything

### 核心思想

- **结构是恒定的，实例随时间不断变化**：
  - `state` 的结构是固定的，但每次 render 拿到的是“该时刻的取值版本”。
  - `function` 的定义位置是固定的，但每次 render 都会得到一个新的函数实例，闭包捕获的是“当次快照”的值。

```js
useEffect(() => {
  console.log(count); // 这里只能看到“这一次渲染”的 count
}, [count]);
```

### 最佳实践

- 在脑子里这么看待：
  - **不是一个组件在运行**，而是很多时间点上的组件快照在轮流登场。
- 写 Effect 时默认：

  > “这段 Effect 只能看见它所属那一次渲染时的世界。”

### 典型误用

```js
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1); // ❌ count 被固定在当次快照
  }, 1000);
  return () => clearInterval(id);
}, [count]);
```

### 正确写法

```js
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + 1); // ✅ 推导逻辑，不依赖闭包里的 count
  }, 1000);
  return () => clearInterval(id);
}, []);
```

---

## 3. Cleanup：清理也遵循“快照逻辑”

### 核心思想

- **每一次 Effect，都对应一次独立的 cleanup 函数**。
- React 保证：
  1. 先执行**上一次渲染的 cleanup**
  2. 再执行**这一次渲染的 Effect**

形成一条安全的时间线：

> old snapshot cleanup → new snapshot effect

### 最佳实践

- 把 cleanup 看成：

  > “撤销上一个快照对外部世界做的事情”

- 所有外部世界的绑定，都必须在 cleanup 里成对解除：
  - 事件监听
  - 订阅
  - 计时器
  - 手动挂在 DOM 上的东西

### 典型误用

```js
useEffect(() => {
  window.addEventListener('resize', handleResize); // ❌ 泄漏
}, []);
```

### 正确写法

```js
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize); // ✅
}, [handleResize]);
```

---

## 4. Synchronization, Not Lifecycle：同步而非生命周期

### 核心思想

- Effect = **同步描述**：
  - “当 UI 处于这种 state/props 快照时，我要让外部世界保持一致。”
- 而不是：
  - “先做 X，再做 Y，最后在 Z 时机收尾”的剧本。

### 最佳实践

- 把一个组件拆成**多个独立的同步关系**：
  - UI ⇄ localStorage
  - UI ⇄ WebSocket 订阅
  - UI ⇄ `document.title`
- **一个同步关系 = 一个 Effect**，不要把一串人生大事件塞进一个 Effect。

### 典型误用

- 写一个“大 Effect”，里面混杂：
  - 初始化
  - 请求
  - 订阅
  - 业务逻辑
  - 多个 if/else 状态机

结果：Effect 退化成“生命周期脚本”，完全失去同步语义的清晰度。

---

## 5. Teaching React to Diff Your Effects：依赖数组 = Effect 的 diff key

### 核心思想

- React **不会比较 Effect 函数的源码**，只看 `deps`：
  - 依赖数组是在告诉 React：

    > “这次 Effect 和上一次相比，哪些输入变了？”

- 当数组中任一依赖变化：
  1. 执行 cleanup（上一次快照）
  2. 再执行 effect（当前快照）

### 最佳实践

- 规则很简单：

  > **Effect 里用到的、会随时间变化的值，都必须出现在 deps 里。**

- 包括：
  - `props`
  - `state`
  - 在 Effect 内部使用的函数 / 计算结果（如果每次 render 都会重新创建）

- **想减少 deps 数量 → 重构代码，而不是撒谎删依赖。**

### 典型误用（“欺骗 React”）

```js
useEffect(() => {
  fetchData(id); // 用到了 id
}, []); // ❌ 明明依赖 id，却写成 []
```

结果：
- 数据和 UI 脱节
- 逻辑只在“第一次跑”，看起来像是意外的缓存

---

## 6. Don’t Lie About Dependencies：不要对依赖说谎

### 核心思想

- 少写 deps，本质是在**否认一段代码真实依赖的“时间维度”**。
- 你删掉的不是依赖，而是：
  - React 重跑同步逻辑的机会

这会导致：
- 旧闭包
- 过期 state
- 被无限期使用的旧逻辑

### 最佳实践：两种诚实方式（Dan 的两条正路）

#### 1. 让 Effect 完整声明依赖，然后重构逻辑

- 先让 ESLint 帮你列出所有依赖
- 再判断：

  > “是不是这段逻辑本来就不该写在 Effect 里？”

#### 2. 把“事件 → 状态更新”移到别处，Effect 只关心“结果”

- 使用：
  - `useCallback`
  - `useReducer`
- 先把事件触发 → 状态更新的路径处理好
- Effect 只订阅：

  > “某个状态结果变成了 X，需要做一次副作用同步。”

### 典型误用

```js
// eslint-disable-next-line react-hooks/exhaustive-deps
useEffect(() => { ... }, []);
```

- 这通常是在**掩盖设计问题**，而不是解决性能问题。

---

## 7. Making Effects Self-Sufficient：让 Effect 自给自足

### 核心思想

- 一个 Effect，应该**包含完成这段同步关系所需的全部信息**。
- 不要依赖：
  - 某个“外部可变单例”
  - 某次渲染之外的隐蔽状态

### 最佳实践

- 如果 Effect 逻辑越来越重、依赖越来越多：
  - 把纯计算提取成普通函数
  - 把“事件触发逻辑”移到 handler 或 `useReducer`
  - **让 Effect 只剩下：订阅 / 解绑 + 一次同步动作**

---

## 8. Decoupling Updates from Actions：把“事件动作”与“状态更新”解耦

### 核心思想

- 很多 bug 源于这种写法：

```js
useEffect(() => {
  // effect 里塞着各种事件相关 flags
  // 然后决定 setX / setY
}, [flagA, flagB, flagC]);
```

- Dan 的思路是：
  - **事件发生时 → 决定如何更新 state**
  - **Effect 只关心 state 更新之后，需要对外部世界做的事**

### 最佳实践

- 用以下方式管理“事件 → 状态更新”：
  - `useReducer`
  - 一组清晰的 handler 函数

- Effect 只订阅：

  > “某个状态结果发生了变化，需要同步一次副作用。”

### 什么时候使用 `useReducer`

- 页面状态：
  - 变量多
  - 相互约束、联动强
- 当你已经在写：

  > “这个变了，就必须把那个一起改”

  时，直接上 `useReducer`，会比多个 `useState` 清晰得多。

> 总结得很好：
> - 有相互约束的 → 用 `useReducer`
> - 相互独立的小状态 → 用多个 `useState`
> - 同一页面混用 `useReducer + useState` 是完全正常的

---

## 9. Removing Unnecessary Dependencies：通过重构来“减负”

### 核心思想

- 正确的步骤不是：**删依赖**
- 而是：
  1. 让依赖“诚实写全”
  2. 发现 Effect 太胖 → 重构
  3. 重构后自然减少依赖数量

### 最佳实践：常见重构方式

#### 把纯计算移到 Effect 外

```js
const result = expensiveCompute(a, b); // ✅ 纯计算

useEffect(() => {
  syncResult(result);
}, [result]);
```

#### 把“事件行为”移动到 handler

```js
const fetchData = useCallback(() => {
  // 用到 a, b, c
}, [a, b, c]);

useEffect(() => {
  fetchData();
}, [fetchData]); // ✅ Effect 依赖一个稳定定义的动作
```

---

## 结语

> **useEffect 的本质不是“什么时候跑”，而是：**
>
> **“当 UI 处于这个状态快照时，我要如何让外部世界保持一致。”**

一旦你用“快照 + 同步”的心智模型来看待 Effect，
- 依赖数组
- cleanup
- 闭包问题

都会自然地对齐、变得清晰。
