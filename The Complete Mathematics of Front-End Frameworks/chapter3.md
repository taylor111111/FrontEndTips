# 第 3 章：代数数据类型（ADT）如何构成 UI 与状态机

## 核心命题

**UI 和业务逻辑全部都可以被表达为：**

> **Sum × Product × 递归**

这不是编程技巧，而是数学上的**可穷举建模方式**。

---

## 3.1 Product Type（乘积型）= UI 或状态的「结构性部分」

例如在工程中：

```ts
type User = {
  name: string,
  age: number
}
```

数学上等价于：

```code
User = String × Number
```

### Product 的工程含义

* **“必须同时存在” → 完备性（completeness）**
* **结构清晰 → 方便自动推理**
* **组件的 props 本质上就是 Product Type**

也就是说：

> Product 描述的是： **一个状态 / UI 由哪些部分「共同构成」**

后续我们会看到：

* Product 决定了结构稳定性
* Sum 决定了分支与状态机
* 递归决定了规模与层级

---

## 3.2 Sum Type（和型）= 状态机、Action、UI 状态

例如：

```ts
type Status = "loading" | "error" | "success"
```

数学上等价于：

```code
Status = Loading + Error + Success
```

### Sum 的工程含义

* **“只能取其中之一” → 排他性（Exclusivity）**
* **“必须覆盖所有情况” → 穷尽性（Exhaustiveness）**
* ``** 本质上是 Sum Type 的判别子（Discriminator）**

也就是说：

> Sum 描述的是： **一个系统在某一时刻「处于哪一种状态」**

这正是：

* UI 状态切换
* Redux Action
* 状态机节点

的统一数学本质。

---

## 3.3 UI Tree = 递归 ADT

最经典的定义是：

```code
VNode =
  Element(Type, Props, List(VNode))
+ Text(String)
```

这本身就是一个 **递归代数数据类型（Recursive ADT）**。

### 递归之美在于：

* **树结构天然可组合**（组件可以无限嵌套）
* **diff 可以基于 ADT 结构匹配**（而不是字符串或 DOM）
* **Fiber 可以按节点粒度进行调度**（中断、恢复、优先级）

也就是说：

> UI Tree 之所以可优化，
> 是因为它是一个**严格定义的递归数据结构**。

---

## 3.4 状态机（State Machine）= ADT × ADT

一个完整的状态机，至少由两部分构成：

* **状态（State）**：Sum Type
* **输入（Action / Event）**：Sum Type

数学上可以写成：

```code
Next : State × Action → State
```

也就是说：

> **状态机 = 状态 ADT × 行为 ADT**

### 在前端中的对应关系

* UI 状态 = Sum Type
* Action = Sum Type
* reducer = 状态转移函数

Redux / XState / Elm Architecture 本质上都在做同一件事：

> **用 ADT 明确描述「允许的状态」与「合法的跃迁」**

这正是为什么：

* 状态可穷举
* 行为可验证
* UI 不会进入“未知态”

因为整个系统，被限制在一个**可证明的状态空间**之内。

---

## 🌟 第 3 卷总结

**现代前端的所有结构，不论 UI / State / Action，都可以归纳为：**

* **Product（结构）**：哪些部分同时存在
* **Sum（变体）**：当前处于哪一种可能
* **Recursion（树）**：规模、层级与组合
* **Projection（渲染）**：状态 → UI 的映射
* **Transition（状态机）**：状态如何合法演化

> **这不是某个框架的设计哲学，
> 而是现代前端在数学上的最终形态。**

当你理解了这一层：

* React / Redux / XState 不再是工具
* 而是一组必然出现的数学结构
