# 《useEffect 必须做的 5 类事情》（强化 version）



> **Effect = UI（当前快照）→ 外部世界（真实设备 / 浏览器 / 网络）的同步操作**

简单讲：**只要是 React 无法自动管理的事，就放在 Effect 里。**

---

## 🟢 ① 订阅 / 取消订阅类（最典型且最正确的用途）

### ⭐ 必须使用 Effect 的理由

- 订阅是一种外部世界的长期关系，而每次渲染对应不同快照
- 所以你必须告诉 React：
    - 当前这版快照想订阅什么
    - 下次渲染来之前，撤销上一次快照的订阅

### 🍀 场景：浏览器事件（resize）

### 正确写法（最经典最清晰）

```jsx
useEffect(() => {
  function handleResize() {
    setWidth(window.innerWidth);
  }

  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []); // 不依赖任何 UI 变量，只挂一次
```

### 工程师能理解的理由

- “组件挂载时监听，到卸载时取消监听”——他们能理解

### React 的心智模型版本（给你）

- 每次快照都需要告诉 React：我需要一个 resize listener
- React 必须先清理旧 listener，再挂新 listener
- `deps = []` 表示订阅与渲染时的 UI 变量无关 → 可以复用

---

## 🟢 ② 计时器 / Interval / Timeout

### ⭐ 必须使用 Effect 的理由

- 计时器是独立运行的外部时间线，不属于 React 控制

### 正确写法

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + 1); // 永远用函数式更新
  }, 1000);

  return () => clearInterval(id);
}, []); // 不依赖 UI 状态
```

### 为什么（工程师理解版）

- useEffect 定时器 = 启动一个后台线程
- cleanup = 停止线程
- 保证计时器不会越跑越多、不会吃闭包旧值

---

## 🟢 ③ 网络请求（fetch / axios）

### 重点（工程师务必理解）

- **Effect 不是负责发起逻辑判断的地方**
- Effect 只负责：**“结果状态变化 → 同步外部世界”**
- 事件驱动逻辑应放在 handler，不要塞给 Effect

---

## 🍀 场景一：进入页面自动加载数据（典型业务场景）

### 正确写法

```jsx
useEffect(() => {
  fetchUser();
}, [fetchUser]); // fetchUser 是 useCallback 包装的“动作”
```

### 为什么这样写：

- 事件（fetchUser 动作）→ `useCallback`
- Effect 只订阅一个稳定的“动作”
- 清晰、可控、不会重复执行

---

## 🍀 场景二：条件变化触发请求（例如 id 变化）

### 正确写法

```jsx
useEffect(() => {
  getDetail(id);
}, [id]);
```

### 工程师会认的解释

- “id 变了，当然要重新请求”
- 他们完全理解这个！

### 你理解的深度解释

- 当前快照对应的 id 变化
- Effect 用 id 做 diff → 需要同步外部世界（请求新数据）

---

## 🟩 ④ DOM 操作类（非 React 管控）

例如：`focus`、`scroll`、第三方库、`canvas`、`map`、`d3` 等

### 🍀 场景一：focus input

```jsx
useEffect(() => {
  inputRef.current?.focus();
}, []);
```

### 🍀 场景二：scroll 到某个位置

```jsx
useEffect(() => {
  divRef.current.scrollTo(0, 0);
}, [someTrigger]);
```

### 为什么必须用 Effect？

- React 的渲染只是虚拟 DOM → 真实 DOM 在 commit 阶段才出现
- Effect 是 commit 后的执行点 → 可安全操作真实 DOM
- 不在 Effect 中操作 DOM = 可能操作不存在的 DOM

---

## 🟩 ⑤ localStorage / sessionStorage / history 同步类

### 🍀 场景：主题色同步到 localStorage

```jsx
useEffect(() => {
  localStorage.setItem('theme', theme);
}, [theme]);
```

### 工程师理解

- “theme 变了，就写入 localStorage” → 完全理解

### React 心智模型

- 当前快照的 theme = 真实世界的 theme
- 这是一个稳定的同步关系 → 必须在 Effect 中表达

---

## 🟥 用一句话总结工程师应该记住的事

> **只要在操作“浏览器 / 外部系统”，就必须用 Effect。**  
> **副作用 = 与 React 之外的世界建立联系。**

---


