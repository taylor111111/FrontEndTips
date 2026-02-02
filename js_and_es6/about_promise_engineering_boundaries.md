

# Promise Engineering Boundaries  
## Promise 在前端工程中的合法场景与越界红线

> **Promise 不是复杂，  
> 而是经常被用在不属于它的事实场景里。**

这篇文章不是 Promise API 教程，也不是 async / await 用法总结。  
它只回答一个工程问题：

> **Promise 在前端工程中，  
> 哪些场景是“必须且合理”的，  
> 哪些场景应该被明确禁止。**

---

## 1. 一个工程前提：语言服务于事实存在

Promise 并不是为了“写起来优雅”而存在的。  
它之所以被设计出来，是因为前端工程中长期存在这样一类**客观事实**：

- 结果一定来自未来
- 完成时间不可预测
- 无法同步化
- 结果只出现一次（成功或失败）

**Promise 是对这类事实的语言建模。**

一旦脱离这些事实场景，Promise 的使用就会迅速演变为工程风险。

---

## 2. Promise 在前端工程中【常用且必须】的场景

这些场景是 Promise 的**合法领地**。

---

### 2.1 I/O 边界（最核心、不可替代）

**典型例子**

- `fetch / axios`
- IndexedDB
- File API
- Clipboard API
- Web Worker 返回结果

```ts
fetch('/api/user').then(res => res.json())
```

**为什么必须用 Promise**

- 数据来自外部世界
- 返回时间无法预测
- 不存在同步替代方案
- 结果天然是一次性的

👉 这是**事实本身**，不是设计选择。

---

### 2.2 逻辑层的一次性结果组合（非 UI）

```ts
Promise.all([
  loadUser(),
  loadConfig(),
  loadPermissions(),
])
```

**适用前提**

- 位于业务逻辑层 / service 层
- 不直接操作 UI 状态
- 不依赖组件生命周期

Promise 在这里是**纯结果管道**。

---

### 2.3 async / await 作为“顺序表达”工具

```ts
async function submit() {
  await validate()
  await save()
  await notify()
}
```

这里的 async / await：

- 只负责表达顺序
- 不承担生命周期管理
- 不直接驱动 UI 世界

👉 合法，但边界必须清楚。

---

## 3. Promise【不常用，但合理存在】的场景

这些不是主流，但工程上是合理的。

---

### 3.1 Worker / WASM 的一次性计算任务

```ts
worker.runTask().then(result => ...)
```

**特征**

- 计算成本高
- 不适合主线程
- 不关心中间过程

Promise 只负责**接收结果**。

---

### 3.2 对旧式 callback API 的边界封装

```ts
function readFile(file): Promise<string> {
  return new Promise(...)
}
```

**注意**

- 只用于 API 边界
- 不向业务层扩散 Promise 语义

---

## 4. Promise【极少用，但确实存在】的场景

---

### 4.1 应用启动期的一次性初始化

```ts
await initApp()
```

**前提**

- 发生在 React render 之前
- 或发生在应用 bootstrap 阶段
- UI 世界尚未存在

---

### 4.2 工具层的纯异步算法

```ts
async function hashLargeFile() {}
```

- 与 UI 无关
- 与状态无关
- 纯工具属性

---

## 5. 明确禁止使用 Promise 的工程场景（红线）

这一部分是本文的重点。

---

### ❌ 5.1 UI 生命周期管理

```ts
useEffect(() => {
  fetchData().then(setState)
}, [])
```

**问题**

- Promise 不知道组件是否已 unmount
- 不知道结果是否已过期
- 不具备生命周期感知能力

👉 **UI 生命周期 ≠ Promise 事实模型**

---

### ❌ 5.2 需要“可取消 / 可忽略”的交互场景

**典型例子**

- 搜索联想
- 实时校验
- hover 请求
- 输入驱动请求

**原因**

- Promise 不可取消
- 只能完成，不能失效

这类问题的本质是**时间线竞争**。

---

### ❌ 5.3 事件驱动的状态流控制

```ts
onClick(async () => {
  await doSomething()
  setState(...)
})
```

**问题**

- 把事件时间线与 Promise 时间线耦合
- 状态语义不清晰
- 调试成本高

事件应该只负责**触发状态变化**，  
而不是“等待未来”。

---

### ❌ 5.4 长期存在的逻辑（订阅 / 监听 / 流）

- 用 Promise 模拟流
- 用 Promise 表示 ongoing 状态

Promise 是**一次性结果模型**，  
不适用于持续状态。

---

## 6. 一句工程总结

> **Promise 只适用于：  
> 单次、未来、不可同步、不可取消的结果。**

一旦问题涉及：

- 生命周期
- 状态有效性
- 用户交互
- 时间线竞争

Promise 就不再是合适的语言。

---

## 7. 结语

Promise 本身并不危险。  
危险的是**语言脱离事实场景的滥用**。

> **清楚 Promise 的边界，  
> 比熟练 Promise 的 API 更重要。**

工程稳定性，  
永远来自对边界的尊重。
