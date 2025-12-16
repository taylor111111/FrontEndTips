# 第 6.3 节

**Dynamic Root Selection 的数学模型**

（优先级队列 + 时间片 + 局部根）

---

## 6.3.1 背景：为什么 Fiber 需要“动态根节点”？

在传统 DFS 渲染中：

```code
root → children → grandchildren → …
```

遍历路径是固定的；

但 Fiber 目标是：

> 允许高优先级任务插队、允许暂停、允许从任意节点恢复渲染。

要做到这一点，React 引入一个强大的概念：

⭐ **Dynamic Root Selection（动态根选择）**

数学解释：

> 渲染是一条拓扑链，但这条链条的入口点（root）可以在运行中改变。

---

## 6.3.2 核心结构：三大数学组件

Dynamic Root Selection 基于三个数学结构：

---

### ① 优先级队列（Priority Queue）

调度器维护一个工作集合：

```code
W = { w₁, w₂, …, wₙ }
```

每一个 WorkItem wᵢ（即某个 Fiber 更新）有一个优先级 pᵢ：

```code
pᵢ ∈ ℕ   // 0 表示最高优先级
```

React 18 用的是 “lanes”：

```code
Lane = 位掩码 (bitmask)
More significant bits = lower priority
```

数学等价于：

> 一个用位掩码实现的多级优先队列。

---

### ② 时间片（time slice）

渲染采用 cooperative scheduling：

```code
每次执行不超过 Δt（通常 ≈ 5ms）
```

记：

```code
T = 每一帧可用的 CPU 时间
δ = 一次 Fiber 执行花费的时间
```

调度条件：

```code
while (timeRemaining > δ) { performUnitOfWork(); }
```

数学上等价于：

> 在局部时间窗口内对拓扑序列的局部前缀。

---

### ③ 局部根（Local Root）

每个高优事件（click、input）会产生一个更新：

```code
update ∈ W
update.fiber = 某一棵子树的根节点
```

调度器会在优先级队列中挑选一个“局部 root”：

```code
root* = argmin_priority(W)
```

其中：

* root* = 本次渲染的“入口点”
* priority(root*) 最优（数值最小或 lane 最紧急）

这就是 Dynamic Root Selection 的本质。

---

## 6.3.3 三者如何结合？（正式模型）

### 模型定义

我们把渲染看成一个可中断的拓扑序列：

```code
Σ = 所有 Fiber 的 begin/complete 步骤集合
```

调度器要在 Σ 中选出一个可执行序列：

```code
σ = < s₁, s₂, …, sₖ >
```

满足：

* 每一步 sᵢ 都在可达状态空间内（valid Fiber）
* 总时间不超过 Δt 时间片
* Sequence 的起点是 root*

因此：

```code
σ = prefix(topological_order(root*)),  |σ| · δ ≤ Δt
```

这意味着：

> 不同 root* 对应不同拓扑“局部路径”。
> 调度器改变 root = 改变拓扑序列的起点 = 改变渲染路径。

---

## 6.3.4 Dynamic Root Selection 的数学流程（精确描述）

假设当前有 3 个更新：

* w₁ (click) → priority = 0 → fiber A
* w₂ (promise resolve) → priority = 4 → fiber C
* w₃ (低优) → priority = 8 → fiber root

则优先级队列：

```code
W = {
  A: p=0,
  C: p=4,
  root: p=8
}
```

调度器选择：

```code
root* = A
```

于是 Fiber 遍历顺序从 A 的子树开始：

```code
A → A.child → … → A.sibling.child → …
```

执行若干 steps 后（时间片结束），暂停。

下一帧，根据当前优先级继续：

```code
root* = A   // 只要 lane 不变
```

当 A 的 subtree 完成后，调度器重新从队列选：

```code
root* = C
```

再继续：

```code
C → C.child → C.sibling → …
```

之后才轮到最低优先级的 root。

---

## 6.3.5 这跟“动态改变遍历顺序”有什么关系？

你修正得非常对：React 并不是直接重排 traversal，而是：

> 通过动态选择 root，来改变 traversal 起点，从而改变整条 DFS 的可达路径。

数学形式：

```code
σ = prefix(topological_order(root*(W)))
```

因此：

* root* 是动态的
* topological_order 固定，但路径起点不同
* 输出的序列 σ 每次都不同

这就是：

⭐ **“动态重选根节点 = 动态改变遍历顺序”**

是唯一数学上精确成立的形式。

---

## 6.3.6 最终总结（放在卷末）

> **Dynamic Root Selection =**
> 在优先级队列驱动下，调度器动态选择 Fiber 子树作为新的 traversal entry，
> 渲染序列 σ 是该子树拓扑排序的时间片前缀。
>
> 因此，序列的全局顺序随 root 的变化而变化。
> 这使得 Fiber 渲染具备调度能力，而不破坏树的拓扑结构。
