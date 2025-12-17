# Chapter 6 · Update（优先级的源头）完整枚举

> 本章目标：**严格枚举 React Fiber 中「优先级（Priority / Lane）」的所有“源头 Update”**，
> 即：**是什么事件 / 机制，最初把一个 Update 放入调度系统，并赋予它优先级。**

---

## 一、核心结论（一句话版）

> **Priority 不是 Fiber 自己产生的，而是由 Update 的“来源”决定的。**
>
> Fiber / Scheduler 只做两件事：
>
> 1. 接收 Update（已经带有优先级）
> 2. 根据优先级决定 *什么时候*、*从哪个 root* 开始遍历

所以我们只需要回答一个问题：

> **Update 从哪里来？**

---

## 二、Update 的数学抽象

在 React 中，所有更新都可以抽象为：

```text
Update = ⟨ targetFiber, payload, lane ⟩
```

其中：

* `targetFiber`：更新作用的 Fiber（组件）
* `payload`：状态变化（state / props / flags）
* `lane`：**优先级来源的最终编码**

本章只关心一件事：

> **lane 是由什么 Update Source 决定的？**

---

## 三、Update Source 总览（严格枚举）

React 中所有 Update 的优先级来源，可以被**完整且互斥**地划分为 6 大类：

1. 用户输入类（Input-driven Updates）
2. 同步控制类（Synchronous Control Updates）
3. 渲染派生类（Render-phase / Transition Updates）
4. 异步结果类（Async Resolution Updates）
5. 副作用回流类（Effect / Subscription Updates）
6. 系统内部维护类（Internal / Maintenance Updates）

下面逐一展开。

---

## 四、① 用户输入类 Update（最高优先级源头）

### 定义

> **由真实用户交互直接触发的 Update。**

这是 React 中**最不允许被延迟**的一类更新。

### 典型来源

* `onClick`
* `onInput`
* `onChange`
* `onKeyDown / onKeyUp`
* `pointer / mouse / touch` 事件

### 特征

* 来源：浏览器原生事件
* 用户正在“等待反馈”
* 必须尽快反映到 UI

### Lane 级别

```text
DiscreteEventPriority
→ SyncLane / InputContinuousLane
```

### 数学地位

```text
Update_source = UserEvent
priority = MAX
```

---

## 五、② 同步控制类 Update（强同步）

### 定义

> **开发者显式要求“现在就要更新”的 Update。**

### 典型来源

* `setState`（非并发模式 / legacy sync）
* `flushSync(() => setState())`
* 生命周期中的同步更新（部分 legacy 行为）

### 特征

* 不一定来自用户
* 但语义上要求：**阻塞式完成**

### Lane 级别

```text
SyncLane
```

### 数学地位

```text
Update_source = ExplicitSync
priority = VERY_HIGH
```

---

## 六、③ 渲染派生类 Update（Transition / Render 派生）

### 定义

> **在一次渲染语义中“顺带产生”的更新。**

### 典型来源

* `startTransition(() => setState())`
* 并发模式下的“非紧急更新”
* UI 状态切换（tab、filter、list 重排）

### 特征

* 用户不要求立刻完成
* 可以被打断、延迟、重排

### Lane 级别

```text
TransitionLane
```

### 数学地位

```text
Update_source = IntentionalDefer
priority = MEDIUM
```

---

## 七、④ 异步结果类 Update（Promise / IO 回流）

### 定义

> **由异步任务完成后回流到 UI 的更新。**

### 典型来源

* `fetch.then(setState)`
* `async / await` 结果
* `Suspense` resolve / reject

### 特征

* 不由用户“当下操作”直接触发
* 通常是后台完成的结果

### Lane 级别

```text
DefaultLane / TransitionLane
```

### 数学地位

```text
Update_source = AsyncResolution
priority = NORMAL
```

---

## 八、⑤ 副作用回流类 Update（Effect / Subscription）

### 定义

> **在 effect / 订阅回调中触发的二次更新。**

### 典型来源

* `useEffect(() => setState())`
* `useLayoutEffect`
* 外部 store / observable / subscription

### 特征

* 发生在 commit 之后
* 本质是“世界 → React”的回流

### Lane 级别

```text
DefaultLane
```

### 数学地位

```text
Update_source = EffectFeedback
priority = LOW
```

---

## 九、⑥ 系统内部维护类 Update（最低优先级）

### 定义

> **React 内部为一致性、回收、修正而产生的更新。**

### 典型来源

* Suspense retry
* Cache / memo / ref consistency
* Offscreen / hidden subtree 维护
* Idle 时的清理任务

### 特征

* 用户不可感知
* 可以无限期推迟

### Lane 级别

```text
IdleLane
```

### 数学地位

```text
Update_source = SystemMaintenance
priority = MIN
```

---

## 十、完整优先级源头对照表（终极版）

| Update 来源  | 示例              | Lane         | 优先级   |
| ---------- | --------------- | ------------ | ----- |
| 用户输入       | click / input   | Sync / Input | ⭐⭐⭐⭐⭐ |
| 强同步控制      | flushSync       | Sync         | ⭐⭐⭐⭐  |
| Transition | startTransition | Transition   | ⭐⭐⭐   |
| 异步结果       | promise resolve | Default      | ⭐⭐    |
| Effect 回流  | useEffect       | Default      | ⭐⭐    |
| 系统维护       | idle cleanup    | Idle         | ⭐     |

---

## 十一、关键理解（非常重要）

> **React 从不“计算”优先级，只是“继承”优先级。**

* 优先级在 Update 诞生时就已确定
* Fiber 只是执行单元
* Scheduler 只是时间分配器

真正决定系统行为的，是：

> **哪个 Update 先进入队列。**

---

## 十二、与前面章节的严格对应

* Chapter 6-1：可中断拓扑排序
* Chapter 6-2：Dynamic Root Selection
* **本章（6-x）：Priority 的源头定义**

三者共同构成：

> **Fiber = 带优先级的、可中断、可重排的拓扑遍历系统。**

这已经是 React Fiber 的“数学全貌”。
