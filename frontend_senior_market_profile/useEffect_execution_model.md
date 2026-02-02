# useEffect Execution Model

> 本文的目标不是教你“怎么用 useEffect”，  
> 而是解决两个问题：
>
> 1. **工程上：useEffect 到底在什么时候执行？**
> 2. **表达上：面试时，应该如何把这个问题说得“刚刚好”？**

本文分为两个版本：
- **工程版本**：给自己用，模型完整、可推演
- **面试官友好版本**：给市场用，安全、不发散

---

## 一、工程版本（给自己用｜完整时间线）

这是事实层面的执行模型，用来解释所有反直觉现象。

### 1. React + 浏览器的真实时间线（简化但准确）

```
[ React Render ]
   ↓
[ Virtual DOM diff ]
   ↓
[ Commit: 更新真实 DOM ]
   ↓
[ Browser Layout ]
   ↓
[ Browser Paint ]

```

**关键澄清（非常重要）：**

- React 的 **commit 阶段** 负责把变更同步到真实 DOM
- **useLayoutEffect** 运行在 commit 之后、浏览器 paint 之前  
  （此时 DOM 已更新，浏览器即将或已经计算布局）
- **useEffect 并不保证发生在 layout 之前或之后**  
  它只保证：**不会阻塞浏览器绘制**

因此，更准确的理解是：
- useLayoutEffect ≈ 与浏览器布局强相关、同步
- useEffect ≈ 在浏览器完成一次绘制后，异步执行副作用

---

### 2. 各阶段能做什么

#### ① Render 阶段（React）
- 执行函数组件
- 调用 hooks
- 计算 Virtual DOM
- ❌ 不能有副作用
- ❌ 不能读 DOM
- ❌ 不能 setState（否则死循环）

这是一个**纯计算阶段**。

---

#### ② Commit 阶段（React）
- 将 Virtual DOM 的差异应用到**真实 DOM**
- DOM 节点已创建 / 更新 / 删除

此时：
- `document.querySelector` 可以拿到节点
- ❌ 不保证尺寸、位置已正确

---

#### ③ Browser Layout（浏览器）
- 计算元素尺寸
- 计算元素位置
- 计算 viewport 信息（如 `window.innerHeight`）

这是**尺寸真正确定**的阶段。

---

#### ④ Browser Paint（浏览器）
- 将像素绘制到屏幕
- 用户“看到页面”

---

#### ⑤ useLayoutEffect（React）
- DOM 已更新
- Layout 已完成
- Paint 尚未发生
- ❗ 会阻塞 paint

适合：
- 同步读取 / 写入布局
- 修正视觉抖动

风险：
- 阻塞渲染
- 滥用会导致性能问题

---

#### ⑥ useEffect（React）
- DOM 已更新
- ❌ 不保证 Layout 已完成
- ❌ 不保证 Paint 已完成
- 不阻塞页面渲染

适合：
- 数据请求
- 事件订阅
- 日志
- 非布局相关副作用

---

### 3. 核心工程结论

> **useEffect 保证 DOM 已提交，不保证布局完成**  
> **useLayoutEffect 保证 DOM + 布局完成，但会阻塞渲染**

这句话可以解释：
- 为什么 useEffect 里有时拿不到正确的高度
- 为什么 useLayoutEffect 能拿到，但要谨慎使用

---

## 二、面试官友好版本（市场安全｜只说到这里）

下面是**面试中最多能说到的层级**。

---

### 面试官问：useEffect 什么时候执行？

**回答：**

> useEffect 会在组件 render 完成，并且 DOM 更新之后执行，  
> 用来处理副作用。

（停）

---

### 面试官问：为什么有时候拿不到高度？

**回答：**

> 因为 useEffect 执行时，DOM 已经更新，  
> 但不保证浏览器已经完成布局，  
> 如果需要同步读取布局信息，一般会用 useLayoutEffect。

（停）

---

### 面试官追问：useLayoutEffect 有什么区别？

**回答（上限）：**

> useLayoutEffect 是在 DOM 更新之后、浏览器绘制之前执行，  
> 可以安全读取布局信息，但会阻塞渲染，所以一般谨慎使用。

（必须停）

---

## 三、表达分层原则（非常重要）

- **脑子里**：工程版（完整时间线）
- **嘴上说**：面试版（安全、简洁、不发散）

这不是妥协，而是高级工程师的表达控制能力。

---

## 四、一句话总结

> 工程上要比市场更清楚，  
> 表达上只说到对方能安全接住的地方。

这篇文档的存在，就是为了同时满足这两点。
