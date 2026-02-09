

# Promise 常见坑位清单（附工程示例）

> 这不是一篇 Promise 教程  
> 也不是 Promise A+ 规范解读  
>  
> 这是一份 **工程向的 Promise 常见事故清单**  
> 目的只有一个：  
> 👉 **当 Promise 被问到 / 被用到时，不踩坑、不背锅、不被发散追问拖死**

---

## 使用原则（先给结论）

一句话工程结论：

> **Promise 本身不复杂，复杂的是它所处的「时间线」是否被正确控制。**

Promise **应该被使用在：**
- 明确触发点（useEffect / onClick）
- 明确生命周期（一次性 / 可取消 / 可回收）

Promise **不应该被使用在：**
- 不受控循环
- 定时器回调
- 隐式并发场景

## Promise 坑位的工程级替代方案

> 当你频繁遇到 Promise 的坑，  
> 问题通常 **不在 Promise 写法本身**，  
> 而在于：Promise 被迫承担了它不擅长的工程职责。

下面是常见 Promise 坑位，对应的 **工程级替代方案**。

---

### 1️⃣ 并发 / 去重 / 缓存 → React Query / SWR

**Promise 的问题：**
- 无法感知是否已有相同请求
- 无法共享请求结果
- 无法统一管理 loading / error / success

**工程替代方案：React Query**

```ts
useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
})
```

React Query 自动提供：
- 请求去重
- 并发合并
- 缓存与过期
- 状态可观测（loading / error / success）

👉 Promise 只负责「怎么拿数据」，  
👉 Query 负责「什么时候拿、拿几次、给谁用」。

---

### 2️⃣ 竞态问题 → React Query / 产品方案

**Promise 的问题：**
- 无法判断「哪个结果应该被丢弃」
- 后发请求可能先返回

**工程替代方案一：React Query**

```ts
useQuery(['search', keyword], fetchSearch)
```

- key 变化 → 自动取消旧请求
- 状态永远以“最新 key”为准

**工程替代方案二：产品级方案（Google Search）**

- 不直接搜索结果
- 先匹配候选词
- 从源头规避竞态

👉 这是 **产品设计 + 架构** 问题，不是 Promise 技巧问题。

---

### 3️⃣ 复杂异步流程 → XState（状态机）

**Promise 的问题：**
- 流程依赖人脑维护
- then / catch 难以表达真实业务状态

**工程替代方案：XState**

```ts
invoke: {
  src: fetchData,
  onDone: { target: 'success' },
  onError: { target: 'failure' },
}
```

XState 提供：
- 显式状态
- 自动中断旧流程
- 可视化流程图

👉 Promise 退化为 effect，  
👉 流程由状态机保证正确性。

---

### 4️⃣ 定时 / 轮询 → 托管式方案

**Promise 的问题：**
- setTimeout / setInterval + Promise 易失控
- 请求堆积、资源泄漏常见

**工程替代方案：React Query**

```ts
useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  refetchInterval: 5000,
})
```

- 自动暂停
- 自动回收
- 与组件生命周期绑定

---

### 5️⃣ Promise + React 生命周期错配 → Server State 抽象

**Promise 的问题：**
- 不理解组件卸载
- 不理解依赖变化

**工程替代方案：**
- React Query / SWR：管理 Server State
- XState：管理流程状态

👉 React 只负责渲染  
👉 异步逻辑交给专门的系统

---

## 1️⃣ 忘记 return Promise（链断事故 · Top 1）

### 错误示例

```ts
fetchData()
  .then(() => {
    doSomethingAsync() // ❌ 没有 return
  })
  .then(() => {
    // 这里会立即执行
  })
```

### 问题本质

- then 只认 **返回值**
- 没 return = then(() => undefined)

### 正确写法

```ts
fetchData()
  .then(() => {
    return doSomethingAsync()
  })
  .then(() => {
    // 正确等待
  })
```

### 工程判断

- ❌ 链式 Promise 中漏 return 是 **高频事故**
- ✅ 面试中只需说一句：  
  > Promise 链必须保证每一段都 return

---

## 2️⃣ Promise.all 语义误解（常被问，但很少用对）

### 常见误解

> Promise.all 是“哪个先完成就返回” ❌

### 实际语义

```ts
Promise.all([p1, p2, p3])
```

- 全部 fulfilled → resolve
- 任意一个 reject → reject

### 工程风险

- 一个失败 → 全部失败
- 很容易被用在 **不该整体失败的场景**

### 工程建议

- 能不用 `Promise.all` 就不用
- 并发管理应交给 **React Query / SWR**

---

## 3️⃣ 在 setTimeout / setInterval 中使用 Promise（时间线事故）

### 错误示例

```ts
setInterval(() => {
  fetchData().then(updateUI)
}, 1000)
```

### 问题本质

- 定时器 + Promise = **不可控并发**
- 可能出现：
  - 请求堆积
  - 响应乱序
  - 资源泄漏

### 工程结论

> **Promise 不应该出现在定时器内部**

正确做法：
- useEffect + 清理
- 或交给轮询型库（React Query refetchInterval）

---

## 4️⃣ async / await + forEach（经典无效 await）

### 错误示例

```ts
list.forEach(async (item) => {
  await fetchItem(item)
})
```

### 实际行为

- forEach 不等待
- 外层无法感知完成时机

### 正确方式

```ts
for (const item of list) {
  await fetchItem(item)
}
```

或

```ts
await Promise.all(list.map(fetchItem))
```

---

## 5️⃣ Promise 竞态（最后返回的不是最后发出的）

### 典型场景

- 搜索输入
- 联想建议
- 快速切换条件

### 错误模型

```ts
search("a")
search("ab")
search("abc")
```

可能返回顺序：
- abc → a → ab

### 工程结论

> **竞态不是 Promise 能单独解决的问题**

解决方案：
- React Query（key + cancellation）
- 或产品方案（Google Search：只匹配候选词）

---

## 6️⃣ Promise + React useEffect（依赖错配）

### 常见问题

```ts
useEffect(() => {
  fetchData().then(setData)
}, [])
```

- 捕获的是 **初始状态**
- 后续 state 变化与 Promise 脱钩

### 工程建议

- 明确 effect 的依赖
- 或使用成熟方案（React Query）

---

## 7️⃣ 吞掉错误（catch 但不处理）

### 错误示例

```ts
fetchData().catch(() => {})
```

### 工程风险

- 错误不可观测
- 状态无法区分成功 / 失败

### 正确做法

```ts
try {
  await fetchData()
} catch (e) {
  reportError(e)
}
```

---

## 总结（工程版）

### Promise 不是问题核心

真正的工程问题是：

- 并发如何管理？
- 竞态如何避免？
- 生命周期如何绑定？
- 状态如何共享？

### 高级工程师判断标准

> **当 Promise 开始变复杂，说明你已经进入工程问题域，应停止手写。**

推荐方案：
- React Query
- SWR
- XState（复杂流程）

---

## 面试安全句式（可直接用）

> Promise 我会用，但我更关注它所处的工程时间线。  
> 并发、竞态、缓存这些问题，我一般交给成熟方案处理。

这句话，**足够、克制、安全**。
