# forwardRef / useImperativeHandle：受控地暴露命令式能力，而不是打破封装

在 React 的 API 体系中，`forwardRef` 与 `useImperativeHandle` 是**最容易被误解、也最需要被克制使用**的一组工具。

它们的存在，并不是为了让组件“更灵活”，而是为了在一个**严格的声明式模型**中，
**有限地、可控地**对接命令式世界。

本文从 **设计目的 → 建议使用的场景 → 最好别用的场景 → 明确禁止的使用边界** 四个层面，
重新把这组 API 放回它们真实、危险、但必要的工程位置。

---

## 一、forwardRef / useImperativeHandle 的设计目的

### 1. 一句话定义（工程语义版）

> **`forwardRef + useImperativeHandle` 用来在组件边界上，
> 有意识地、受控地暴露一组“命令式能力接口”。**

这句话里每一个词都不可省略：

- **组件边界**：它发生在组件封装的边缘
- **命令式（imperative）**：通过调用动作，而不是声明状态
- **能力接口（capability interface）**：暴露“能做什么”，而不是“内部是什么”
- **受控（controlled exposure）**：刻意限制暴露内容，而不是全部开放

它们不是用来“拿到子组件内部状态”的，
而是用来**声明：这个组件允许外界命令它做什么**。

---

### 2. 两个 API 各自的职责（必须区分）

#### `forwardRef`

> **解决的是：ref 能否穿过组件边界的问题**

```ts
const Input = forwardRef((props, ref) => {
  return <input ref={ref} />
})
```

`forwardRef` 本身**不包含任何业务语义**，
它只是一个**通道**。

---

#### `useImperativeHandle`

> **解决的是：你到底向外暴露什么能力的问题**

```ts
useImperativeHandle(ref, () => ({
  focus() {},
  reset() {}
}))
```

它的核心作用并不是“增强 ref”，
而是：

> **限制 ref 的能力边界，而不是放大它。**

这是这组 API 最容易被误解、但最重要的设计点。

---

### 3. 官方真正的设计动机

React 设计这组 API，是为了一个非常明确的工程需求：

> **在声明式 UI 框架中，
> 仍然需要与“命令式世界”进行少量、受控的交互。**

典型对象包括：

- DOM（focus / scroll / measure）
- 第三方 imperative library
- Canvas / WebGL / Media API

而**不是**为了跨组件控制业务逻辑。

---

## 二、什么时候建议使用（工程上成立的场景）

下面这些场景，使用 `forwardRef / useImperativeHandle` 是**合理、干净、且专业的**。

---

### 1. 封装具备明确“命令语义”的 UI 组件

#### 典型例子

- Input / Textarea
- Modal / Drawer
- Tooltip / Popover
- Media Player

```ts
useImperativeHandle(ref, () => ({
  focus() {
    inputRef.current?.focus()
  },
  clear() {
    setValue('')
  }
}))
```

这些暴露出来的能力具有共同特征：

- 与 UI 行为直接相关
- 不依赖业务状态
- 调用是“动作”，而不是“状态同步”

这是 `useImperativeHandle` 的**本职工作**。

---

### 2. 作为命令式第三方库的桥接层

例如：

- Chart / Map / Editor 实例
- Canvas / WebGL
- Video / Audio 控制器

你在做的事情本质是：

> **把第三方 imperative API，
> 封装成一个 React 组件对外暴露的最小能力集。**

这是一个非常正当的使用场景。

---

### 3. 需要绕开 render 流程的一次性动作

例如：

- 手动触发 focus
- 触发动画
- 测量尺寸

这些行为：

- 不应进入 state
- 不应触发 re-render
- 本质上是“瞬时动作”

---

## 三、什么时候最好别用（强烈危险信号）

下面这些情况并非立刻错误，
但几乎总是意味着：

> **组件设计正在开始腐化。**

---

### 1. 用 ref 调用子组件的业务方法

```ts
childRef.current.submit()
childRef.current.saveDraft()
```

这意味着：

- 父组件正在驱动子组件的业务流程
- 子组件失去封装性
- 数据流方向被打破

这是**组件解耦失败**的典型信号。

---

### 2. 用 ref 读取或修改子组件内部状态

```ts
childRef.current.value = xxx
```

这相当于：

> **把组件当成一个可变对象，而不是声明式单元。**

这会让：

- 调试极其困难
- 状态一致性崩溃

---

### 3. 把 useImperativeHandle 当成“更高级的 props / Context”

如果你发现：

- 用 ref 传递数据
- 用 ref 同步状态
- 用 ref 替代 props

那么几乎可以确定：

> **你在逃避建模，而不是在封装能力。**

---

## 四、什么时候禁止使用（工程红线）

下面这些场景，可以非常明确地认为是**工程错误**。

---

### 1. 用它们做跨组件业务通信

```ts
parent -> childRef -> child internal logic
```

这会导致：

- 隐式依赖
- 破坏封装
- 难以测试
- 难以重构

这是 React 中**最危险的反模式之一**。

---

### 2. 在纯展示组件中暴露 imperative API

如果一个组件：

- 只负责展示 props
- 不具备命令语义
- 不应产生副作用

那它**不应当拥有 ref 接口**。

否则就是：

> **人为制造命令式入口。**

---

### 3. 把 forwardRef 当作“设计逃生口”

```ts
// 不知道怎么设计 → 先 forwardRef
```

这是一个非常明确的红线信号。

因为这意味着：

> **你在用打破封装，来替代架构设计。**

---

## 五、一个通用且可靠的判断法则

在考虑是否使用这组 API 时，可以问自己一句：

> **“我向外暴露的，是一个‘动作能力’，
> 还是在让外部操纵我的内部状态？”**

- 如果是 **动作能力** → ✅ 可以考虑
- 如果是 **内部状态 / 业务流程** → ❌ 禁止
- 如果你说不清楚 → ⚠️ 默认不用

---

## 六、最终总结

> **forwardRef / useImperativeHandle 的存在，
> 是为了“有限地违反声明式模型”，
> 而不是为了“绕开它”。**

它们应当被用作：

- 命令式世界的桥接层
- UI 行为的受控出口
- 最小、稳定、可预测的能力接口

一旦它们被用来：

- 承载业务逻辑
- 作为通信机制
- 破坏数据流方向

它们就会从**必要的边界工具**，
退化为**架构腐蚀的起点**。
