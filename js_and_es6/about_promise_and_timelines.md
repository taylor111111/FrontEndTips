

# Promise 的问题，从来不是语义，而是时间线

> 这不是一篇 Promise 教程。
> 如果你想找 `then / catch / finally` 的用法，这篇文章不适合你。
>
> 这篇文章只回答一个工程问题：
> **Promise 运行在哪条时间线上？这条时间线是否被明确控制？**

---

## 一、为什么「Promise 语义」不是工程难点

Promise 的语义其实非常简单：

- 创建
- resolve / reject
- then / catch / finally

绝大多数工程事故，并不是因为：

- 不知道 Promise 是微任务
- 不清楚 Promise A+ 规范
- 忘记 then 的参数顺序

而是因为：

> **Promise 被放进了一条失控的时间线里。**

当时间线失控时：

- 再正确的语义也会被破坏
- 再熟练的 API 也会制造 bug

---

## 二、什么是「时间线」

在前端工程中，可以把下面这些都理解为**时间线**：

- React render / effect 生命周期
- 用户事件（onClick / onChange）
- Promise
- setTimeout / setInterval
- 浏览器事件

**时间线的本质不是异步，而是：**

> 谁决定它什么时候开始、什么时候结束、是否可以被忽略。

---

## 三、哪些 Promise 使用方式是「低风险」的

### 1️⃣ Promise + useEffect（时间线被 React 控制）

```js
useEffect(() => {
  fetchData().then(setData);
}, []);
```

特点：

- 触发频率明确（mount 一次）
- 生命周期明确（unmount cleanup）
- React 拥有主导权

**工程判断：**
> Promise 被托管在 React 时间线中，是安全的。

---

### 2️⃣ Promise + 用户事件（时间线由用户控制）

```js
const onSubmit = async () => {
  await submit();
};
```

特点：

- 触发源明确
- 调用频率自然受限
- 不需要复杂取消语义

**工程判断：**
> 用户事件本身就是一个天然调度器。

---

## 四、哪些 Promise 使用方式是「高风险」的

### 🚨 Promise + setTimeout / setInterval

```js
setInterval(() => {
  fetchData().then(update);
}, 1000);
```

问题不在 Promise，而在：

- setInterval 是**独立时间线**
- Promise 也是**独立时间线**
- 两条时间线叠加，没有统一控制者

可能出现：

- 竞态条件
- 状态覆盖
- 内存泄漏
- 组件卸载后仍在运行

**工程结论：**
> Promise 不应该直接由 setTimeout / setInterval 驱动。

---

### 🚨 forEach + async

```js
items.forEach(async item => {
  await save(item);
});
```

问题：

- forEach 不等待 Promise
- 无法捕获错误
- 顺序不可控

**工程结论：**
> forEach 不是异步控制结构。

---

## 五、如何降低 Promise 时间线管理复杂度

### 核心原则

> **不要管理 Promise，
> 而是把 Promise 放进一个已经被管理好的时间线。**

可选宿主包括：

- React 生命周期
- 用户事件
- 专用数据层（React Query / SWR）

---

## 六、什么时候「必须」管理复杂 Promise 时间线

绝大多数业务代码，**不需要**。

以下三类是例外：

### 1️⃣ 请求并发与失效管理

- 搜索联想
- 参数变化触发请求
- 后发请求覆盖先发请求

这是**数据一致性问题**，而不是 Promise 问题。

**正确答案：** React Query / SWR

---

### 2️⃣ 任务流程编排（偏基础设施）

- 文件上传
- 多步骤异步流程
- 重试 / 回滚

特点：

- 高复杂度
- 高维护成本
- 不适合普通业务组件

---

### 3️⃣ 跨时间线协作（极少数）

- WebSocket
- Worker
- 长连接

**工程结论：**
> 这类代码应该被隔离在基础设施层。

---

## 七、一个实用的工程判断公式

你可以用下面这句话快速判断：

> **如果你开始思考：
> “这个 Promise 要不要取消 / 忽略 / 去重 / 排序”，
> 那你就不该自己管理它。**

---

## 八、总结

- Promise 的复杂度来自时间线，而不是语义
- 能托管给 React / 用户事件，就不要自己管
- 一旦需要手写复杂 Promise 管理，说明这是工程问题，不是业务问题

> **Promise 是时间线工具，而不是业务抽象工具。**

当你开始用「时间线是否可控」来审视 Promise，
你已经站在了工程视角，而不是 API 视角。
