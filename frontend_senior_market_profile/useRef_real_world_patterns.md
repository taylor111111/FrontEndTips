

# useRef · 真实工程使用模式（Real World Patterns）

> 这不是一篇 useRef API 教程。
> 这篇文章只回答一个工程问题：
>
> **useRef 在真实前端工程中，究竟被用来解决什么问题？**
>
> 如果一个 useRef 的使用理由无法用“工程边界”解释，那它大概率是不必要的。

---

## useRef 的工程本质（一句话）

> **useRef 是一个：跨 render 持久存在，但不参与渲染与调度的可变容器。**

它解决的从来不是“数据展示”，而是：

- 生命周期绑定
- 时间线对齐
- 命令式接口

---

## Pattern 1：DOM ref（命令式边界）

### 解决的问题

React 是声明式系统，但 DOM 是命令式系统。

在以下场景中，你**必须**越过声明式边界：

- focus / blur
- scrollTo
- 获取 DOM 尺寸

### 工程示例

```tsx
function Input() {
  const inputRef = useRef<HTMLInputElement>(null);

  const focus = () => {
    inputRef.current?.focus();
  };

  return <input ref={inputRef} />;
}
```

### 工程判断

- 不触发 render
- 不参与状态
- 只作为命令式出口

> **这是 useRef 最正当、最稳定的用途。**

---

## Pattern 2：定时器 ref（生命周期绑定）

### 真实工程问题

- setTimeout / setInterval 返回的 id 需要被保存
- 但这个 id 的变化**不应该触发 render**

### 错误做法（高风险）

```tsx
const [timer, setTimer] = useState<number | null>(null);
```

问题：
- 无意义 render
- 时间线混乱
- 容易泄漏

### 正确做法

```tsx
function Comp() {
  const timerRef = useRef<number | null>(null);

  useEffect(() => {
    timerRef.current = window.setInterval(() => {
      console.log('tick');
    }, 1000);

    return () => {
      if (timerRef.current !== null) {
        clearInterval(timerRef.current);
      }
    };
  }, []);
}
```

### 工程判断

- timer 生命周期与组件绑定
- 不进入 render 体系
- 不制造额外状态

> **这是 useRef 在工程中非常“值钱”的用途。**

---

## Pattern 3：最新值 ref（解决闭包 + 时间线问题）

### 真实工程问题

以下场景极其常见：

- setTimeout / setInterval
- Promise.then
- event listener

它们运行在**另一条时间线**中。

### 典型问题

```tsx
useEffect(() => {
  setInterval(() => {
    console.log(count); // 可能是旧值
  }, 1000);
}, []);
```

### useRef 的工程解法

```tsx
const countRef = useRef(count);

useEffect(() => {
  countRef.current = count;
});

useEffect(() => {
  const id = setInterval(() => {
    console.log(countRef.current);
  }, 1000);

  return () => clearInterval(id);
}, []);
```

### 工程本质

- React 状态：用于渲染
- ref：用于跨时间线读取最新值

> **useRef 在这里是“时间线对齐器”，而不是状态替代品。**

---

## 反模式：哪些 useRef 用法是危险的

### ❌ 用 ref 模拟 state

```tsx
const countRef = useRef(0);
countRef.current++;
```

问题：
- 不可预测
- 不可追踪
- 与 React 模型冲突

### ❌ 把业务数据放进 ref

- 无法触发 UI 更新
- 无法被调试工具感知

---

---

## 工程讨论：useRef 是否会在每次 render 时重新创建？

这是一个在工程上**非常关键**、但经常被模糊处理的问题。

### 结论先行

```ts
const timerRef = useRef<number | null>(null);
```

- `useRef` 创建的 **ref 对象只会在组件首次 mount 时创建一次**
- 后续每一次 render：
  - React 都会返回**同一个 ref 对象**
  - 不会重新分配、不改变身份

也就是说：

> **useRef 不属于 render 时间线，而属于组件生命周期时间线。**

---

### 对比：为什么普通变量不行？

```ts
function Comp() {
  let timerId: number | null = null;
}
```

React function component 的本质是：

> **render = 函数重新执行一次**

因此：

- 每次 render，`timerId` 都会被重新声明
- 上一次 render 中的 `timerId` 会直接丢失
- 不同时间线（effect / cleanup / handler）会引用不同的变量实例

这就是典型的 **render 级不稳定变量问题**。

---

### 再对比：useRef 的行为模型

```ts
function Comp() {
  const timerRef = useRef<number | null>(null);
}
```

这里发生的事情是：

- 首次 mount：
  - React 创建一个 ref 对象 `{ current: null }`
- 后续 render：
  - React 返回同一个 ref 对象引用
  - 只是重新执行函数体

因此：

- `timerRef` 的身份是稳定的
- `timerRef.current` 是可变的
- 修改 `current` **不会触发 render**

---

### 工程层面的真正区别

| 保存方式 | 是否跨 render 稳定 | 是否触发 render | 适合做什么 |
|--------|------------------|----------------|------------|
| let / const | ❌ 否 | ❌ | 临时计算 |
| useState | ✅ 是 | ✅ | UI 状态 |
| useRef | ✅ 是 | ❌ | 生命周期 / 时间线绑定 |

---

### 一个工程判断句（建议记住）

> **如果一个值需要在多次 render 之间保持“身份一致”，  
> 但它的变化不应该影响 UI，  
> 那它就不属于 render，而应该放进 useRef。**

## 一个工程判断公式

你可以用下面的问题快速判断是否该用 useRef：

> **这个值的变化：**
> 1️⃣ 是否需要触发 render？  
> 2️⃣ 是否属于 React 的状态系统？

- 如果两个答案都是 ❌ → useRef  
- 只要有一个是 ✅ → state  

---

---

### 工程补充讨论：useRef 能不能“修改”？边界在哪里？

一个常见的误解是：

> “useRef 不能被修改”

这是不准确的。

**更精确的工程结论是：**

> **useRef 允许被写入，但不允许承载「业务状态的演化」。**

#### ✅ 合法的 useRef 写入（工程上必须）

以下写入是安全且必要的：

```ts
// 1. 保存命令式资源句柄
timerRef.current = setInterval(...);

// 2. 同步最新值（只读用途）
countRef.current = count;

// 3. 操作 DOM / imperative API
inputRef.current?.focus();
```

这些写入的共同特征是：

- 不推动业务逻辑
- 不作为状态来源
- 只是同步“外部事实”或“运行时资源”

---

#### ❌ 危险的 useRef 写入（反模式）

```ts
countRef.current++;
```

问题不在于“写了 ref”，而在于：

- ref 被当成了 state
- 状态变化绕过了 React
- render / effect / devtools 全部失去感知

这类代码通常会导致：

- 隐蔽 bug
- 时间线错位
- 难以调试和维护

---

#### 工程边界总结（一句话）

> **useRef 不是 state 的替代品。  
> 它只能保存「不参与渲染的可变事实」，  
> 不能承载「业务状态的推进」。**

一旦你发现自己在 ref 上“推进业务”，  
说明这个逻辑已经越过了 React 的工程边界。

## 总结

- useRef 不是“高级技巧”，而是**边界工具**
- 它只解决三类问题：
  - 命令式边界
  - 生命周期绑定
  - 时间线对齐

> **当你用 useRef 时，你不是在写 React，而是在给工程划边界。**
