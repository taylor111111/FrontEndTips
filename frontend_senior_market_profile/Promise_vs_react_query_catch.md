

## 4️⃣ 缓存与过期逻辑为什么是 Promise 的“工程死穴”

### 1️⃣ 哪些业务场景必须要缓存，但 Promise 天生做不了？

#### 场景 A：用户资料 / 权限信息

接口示例：
- `/api/user`
- `/api/permissions`

特点：
- 高频读取
- 低频变更
- 多页面 / 多组件复用

业务要求：
- 页面切换时不应反复 loading
- 数据允许短时间旧，但必须可刷新

---

#### 场景 B：列表页 ↔ 详情页切换

接口示例：
- `/api/posts`
- `/api/posts/:id`

业务要求：
- 返回列表页时立刻可用
- 同时后台刷新
- UI 不抖动

---

#### 场景 C：配置数据 / 表单草稿

特点：
- 页面反复进入
- 网络不稳定
- 数据允许缓存，但必须判断是否过期

---

### 2️⃣ 为什么 Promise 无法处理缓存与过期？

Promise 的能力边界只有一条：

> 一次异步计算，只关心成功或失败

Promise 无法表达：
- 结果是否可以复用
- 结果是否过期
- 是否应该后台刷新
- 多组件是否应共享状态

---

### ❌ 典型手写 Promise + 缓存示例（高风险）

```ts
let cache = null
let lastFetchTime = 0

function fetchUserWithCache() {
  const now = Date.now()

  if (cache && now - lastFetchTime < 5 * 60 * 1000) {
    return Promise.resolve(cache)
  }

  return fetch('/api/user')
    .then(res => res.json())
    .then(data => {
      cache = data
      lastFetchTime = now
      return data
    })
}
```

问题：
- 缓存与业务逻辑强耦合
- 无法后台刷新
- 无状态可观测性
- 需求变化成本极高

---

## 3️⃣ React Query 是如何解决这些问题的？

### 核心工程思想

> 缓存不是变量，而是有生命周期、有状态的资源

---

### React Query 写法示例

```ts
useQuery({
  queryKey: ['user'],
  queryFn: fetchUser,
  staleTime: 5 * 60 * 1000,
  gcTime: 30 * 60 * 1000,
})
```

---

### React Query 在工程层面做了什么？

#### ① 用 `staleTime` 模型化“是否新鲜”

- 在 `staleTime` 内：
  - 不重新请求
  - 直接复用缓存
- 超过 `staleTime`：
  - 数据仍可用
  - 自动后台刷新

---

#### ② 用 `gcTime` 管理缓存回收

- 无组件订阅后才开始回收
- 页面切换不导致重复 loading

---

#### ③ 明确区分 UI 状态

```ts
const {
  data,
  isLoading,   // 首次加载
  isFetching,  // 后台刷新
} = useQuery(...)
```

Promise 无法表达该差异。

---

#### ④ 全局共享而非函数级缓存

- 相同 `queryKey`
- 自动共享：
  - 数据
  - 状态
  - 生命周期

---

## 4️⃣ Promise vs React Query 工程能力对比

| 能力维度 | 手写 Promise | React Query |
|--------|-------------|-------------|
| 缓存复用 | ⚠️ | ✅ |
| 过期控制 | ❌ | ✅ |
| 后台刷新 | ❌ | ✅ |
| 状态建模 | ❌ | ✅ |
| 多组件共享 | ❌ | ✅ |
| 可维护性 | ❌ | ✅ |

---

## 工程结论（重要）

> Promise 只能解决“拿结果”
> React Query 管的是“结果的生命周期”

缓存、过期、后台刷新不是异步技巧，而是工程级状态管理问题。

---

## 面试安全收敛用语

> Promise 无法表达数据是否过期或是否该后台刷新，这类问题需要 React Query 这样的工程方案来解决。
