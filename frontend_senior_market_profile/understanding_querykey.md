

# React Query 中的 queryKey：并发去重的工程语义

这个问题问得**非常准**，而且正好卡在 React Query 的**工程语义核心点**上。  
我们把话说清楚，不绕。

---

## 一、先给结论（重要）

**不是：**

> 只有第一个请求有效，后面的请求不会发出去

**而是：**

> 同一个 `queryKey` 在同一时间窗口内，只会存在 **一个真实的进行中请求（in‑flight request）**  
> 后续触发的，不是“新请求”，而是**复用 / 订阅**这个正在进行的请求

👉 这是 **并发去重（request deduplication）**，不是“忽略点击”。

---

## 二、分情况精准说清楚

### 1️⃣ 多个组件同时使用 `useUser()`

```ts
function useUser() {
  return useQuery({
    queryKey: ['user'],
    queryFn: fetchUser,
  })
}
```

```tsx
function UserProfile() {
  const { data } = useUser()
  return <div>{data?.name}</div>
}

function UserSidebar() {
  const { data } = useUser()
  return <div>{data?.role}</div>
}
```

### 实际发生了什么？

**时间线：**

1. `UserProfile` 挂载  
   - React Query 发现 `['user']` 不存在  
   - 👉 发起 **一次真实 HTTP 请求**

2. `UserSidebar` 同时挂载  
   - React Query 发现：
     - `['user']` 已存在
     - 状态是 `fetching`
   - 👉 **不会再发请求**
   - 👉 只是“订阅”这个正在进行的请求

3. 请求返回  
   - 两个组件**同时收到结果**
   - UI 同步更新

### 结论

- ✅ HTTP 只发了一次  
- ✅ 多个组件共享结果  
- ❌ 没有“第二次请求”  
- ❌ 也不是“后面的被忽略”  

👉 它们在**等同一辆车**

---

## 三、和「多次点击」是不是一回事？

### 情况 A：快速多次点击，`queryKey` 不变

```tsx
<button onClick={() => refetch()}>Refresh</button>
```

或：

```ts
useQuery({
  queryKey: ['user'],
  queryFn: fetchUser,
})
```

### 行为规则

| 场景 | React Query 行为 |
|----|----|
| 当前没有请求 | 👉 发起请求 |
| 当前已有请求进行中 | ❌ 不再新发 |
| 请求完成后再次触发 | ✅ 会重新发 |

👉 **并发期间只跑一次，串行可以重复**

---

## 四、为什么 React Query 要这么设计？

因为在工程上：

> **并发多次请求同一资源 = 浪费 + bug**

React Query 的默认哲学是：

- 同一个资源（`queryKey`）
- 同一时间
- 只需要 **一个真实请求**

如果你**真的**希望“每次点击都发请求”，那说明：

- 这是一个 **command / mutation**
- ❌ 不是 query

---

## 五、什么时候「每次都应该发请求」？

👉 用 `useMutation`，不是 `useQuery`

```ts
const mutation = useMutation({
  mutationFn: fetchUser,
})

<button onClick={() => mutation.mutate()}>
  Force Request
</button>
```

### 工具选择对照表

| 场景 | 正确工具 |
|----|----|
| 读取共享资源 | useQuery |
| 用户明确触发动作 | useMutation |
| 不可去重操作 | useMutation |

---

## 六、回到你的问题本身（非常高级）

> “和多次点击一样，只有第一个请求是有效的？”

**严格答案是：**

- ❌ 不是“只有第一个有效”
- ✅ 是“同一时间只允许一个 in‑flight 请求”
- ✅ 后续触发：
  - 不会新建 Promise
  - 不会新发 HTTP
  - 会共享状态与结果

👉 这是 **工程级并发控制**，不是 Promise 能力。
