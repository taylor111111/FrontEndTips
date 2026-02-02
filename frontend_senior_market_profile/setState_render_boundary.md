

# setState / render 边界

## 核心结论（先给结论）

**render 阶段必须是纯计算，不能产生副作用。  
setState 是副作用，因此不能在 render 中调用。**

这是 React 的基本设计前提，不是语法限制，而是架构约束。

---

## render 在 React 中到底是什么

在 React 中，`render`（或函数组件本身）的职责只有一件事：

> **根据当前 props + state，计算 UI 描述（Virtual DOM）**

它的特点是：

- 可能被 **多次调用**
- 可能被 **中断**
- 可能被 **回滚**
- 在 React 18 并发模式下，甚至可能被 **丢弃**

因此 render 必须满足一个条件：

> **无论被调用多少次，都不会对外部世界造成影响**

这就是“纯函数”语义。

---

## setState 为什么不能出现在 render 中

### 1️⃣ setState 会触发新的更新

`setState` / `setCount` 的本质是：

- 向 React **提交一次更新请求**
- 触发下一轮 render

如果在 render 中调用：

```js
function Comp() {
  setCount(1)
  return <div />
}
```

逻辑就变成了：

```
render → setState → render → setState → ...
```

这会导致**无限循环**。

---

### 2️⃣ render 可能被调用，但结果并不会被提交

在并发渲染（React 18）中：

- render 只是“尝试计算 UI”
- 并不保证一定会 commit 到 DOM

如果在 render 中 setState，就意味着：

> **一次“未被采用的 UI 计算”，却产生了真实副作用**

这是 React 设计中明确禁止的。

---

### 3️⃣ React 无法推断你的 setState 是否安全

React 无法判断：

- 你这个 setState 是“只调用一次”
- 还是“条件触发”
- 还是“幂等的”

因此规则只能是：

> **render 中一律禁止产生副作用**

---

## 那 setState 应该写在哪里

### ✅ 合法、安全的位置

#### 1. 用户事件

```js
onClick={() => setCount(count + 1)}
```

特点：
- 由用户触发
- 调用次数可控
- 语义清晰

---

#### 2. useEffect / useLayoutEffect

```js
useEffect(() => {
  setCount(1)
}, [])
```

特点：
- 明确的副作用边界
- 执行时机由 React 控制
- 可通过 deps 精确控制

---


## 一个常见误解：render 只是“第一次”调用

错误认知：

> “我这个 setState 只会执行一次，应该没问题吧？”

现实是：

- render 不等于 mount
- render 可能被调用多次
- React 不保证 render 的执行次数

**只要写在 render 中，就是不安全的。**

---

## 面试安全版回答（30 秒）

> render 阶段在 React 中要求是纯计算，只负责根据 props 和 state 计算 UI，不能产生副作用。  
> setState 本身会触发更新，如果写在 render 里会造成无限循环，而且在并发渲染下 render 可能被多次调用但不一定 commit，因此 React 明确要求在 render 中禁止 setState。  
> 实际工程里，状态更新一般放在事件回调或 effect 里。

——到此为止。

---

## 工程判断总结

一句话记住即可：

> **render 是“算 UI”，  
> effect / event 才是“改世界”。**

只要遵守这条边界，  
这类问题在工程里基本不会再出现。
