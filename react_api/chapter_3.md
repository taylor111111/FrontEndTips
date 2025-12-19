

# Context：环境注入，而不是状态管理

在 React 的 API 体系中，Context 是一个被**高频使用、也高频被误用**的工具。

很多问题并不来自“不会用 Context”，而是来自：

> **把 Context 当成了跨组件状态管理工具，甚至是隐式的全局变量。**

本文从 **设计目的 → 何时应当使用 → 何时最好别用 → 明确禁止的使用边界** 四个层面，重新界定 Context 在工程中的真实位置。

---

## 一、Context 的设计目的

### 1. 一句话定义（工程语义版）

> **Context 用来向“一整个子树”广播一份共享的、语义稳定的运行环境（environment）。**

关键词非常重要：

- 子树（subtree）
- 共享（shared）
- 广播（broadcast）
- 环境（environment）
- 语义稳定（semantically stable）

Context 不是“跨组件通信工具”，
而是**环境注入机制**。

---

### 2. 官方设计 Context 的真实动机

Context 的诞生，是为了解决 **prop drilling 的结构性问题**，
而不是为了解决业务状态管理。

React 官方给出的典型语义包括：

- Theme（主题）
- Locale / i18n（语言环境）
- Direction（ltr / rtl）
- Design System Tokens
- Router / Form / Auth 等框架级上下文

这些数据的共同特征是：

> **它们描述的是“组件运行在什么环境中”，
> 而不是“组件当前处于什么业务状态”。**

---

### 3. 一个必须牢记的事实

Context 在 React 内部的行为是：

> **Context value 变化 → 整个消费该 Context 的子树重新 render**

React 并不知道你“只用了一部分字段”，
它只知道：

> **环境发生了变化。**

这一点，决定了 Context 的工程边界。

---

## 二、什么时候应当使用 Context（正当场景）

以下场景中使用 Context，
在工程上是**合理、清晰且符合设计初衷的**。

---

### 1. 全局 / 半全局的“环境配置”

#### 典型例子

- Theme（暗色 / 亮色）
- 语言 / 时区
- Design Tokens
- Feature Flags（低频变化）

```tsx
<ThemeProvider value={theme}>
  <App />
</ThemeProvider>
```

这些数据的特点是：

- 几乎所有子组件都需要
- 语义是“运行环境”，不是“业务状态”
- 更新频率低
- 一次更新意味着整体环境切换

这是 Context 的**本职工作**。

---

### 2. 框架级 / 系统级能力注入

例如：

- Router（`useLocation` / `useParams`）
- Form（`react-hook-form` / Formik）
- Auth Context（当前用户 + 权限）

这些 Context 通常具备：

- 数据结构稳定
- 变化有明确、可解释的语义
- 子组件天然依赖该环境

---

### 3. 明确受限的“作用域共享状态”（Scope-bound）

例如：

- 一个表单内部的字段上下文
- Tabs / Accordion 内部的当前 index
- Modal 树内的交互上下文

```tsx
<TabsProvider>
  <Tab />
  <Tab />
</TabsProvider>
```

关键点在于：

> **Context 的作用范围是刻意被限制的，而不是默认全局的。**

---

## 三、什么时候最好别用 Context（危险信号）

下面这些情况并非立刻错误，
但通常意味着：

> **组件或状态建模方式已经开始偏离 React 的推荐模型。**

---

### 1. 把 Context 当“轻量版全局状态管理”

```ts
<UserContext.Provider value={user}>
```

然后在整个应用中到处使用。

问题在于：

- `user` 往往变化频繁
- Context 更新会触发大范围 re-render
- 性能与可预测性迅速下降

> **Context 不是 Redux / Zustand。**

---

### 2. 高频变化的数据放进 Context

例如：

- input value
- mouse position
- scroll offset
- animation progress

Context 默认没有“选择性订阅”能力，
因此这是一个**明显的性能雷区**。

---

### 3. Context 里塞“杂货铺式数据”

```ts
value={{
  user,
  theme,
  locale,
  setUser,
  updateTheme,
  logout,
  ...
}}
```

这通常意味着：

- 职责不清
- 更新来源混乱
- 使用方依赖关系模糊

> **Context 应当保持语义单一。**

---

## 四、什么时候禁止使用 Context（工程红线）

下面这些场景，可以明确认为是**工程错误**。

---

### 1. 用 Context 代替 props 传参（只是因为懒）

如果只是：

- 层级不深
- 数据只给一两个组件用
- 不存在结构性 prop drilling

那么使用 Context 只是：

> **为了省几行代码，引入隐式依赖。**

这是工程上的负资产。

---

### 2. 用 Context 作为“跨模块通信机制”

如果你发现：

- 不相关模块通过 Context 共享数据
- Context 成为事实上的“共享内存”
- 修改 Context value 的影响范围难以预测

那么它等同于：

> **把隐式全局状态引入 React。**

---

### 3. 用 Context 隐藏数据流方向

这是最危险的一类滥用。

当 Context 被用来：

- 绕开 props 的显式数据流
- 让数据“从不知道哪里来”
- 增加调试与理解成本

它已经违背了 React 的核心设计哲学。

---

## 五、一个通用的判断法则

在考虑是否使用 Context 时，可以问自己一句：

> **“这个数据，是‘环境’，还是‘业务状态’？”**

- 如果是 **环境** → Context 合理
- 如果是 **业务状态** → 优先使用 state / store
- 如果你说不清楚 → ⚠️ 大概率不该用 Context

---

## 六、最终总结

> **Context 是用来描述“你在哪儿”，
> 而不是“你现在在干什么”。**

因此，它的工程定位应当是：

- 环境注入工具
- 作用域清晰
- 更新频率低
- 语义高度稳定

当 Context 被用来：

- 承载业务逻辑
- 高频变化
- 替代明确的数据流

它就会从一个设计良好的 API，
退化为：

> **隐式的全局状态。**
