## 业务场景：保存草稿（自动保存 + 网络抖动 + 429/500）

### 典型 UI 需求：

- 输入时 800ms debounce 自动保存
- 网络抖动时：可重试（但别把服务打爆）
- 业务错误 (400/422)：不该重试，应提示用户
- 多次失败：进入“降级模式”（比如提示“离线保存，本地暂存”）
- 用户仍可手动点“重试”

---

## 1️⃣ 手写 Promise 的常见失控版本（❌ 坑示范）

### 坑 1：不区分错误类型，逢错就重试

### 坑 2：回退逻辑散落在多个地方（UI / 请求 / 定时器）

### 坑 3：重复触发导致并发重试风暴

```ts
async function saveDraftBad(payload: any) {
  // ❌ 不区分网络错误 vs 业务错误
  // ❌ 没有 backoff / jitter
  // ❌ 上层可能多次调用，造成并发重试风暴
  let attempt = 0;

  while (attempt < 5) {
    try {
      const res = await fetch('/api/draft', {
        method: 'POST',
        body: JSON.stringify(payload),
      });
      if (!res.ok) throw new Error('failed'); // ❌ 粗暴
      return await res.json();
    } catch (e) {
      attempt++;
      await new Promise(r => setTimeout(r, 1000)); // ❌ 固定等待：容易打爆服务
    }
  }

  throw new Error('save failed');
}
```
 ### 为什么说这是“工程失控”？
因为“重试策略/回退策略”本质上是一个系统行为，不是一个函数内部的 while 循环能可靠表达的。它需要：

- 统一的 retry/backoff/jitter

- 统一的 error classification（网络 vs 业务 vs 权限）

- 统一的最大尝试次数 + 熔断/降级

- 统一的可观测性（attempt count / next retry in / reason）

---

## 2️⃣ React Query：工程化可控的重试 + backoff ✅

**目标**：把“重试策略”变成**明确配置**，而不是散落的 `while`。

### 2.1 错误分类工具


```ts
type ApiError = {
  status?: number;
  message: string;
  code?: string;
};

function toApiError(e: unknown): ApiError {
  // fetch 的网络错误通常是 TypeError
  if (e instanceof TypeError) return { message: e.message };

  // 服务端返回的结构化错误
  if (typeof e === 'object' && e && 'status' in e) {
    return e as ApiError;
  }

  return { message: 'Unknown error' };
}

function isRetryable(err: ApiError) {
  // ✅ 网络错误（无 status）
  if (!err.status) return true;

  // ✅ 5xx：服务端错误
  if (err.status >= 500) return true;

  // ✅ 429：限流，可重试但应更长 backoff
  if (err.status === 429) return true;

  // ❌ 4xx：业务错误，不应重试
  return false;
}
```

### 2.2 用 useMutation 实现“保存草稿” + retry / backoff

保存是 mutation，不是 query。

```ts
import { useMutation } from '@tanstack/react-query';

async function saveDraft(payload: any) {
  const res = await fetch('/api/draft', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload),
  });

  if (!res.ok) {
    // 把 status 带出去给 retry 函数判断
    throw { status: res.status, message: await res.text() } satisfies ApiError;
  }

  return res.json();
}

export function useSaveDraft() {
  return useMutation({
    mutationFn: saveDraft,

    // ✅ 统一：是否重试
    retry: (failureCount, error) => {
      const err = toApiError(error);
      if (!isRetryable(err)) return false;
      return failureCount < 3; // 最多 3 次
    },

    // ✅ 统一：backoff（指数退避 + 上限）
    retryDelay: (attemptIndex, error) => {
      const err = toApiError(error);
      const base = err.status === 429 ? 2000 : 500; // 429 更保守
      const delay = Math.min(30_000, base * 2 ** attemptIndex);
      return delay;
    },
  });
}
```

这里的关键不是 `useMutation` 这个 API，
而是：**失败行为被集中建模为策略，而不是散落在控制流中**。

- retry 决定“什么错误值得再试”
- retryDelay 决定“系统如何对失败保持克制”
- UI / debounce / 定时器不再参与失败控制

### 2.3 UI 侧如何“不会失控”

UI 层的职责只有三件事：
- 描述用户意图（输入 / 点击）
- 触发 mutation
- 渲染“当前系统状态”

它**不负责**：
- 决定是否重试
- 决定 retry 间隔
- 判断错误类型

```ts
function DraftEditor() {
  const [text, setText] = React.useState('');
  const save = useSaveDraft();

  // debounce 自动保存
  React.useEffect(() => {
    const id = window.setTimeout(() => {
      save.mutate({ text });
    }, 800);

    return () => window.clearTimeout(id);
  }, [text]);

  return (
    <div>
      <textarea value={text} onChange={(e) => setText(e.target.value)} />

      {/* ✅ 状态可观测：isPending / isError / failureCount */}
      {save.isPending && <p>Saving...</p>}
      {save.isError && (
        <p>
          Save failed.{' '}
          <button onClick={() => save.mutate({ text })}>Retry now</button>
        </p>
      )}
      {save.isSuccess && <p>Saved</p>}
    </div>
  );
}
```

这里 UI **永远不会失控**，因为：

- debounce 只影响「什么时候触发保存」
- 是否重试、何时重试，由 React Query 接管
- 多次点击 / 多次输入，不会制造并发重试风暴
- 用户看到的，是**系统真实状态**，而不是 Promise 的瞬时结果


---

## 3️⃣ React Query 到底帮你「工程化」了什么？

这不是“语法糖”，而是**系统行为被显式建模**。

- ✅ **retry / backoff 成为统一策略**
  - 不再散落在 `while + setTimeout`
  - 可读、可改、可审计
  - 可以被代码审查、被复用、被调整

- ✅ **failureCount / isPending / isError 是状态模型**
  - 不是你手写的临时变量
  - 描述的是「系统当前所处阶段」
  - UI 只消费状态，不推理因果

- ✅ **避免策略漂移**
  - 不会出现：
    - A 页面 retry 3 次
    - B 页面 retry 5 次
    - C 页面无限 retry
  - 所有失败行为来自同一套策略来源

- ✅ **更容易和 Devtools / 日志系统串起来**
  - React Query Devtools 能看到：
    - 请求是否在 retry
    - 失败了几次
    - 当前是否处于 fetching / idle
  - 日志与监控可以基于同一状态模型埋点

---

> 一句话总结：
>
> **React Query 做的不是“帮你写 Promise”，  
> 而是把“失败处理”从控制流，提升为工程级状态与策略。**
