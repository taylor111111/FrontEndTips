# 🔁 A Complete Guide to useEffect

> **一句话版定位**  
> useEffect 不是生命周期 API，而是 **“将某一次渲染（snapshot）的结果，同步到 React 之外世界”** 的机制。

这是一组围绕 Dan Abramov 的  
**《A Complete Guide to useEffect》**  
所整理的 **系统性读书笔记 + 工程实践总结**。

目标不是“记 API”，而是建立一套 **可解释 bug、可指导设计、可用于面试与团队共识** 的 useEffect 心智模型。

---

## 📌 本系列你将学到什么？

- 为什么 **Effect ≠ lifecycle**
- 什么是 **render snapshot**
- 为什么闭包问题、本质上是“时间维度错误”
- 依赖数组（deps）真正的语义是什么
- 什么情况下 **必须用 Effect**，什么情况下 **不该用**
- 如何在真实业务中，把 **事件 / 状态 / 副作用** 分清楚

---

## 🧠 核心心智模型（总览）

> **Effect = 同步描述，而不是执行脚本**

- 每一次 render 都是一张 **独立的时间快照**
- Effect 只“看到”它所属的那次快照
- deps 数组 = 告诉 React：**哪些输入变化了，需要重新同步**
- cleanup = 撤销 **上一个快照** 对外部世界的影响

---

## 📚 目录

### Chapter 1
### **useEffect 的本质与心智模型**

👉 [`chapter_1.md`](./chapter_1.md)

- Effect ≠ lifecycle
- Each Render Has Its Own Everything
- Snapshot / Cleanup / Synchronization
- 依赖数组是 Effect 的 diff key
- 不要对依赖说谎（Don’t Lie About Dependencies）

---

### Chapter 2
### **从 useEffect 提炼的工程实践规则**

👉 [`chapter_2.md`](./chapter_2.md)

- One Effect Per Event Source
- 事件驱动 vs 状态驱动
- Event-specific Effects
- 用 `key` 重建时间线
- 一组可直接写进 README 的 useEffect 规则清单

---

### Chapter 3
### **典型业务场景 & 必须使用 Effect 的情况**

👉 [`chapter_3.md`](./chapter_3.md)

- 页面加载 / 条件变化请求
- DOM 操作（focus / scroll / 第三方库）
- localStorage / sessionStorage / history 同步
- 订阅 / 取消订阅
- 计时器 / interval / timeout
- 网络请求中 Effect 的正确职责

---

## 🚦 何时“应该 / 不应该”使用 useEffect（速查）

**必须用 Effect：**

- 操作浏览器 / DOM
- 订阅外部事件
- 操作 localStorage / history
- 启动计时器
- 与 React 无法控制的系统交互

**不该用 Effect：**

- 事件触发逻辑（onClick / onSubmit）
- 状态之间的推导关系
- 业务流程控制
- 试图“模拟生命周期”

---

## 🎯 适合谁阅读？

- 有 1–3 年 React 经验，但：
    - 经常被 useEffect bug 困扰
    - 对 deps 不自信
    - 不确定“该不该写 Effect”
- 希望：
    - 建立**统一、稳定、可解释的 React 副作用模型**
    - 在面试或团队中讲清楚 **为什么要这样写**

---

## 🧭 推荐阅读方式

1. **先完整读 Chapter 1**（建立心智模型）
2. 再看 Chapter 2（形成工程规则）
3. 最后用 Chapter 3 对照真实业务场景

---

> 如果你只记住一句话：

> **useEffect 解决的不是“什么时候跑”，而是：  
> “当 UI 处于这个状态时，外部世界应该长什么样”。**
