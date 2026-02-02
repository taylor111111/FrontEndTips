

# Hooks：时间，而不是 API

这篇不是 Hooks API 说明书，也不是最佳实践合集。

它回答的是一个更值钱的问题：

> **Hooks 真正复杂的地方是什么？**

答案只有一个关键词：**时间（time）**。

---

## 1. Hooks 的本质模型

Hooks 可以被理解为一句话：

> **Hooks 是 React 暴露出来的「时间接口」**

- render：声明 UI 与状态的关系（纯函数）
- hooks：在 render 的时间轴上，挂接状态、订阅和副作用
- effect：React 与外部世界交互的边界

所以 Hooks 的核心不是：
- useEffect 怎么用
- deps 写不写全

而是：

> **你是否能正确理解：某段代码，运行在“哪一个时间点”**

---

## 2. 最值钱的 Hooks 知识点（工程视角）

### 2.1 Render ≠ Effect（时间分离）

这是 Hooks 的第一性原理。

```js
function Comp() {
  console.log('render')
  useEffect(() => {
    console.log('effect')
  })
}
```

- render：可能被打断、重试、丢弃
- effect：只在 commit 后执行

**结论：**
> render 阶段必须是纯的  
> 一切副作用都必须发生在 effect / event handler 中

---

### 2.2 闭包不是 bug，是设计前提

```js
function Comp({ count }) {
  useEffect(() => {
    setTimeout(() => {
      console.log(count)
    }, 1000)
  }, [])
}
```

这里打印的是：
- effect 创建时的 `count`
- 而不是「最新的 count」

**这是 Hooks 的设计前提，而不是缺陷。**

Hooks 的规则是：

> effect 捕获的是「创建该 effect 的那一次 render 的变量」

这直接引出了三种工程解法：

1. **deps 驱动 effect 重新创建**
2. **useRef 作为逃生舱**
3. **将变化移入 effect 内部**

---

### 2.3 deps ≠ “监听变量变化”

一个极其常见、但极其危险的误解：

> ❌ props 变了，effect 会自动用新值  
> ✅ effect 是否重跑，只由 deps 决定

deps 的真实含义是：

> **当这些依赖变化时，我需要一个“新的副作用实例”**

而不是：
> “我想用到最新值”

---

### 2.4 useRef 的真实定位

useRef 不是为了“固定函数”  
而是为了**跨时间保存可变状态**

```js
const ref = useRef(value)
ref.current = value
```

useRef 的工程语义是：

> **我知道这是可变的，但我不希望它触发重新 render**

典型使用场景：
- interval / timeout 中读取最新值
- 保存外部系统句柄
- 避免在 deps 中引入不稳定依赖

---

### 2.5 StrictMode 不是“多跑一次”，而是时间压力测试

在 React 18（dev + StrictMode）中：

- render 可能执行多次
- effect 会经历：
  - setup → cleanup → setup

目的不是折磨开发者，而是验证：

> **你的副作用，是否具备“可重入 + 可清理”的工程健壮性**

如果一个 effect 在 StrictMode 下崩了，
那它在并发 / 中断场景下也不安全。

---

## 3. Hooks 中最常见、也最值钱的错误模式

### 3.1 在 render 中触发副作用

```js
if (cond) setState(...)
```

这是逻辑错误，而不是写法问题。

原因：
- render 可能被多次调用
- 副作用必须是幂等、可控的

---

### 3.2 为了“修 deps”而引入 useRef

这是典型的反模式。

判断标准只有一个：

> 你是在修“性能问题”，  
> 还是在掩盖“时间模型没想清楚的问题”？

---

### 3.3 误把 effect 当“同步响应 props 的地方”

effect 不是 props 的延伸，
而是 React 对外界的接口层。

---

## 4. 一个工程级总结

Hooks 不是为了让你：
- 少写 class
- 写更炫的函数式代码

Hooks 的真正价值在于：

> **强迫你显式思考时间、状态和副作用的边界**

如果你能清楚回答这三个问题：
1. 这段代码运行在哪个时间点？
2. 捕获的是哪一次 render 的数据？
3. 是否允许被重复执行 / 清理？

那么：
- Hooks 不会成为坑
- useEffect 不会变成玄学
- 面试中的“刁钻问题”会自动消失

---

## 5. 工程态度（也是面试安全答案）

> Hooks 的难点不在 API，  
> 而在时间模型。

> 在不确定时间语义是否安全的情况下，  
> 我会选择更保守、更可预测的写法。

这不是逃避复杂度，
而是**工程成熟度**。
