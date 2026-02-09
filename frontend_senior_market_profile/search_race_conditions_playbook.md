

# 搜索竞态工程手册（Search Race Conditions Playbook）

> 本文不是讲 Promise API，而是讲 **真实工程中搜索竞态是如何出现的，以及工程级如何解决**。  
> 适用于：高级前端 / 架构面试 / 真实复杂搜索场景。

---

## 一、问题定义：什么是搜索竞态？

**搜索竞态（Search Race Condition）** 指的是：

- 用户连续输入 / 快速点击
- 多个请求并发发出
- **后发请求先返回**
- UI 被「旧结果」覆盖

```txt
input: a → ab → abc
request: R1 → R2 → R3
response: R1 ← R3 ← R2   ❌
```

---

## 二、为什么 Promise 天生不适合解决搜索竞态？

### 1️⃣ 手写 Promise 的常见写法（错误示例）

```ts
function search(keyword) {
  return fetch(`/api/search?q=${keyword}`).then(res => res.json())
}

input.onChange = async (e) => {
  const result = await search(e.target.value)
  setResult(result)
}
```

### ❌ 问题本质

- Promise **只描述一次异步完成**
- 不知道：
  - 哪个请求“应该被丢弃”
  - 当前 UI 关心的是哪一个
- 后返回的旧 Promise **仍然会 setState**

👉 **Promise 无法表达「竞态优先级」**

---

## 三、补丁方案：AbortController + 序号（勉强可用）

### 2️⃣ AbortController 取消请求

```ts
let controller

function search(keyword) {
  controller?.abort()
  controller = new AbortController()

  return fetch(`/api/search?q=${keyword}`, {
    signal: controller.signal
  }).then(res => res.json())
}
```

### ⚠️ 局限

- 只能取消 HTTP
- **无法保证 UI 一致性**
- 状态仍需手写

---

### 3️⃣ 序号法（最后一次胜出）

```ts
let requestId = 0

async function search(keyword) {
  const id = ++requestId
  const result = await fetchSearch(keyword)

  if (id === requestId) {
    setResult(result)
  }
}
```

### ⚠️ 问题

- 容易漏写
- 与组件生命周期强耦合
- 难以复用

---

## 四、工程解法一：React Query（推荐）

### 4️⃣ queryKey = 竞态的“工程语义”

```ts
useQuery({
  queryKey: ['search', keyword],
  queryFn: ({ signal }) => fetchSearch(keyword, signal)
})
```

### React Query 做了什么？

- `queryKey` 定义 **“我现在关心的是什么”**
- key 变化：
  - 自动取消旧请求
  - 旧结果不会进入 UI
- **状态永远以“最后一次 key”为准**

### ✔️ 解决了什么？

| 问题 | Promise | React Query |
|---|---|---|
| 谁是有效请求 | ❌ | ✅ |
| 自动取消 | ❌ | ✅ |
| 状态一致性 | ❌ | ✅ |
| 可复用 | ❌ | ✅ |

---

## 五、工程解法二：Google Search 的产品级方案

### 5️⃣ 搜索不是“结果匹配”，而是“候选词匹配”

Google Search 的关键设计：

- 不直接搜索结果
- 先返回 **候选关键词**
- 搜索行为是 **离散化的**

```txt
输入 → 候选词 → 用户确认 → 搜索结果
```

### 本质

> **通过产品拆分，规避竞态**

---

## 六、工程解法三：XState（显式竞态建模）

### 6️⃣ 用状态机表达“谁能赢”

```ts
createMachine({
  initial: 'idle',
  states: {
    idle: {
      on: { INPUT: 'loading' }
    },
    loading: {
      invoke: {
        src: 'search',
        onDone: 'success',
        onError: 'error'
      },
      on: {
        INPUT: { actions: 'cancelAndRestart' }
      }
    }
  }
})
```

### 优势

- 输入 / 请求 / 取消 **显式建模**
- 没有“幽灵 Promise”
- 非常适合复杂搜索流

---

## 七、总结：该选哪个？

| 场景 | 推荐方案 |
|---|---|
| 简单搜索 | React Query |
| 高复杂搜索 | XState |
| 搜索产品 | 产品级拆分 |
| 面试解释 | React Query + Google Search |

---

## 结论（重要）

> **搜索竞态不是 Promise 问题，而是工程建模问题。**  
> 能把这件事讲清楚，已经是高级工程师的上限。
