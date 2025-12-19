# useMemo / useCallback：从源码到工程边界的再认识

在 React 生态中，`useMemo` 和 `useCallback` 经常被当作“性能优化必备 API”。
但如果回到 **源码与设计目标**，再结合真实工程实践，会发现它们其实是**边界型工具**，而不是日常 API。

本文分两部分：

1. `useMemo / useCallback` 的源码层行为与官方设计目的
2. 哪些 **第三方类库类型** 会真正“迫使”我们使用它们，以及为什么

---

## 一、useMemo / useCallback 的源码分析

### 1. useMemo 的源码行为

> 以下是高度贴近 React 源码语义的简化版本（忽略调度、lane、DEV 校验）。

#### API 形式

```ts
useMemo(create, deps)
```

#### Fiber 上存储的数据结构

```ts
hook.memoizedState = {
  value,
  deps
}
```

---

### Mount 阶段

```js
function mountMemo(create, deps) {
  const value = create()
  hook.memoizedState = { value, deps }
  return value
}
```

- 执行 `create()`
- 缓存返回值
- 缓存依赖数组

---

### Update 阶段

```js
function updateMemo(create, deps) {
  const prev = hook.memoizedState

  if (areHookInputsEqual(deps, prev.deps)) {
    return prev.value
  }

  const value = create()
  hook.memoizedState = { value, deps }
  return value
}
```

#### 关键点只有一个

```js
areHookInputsEqual = deps.every(Object.is)
```

也就是说：

- React **只做 `Object.is` 级别的比较**
- 不理解语义
- 不判断“是否真的相等”
- 不分析表达式内容

---

### useMemo 的本质

> **在依赖 identity 不变时，复用上一次计算结果**

它不是语义缓存，而是**引用缓存**。

---

## 2. useCallback 的源码本质

### API 形式

```ts
useCallback(fn, deps)
```

### 实现等价关系（源码层面）

```ts
useCallback(fn, deps)
≡
useMemo(() => fn, deps)
```

也就是说：

- 缓存的是 **函数引用**
- 判断条件仍然是 `deps` 的 `Object.is`

---

### useCallback 不会做的事情

- ❌ 不会让函数“更纯”
- ❌ 不会解决闭包问题
- ❌ 不会自动避免 bug
- ❌ 不会理解函数行为

它只做一件事：

> **延长一个函数引用的生命周期**

---

## 3. 官方设计目的（准确理解）

综合 React 文档与源码行为，可以得出一个非常明确的定位：

> **useMemo / useCallback 是 identity 稳定工具，用于避免“引用变化”导致的副作用或 memo 边界触发。**

关键词只有三个：

- identity
- 避免不必要的触发
- 边界控制

React 官方**从未**将它们定位为：

- 必用 API
- 性能优化银弹
- 默认最佳实践

---

## 二、哪些第三方类库会“迫使”我们使用 useMemo / useCallback？

### 一个重要前提

> **真正“不得不用”的场景，几乎全部来自 React 之外的世界。**

也就是说：
不是 React 要你用，而是 **第三方系统的契约** 要你用。

---

## 1. 命令式事件 / 订阅型类库（最典型）

### 类库类型

- WebSocket / Socket.io
- EventEmitter
- Observer / Pub-Sub 系统
- 原生 DOM API

### 典型接口形式

```ts
subscribe(callback)
unsubscribe(callback)
```

或：

```ts
addEventListener(type, handler)
removeEventListener(type, handler)
```

### 为什么一定要用？

这些系统的事实是：

- 以 **callback identity** 作为解绑契约
- 不理解 React
- 不理解 render
- 不理解 closure

如果 callback 每次 render 都变：

- 无法正确解绑
- 会造成重复订阅或内存泄漏

### 正确使用方式

```ts
const handler = useCallback((data) => {
  setState(data)
}, [])

useEffect(() => {
  socket.on('message', handler)
  return () => socket.off('message', handler)
}, [handler])
```

这里使用 `useCallback` 的原因是：
**正确性，而不是性能。**

---

## 2. 图表 / 地图 / 可视化类库

### 类库类型示例

- ECharts
- D3
- Three.js
- Mapbox
- Google Maps

### 共同特征

- 内部是命令式系统
- 持有 callback 引用
- 生命周期不受 React 控制

### 为什么一定要用？

这些库通常要求：

- callback identity 稳定
- handler 可解绑
- 注册 / 反注册必须使用同一引用

React render 本身并不会帮你解决这些问题。

---

## 3. 编辑器 / 富文本 / IDE 内核类库

### 类库类型示例

- Monaco Editor
- ProseMirror
- Slate（底层）
- CodeMirror

### 特点

- 内部事件系统复杂
- callback 长期存在
- 高频事件触发

在这些场景中：

> **useCallback 是 React → 命令式内核的边界适配器**

---

## 4. 原生平台 API / 浏览器能力封装

### 类库或能力示例

- ResizeObserver
- IntersectionObserver
- Media / RTC API
- Device / Sensor API

这些 API：

- 强依赖 identity
- 不受 React diff 影响
- 不具备声明式模型

---

## 5. 哪些情况「不是必须」？

### React 内部声明式组件库

如果一个 **React 专用组件库**：

- 要求你高频使用 useCallback
- 才能避免内部 bug 或性能问题

那么通常意味着：

> **库把内部实现细节泄漏给了使用者**

这是抽象设计问题，而不是使用者的问题。

---

## 6. 哪些情况下，使用 useMemo / useCallback **一定是错的**？

下面这些情况，并不是“可讨论的风格差异”，而是**工程上明确不成立的使用场景**。

### 1. 为了“性能安全感”而默认包一层

```ts
const onClick = useCallback(() => {
  doSomething()
}, [])
```

如果满足以下条件：

- 函数只在当前组件内部使用
- 没有作为 effect 依赖
- 没有作为 memo 组件的 props
- 没有传递给第三方命令式系统

那么使用 `useCallback`：

- ❌ 不会带来任何性能收益
- ❌ 反而引入额外的认知与维护成本
- ❌ 掩盖真正的性能瓶颈

这是**典型的“仪式化优化”**，在工程上是明确错误的。

---

### 2. 为了“稳定 deps”，而人为冻结闭包

```ts
const handler = useCallback(() => {
  console.log(state)
}, [])
```

这种写法的本质是：

- 人为切断状态更新
- 依赖陈旧的闭包值
- 用“稳定 identity”换取“逻辑不正确”

这不是优化，而是 **bug 延迟触发器**。

如果逻辑需要最新状态，
那么就不应该通过 `useCallback` 强行稳定它。

---

### 3. 为了通过 lint / code review，而被动添加

```ts
const fn = useCallback(() => {
  ...
}, [a, b, c])
```

如果使用原因仅仅是：

- lint 提示
- 团队约定
- code review 要求

而你无法回答：

> “这个 identity 稳定，对谁是必要的？”

那么这个 `useCallback` **在工程上就是错误的**。

工具不应成为判断的替代品。

---

### 4. 在纯 render 计算中滥用 useMemo

```ts
const value = useMemo(() => a + b, [a, b])
```

如果：

- 计算成本极低
- render 频率正常
- 没有性能瓶颈证据

那么这类 `useMemo`：

- ❌ 不会提升性能
- ❌ 增加代码复杂度
- ❌ 误导未来维护者

React 官方也明确指出：

> **不要为了 useMemo 而 useMemo。**

---

### 5. 在 React 内部抽象中，把 identity 当作 API 契约

如果一个 React 内部模块：

- 要求调用方必须传 stable function
- 否则组件行为异常或性能恶化

那么问题的根源通常在于：

> **抽象边界设计不当，而不是调用方式不规范。**

把 identity 稳定性作为 API 契约，
是 React 抽象中应当避免的设计。

---

### 小结：一个判断标准

如果你无法明确回答以下问题：

> **“这个稳定的 identity，被谁依赖？”**

那么这个 `useMemo / useCallback`，
在工程上就**不应当存在**。

---

## 三、工程层面的结论

### 1. useMemo / useCallback 的角色总结

> **它们是边界工具，不是日常工具。**

- 用于对接 identity 契约
- 用于系统边界
- 用于命令式世界

---

### 2. 一个简单的工程自检规则

当你想使用 `useCallback` 时，可以先问自己一句：

> **我是在解决 React 内部问题，还是在适配 React 之外的系统？**

- 后者 → 用得非常干净
- 前者 → 大概率可以重新设计

---

### 3. 最终总结

> **useMemo / useCallback 的必要性，**  
> **不来自 React 本身，**  
> **而来自 React 之外。**

当它们变成“高频 API”时，  
通常意味着**边界被混淆，或抽象出了问题**。
