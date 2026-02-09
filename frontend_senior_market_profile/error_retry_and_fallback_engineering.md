

# 错误重试与回退策略的工程治理  
Error, Retry & Fallback Engineering

> 本文不是 Promise 使用指南，而是一次**工程级失败治理**的拆解：  
> 当系统必然失败时，如何让失败变得**可控、可观测、可恢复**。

---

## 6️⃣ 错误重试 / 回退策略失控

### 坑的本质

- **Promise 失败即失败**
- 错误语义被压缩为 `reject`
- 重试 / 回退逻辑只能由调用者“外包式处理”
- 难以区分：
  - 网络错误 vs 业务错误
  - 可重试错误 vs 不可重试错误
  - 临时失败 vs 终态失败

```ts
fetchData()
  .then(render)
  .catch(showError)
```

👉 上述代码**无法表达任何策略信息**。

---

## 真实业务场景

### 场景 1：列表接口偶发 500

- 是否需要自动重试？
- 重试几次？
- 是否指数退避？
- 用户是否感知？

### 场景 2：支付 / 下单失败

- ❌ 绝不能自动重试
- ❌ 不能静默失败
- 必须进入「人工确认 / 回退流程」

👉 **错误不是异常，而是状态分叉点。**

---

## ❌ 手写 Promise 能解决什么？不能解决什么？

### 能解决的

- 捕获异常
- 写 if / else

### 解决不了的

- 统一重试策略
- 请求间策略复用
- 错误类型建模
- 回退路径可验证性

```ts
async function loadData() {
  try {
    return await fetchData()
  } catch (e) {
    if (retryCount < 3) {
      retryCount++
      return loadData()
    }
    throw e
  }
}
```

❌ 问题：
- retryCount 是隐式状态
- 无退避策略
- 调用方无法感知过程

---

## ✅ React Query：错误 = 状态 + 策略

### 核心思想

- 错误不是异常，是 **query 状态**
- 重试是声明式策略
- 回退是 UI 决策，而非 Promise 链

### 示例代码

```ts
useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  retry: (failureCount, error) => {
    // 网络错误才重试
    return error.status >= 500 && failureCount < 3
  },
  retryDelay: attempt => Math.min(1000 * 2 ** attempt, 30000),
})
```

### React Query 做了什么？

- 自动区分：
  - `isError`
  - `failureCount`
  - `isFetching`
- 自动退避
- 自动终止重试
- 自动通知 UI

```tsx
if (isError) {
  return <ErrorView retry={refetch} />
}
```

👉 **错误治理从 Promise 层，提升到系统层。**

---

## ✅ XState：错误是状态，不是异常

### 思想升级

> 错误不是“抛出”，而是「状态迁移」

### 状态机示例

```ts
createMachine({
  initial: 'loading',
  states: {
    loading: {
      invoke: {
        src: fetchData,
        onDone: 'success',
        onError: 'retrying'
      }
    },
    retrying: {
      after: {
        2000: 'loading'
      }
    },
    success: {},
    failed: {}
  }
})
```

### 优势

- 每条回退路径**显式存在**
- 可测试、可推演
- 不依赖调用者“记得处理”

---

## 对比总结

| 维度 | Promise | React Query | XState |
|----|----|----|----|
| 错误建模 | ❌ reject | ✅ 状态 | ✅ 状态 |
| 重试策略 | ❌ 手写 | ✅ 内建 | ✅ 显式 |
| 回退路径 | ❌ 隐式 | ⚠ UI 控制 | ✅ 状态机 |
| 可维护性 | ❌ | ✅ | ✅✅ |

---

## 结论

> **错误重试与回退不是 API 问题，而是系统设计问题。**

- Promise 只能「感知失败」
- React Query 能「治理失败」
- XState 能「规划失败」

这就是工程级抽象的分水岭。
