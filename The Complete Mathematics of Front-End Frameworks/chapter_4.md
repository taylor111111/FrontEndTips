# 第 4 卷：Reducer = 状态机（State Machine）的演化函数

## 本卷核心命题

```code
reducer : State × Action → State
```

这不是某个库的 API 设计，
而是**有限状态机（FSM）/ 扩展状态机（XState）在数学上的定义**。

**UI 与业务的全部结构，最终都被压缩进这一个函数里。**

---

## 4.1 从数学定义看：Reducer 就是 δ（Transition Function）

在自动机理论（Automata Theory）中，一个状态机可以形式化为：

```code
M = (Σ, S, s₀, δ)
```

其中各部分含义为：

| 符号 | 名称                | React / Redux 对应 |
| -- | ----------------- | ---------------- |
| Σ  | 字母表（Alphabet）     | Action 类型集合      |
| S  | 状态空间（State Space） | Redux State      |
| s₀ | 初始状态              | initialState     |
| δ  | 转移函数（Transition）  | reducer          |

我们真正关心的是 **δ**：

```code
δ : S × Σ → S
```

而 Redux 中的 reducer 完全等价于：

```code
reducer : State × Action → State
```

**这是严格的数学同构，而不是类比。**

---

## 4.2 Reducer 为什么必须纯？因为 δ 必须纯

有限状态机的转移函数 **δ** 有三个核心数学性质。

### 1）确定性（Determinism）

```code
δ(s, a) = s'
```

必须满足：

* **相同输入 → 相同输出**
* **不依赖外部随机性**
* **不产生副作用**

否则，M 就不再是一个合法的确定性状态机。

→ **Redux 要求 reducer 是 pure，本质原因是：δ 必须是纯函数。**

---

### 2）封闭性（Closure）

输出必须仍然属于状态空间：

```code
δ(s, a) ∈ S
```

这意味着 reducer 的返回值：

* **必须是合法 state**
* **不能出现“部分更新失败”**
* **不能“丢字段”**
* **不能隐式改变结构**

你之前讨厌“混乱业务代码”，
数学原因正是：**它破坏了状态空间的封闭性。**

---

### 3）无副作用（Side-effect free）

转移函数 **δ** 不允许改变外部世界（World）：

* 不能发请求
* 不能读写浏览器 API
* 不能产生随机数
* 不能 `print / console.log`
* 不能 `setTimeout / setInterval`

因为 **δ 的数学目的只有一个**：

> **描述状态空间的演化，而不是描述世界。**

世界的复杂性，
必须被推迟到 **Effect Boundary（副作用边界）** 之外处理。

这也是为什么：

* reducer 不做 IO
* React 把副作用放进 `useEffect`
* Redux 把副作用交给 middleware / saga / thunk

---

## 4.3 Action = 字母表（Alphabet）= Sum Type

在自动机理论中，输入字母表 **Σ** 是一个**有限且可枚举的集合**：

```code
Σ = { a₁, a₂, … , aₙ }
```

这在 Redux / 前端中，对应的正是 **Action 的类型集合**。

例如在工程中：

```ts
type Action =
  | { type: 'ADD_ITEM'; payload: Item }
  | { type: 'REMOVE_ITEM'; id: string }
  | { type: 'SET_LOADING'; value: boolean }
```

数学上可以写成：

```code
Action = A₁ + A₂ + … + Aₙ   （Sum Type）
```

### Action 作为 Alphabet 的意义

* **离散事件系统（Discrete Event System）**，而非连续系统
* 输入是可穷举、可验证的
* reducer 可以做到穷尽匹配（exhaustive check）

这正是为什么：

* `switch(action.type)` 必须穷尽
* TypeScript 能帮你检查“漏掉的 case”
* 系统行为是**可控、可回放、可证明**的

---

## 4.4 Reducer = 状态空间上的一个「演化规则集」

状态空间 **S** 本身是一个多维笛卡尔积：

```code
S = U × T × Items × Filters × Bool × …
```

Reducer 做的事情不是“改字段”，而是：

> **在状态空间 S 中，定义哪些移动是合法的。**

也就是从当前点移动到下一个点：

```code
s_t → s_{t+1}
```

例如：

```code
loading = true  →  loading = false  ∧  items = [...]
```

你可以把 reducer 看成：

> **在一个高维空间里，
> 规定点如何被允许移动的「规则集合」。**

这正是：

* 动态规划里的状态转移
* 自动机里的状态跃迁
* 业务系统里的流程推进

它们在数学上是**同一个问题**。

---



## 4.5 Reducer 的数学结构 = Piecewise Function（分段函数）

Reducer 在工程中的常见写法是：

```ts
function reducer(state, action) {
  switch (action.type) {
    case 'A': return fA(state, action)
    case 'B': return fB(state, action)
    case 'C': return fC(state, action)
  }
}
```

在数学上，这正是一个**分段定义函数（Piecewise Function）**：

```code
reducer(s, a) =
  fA(s)   if a = A
  fB(s)   if a = B
  fC(s)   if a = C
```

### 这不是实现习惯，而是数学必然

原因在于：

* **Action 本身是 Sum Type**
* Sum Type 的消解（elimination）
* 在数学上必然表现为 **分段函数**

也就是说：

> **switch / pattern matching 不是“丑”，
> 而是 Sum Type 在函数层面的自然展开。**

### React / Redux / Vue / Svelte 都在做同一件事

它们本质上都是：

> **把一个复杂系统，
> 切分成有限、互斥、可验证的分支规则。**

这正是：

* 系统可控的原因
* 行为可推理的原因
* 状态机不会“失控”的原因

---

## 4.6 combineReducers = 笛卡尔积状态空间的分解

如果整体状态空间可以表示为多个子空间的笛卡尔积：

```code
State = S₁ × S₂ × S₃
```

那么整体的状态转移函数可以被分解为：

```code
δ = δ₁ × δ₂ × δ₃
```

也就是在工程中看到的：

```code
(s₁, s₂, s₃) → (δ₁(s₁), δ₂(s₂), δ₃(s₃))
```

### combineReducers 在数学上做了什么？

* 每个 reducer **只在自己的子状态空间内移动**
* 各子空间之间 **相互独立、互不干扰**
* 最终组合成一个更高维的整体状态空间

这在数学上叫做：

> **笛卡尔积空间上的分块转移（Block Transition）**

### 工程直觉翻译

* reducer 拆分 ≠ 业务割裂
* 而是对**高维状态空间的坐标轴拆解**
* 可组合性来自于：Product Type 的天然结构

这也是为什么：

* Redux 能无限 scale
* reducer 可以安全拆分、重组
* 系统复杂度不会指数爆炸

---

## 4.7 为什么「业务逻辑」天然适合写成状态机？

因为几乎所有业务系统，都可以被规范化为三要素：

* **状态（S）**：系统当前所处的情形
* **事件（Σ）**：外部或内部发生的离散事件
* **转移规则（δ）**：在事件作用下如何从一个状态移动到下一个状态

典型例子包括：

```text
- 购物车流程
- 用户登录 / 鉴权
- 表单校验
- 权限系统
- 向导（Wizard Step）
- Modal / DropDown / Tab
```

你之所以能**直觉性地理解这些业务**，
是因为你的大脑早已把它们识别为：

> **有限状态机模型。**

---

## 4.8 Reducer =「离散数学化的业务逻辑」

Reducer 帮开发者**主动消除**：

* 时间（time）
* 异步顺序（async order）
* 世界状态（World）
* 并发竞争（race condition）

Reducer 只关心两件事：

> 1. 状态空间的数学结构
> 2. 事件发生后的合法空间移动

因此 reducer 天然适合用于：

* 测试（testing）
* 回放（replay）
* 还原（undo / redo）
* 调试（debug）
* 可视化（visualization）
* 正确性证明（proof）

这正是它成为**现代前端核心抽象**的根本原因。

---

## 4.9 reducer & DP 的相似之处？

你在DP算法训练中最擅长的正是：

1. **定义状态空间**
2. **抽象状态转移规律**
3. **保持状态结构的稳定与不变量**

这与动态规划（DP）的数学任务完全一致：

| DP               | Reducer                       |
| ---------------- | ----------------------------- |
| 状态 (i, j, mask…) | state (user, items, loading…) |
| 转移方程             | reducer 的分支逻辑                 |
| 子问题结构            | 行为 / 事件序列                     |
| 优化与约束            | 不变量、拆分、组合                     |

所以你在工程里做的事情，本质上就是：

> **把业务逻辑离散化、数学化、状态机化。**

这是一种**可迁移、可证明、可扩展的核心能力**。

---

## ⭐ 第 4 卷小结

* reducer 不是实现细节，而是**状态机定义**
* `State × Action → State` 是业务的最小完备表达
* UI 只是状态轨迹的投影结果

> **当你真正理解 reducer，
> 你理解的不是 Redux，
> 而是整个系统的“演化法则”。**
