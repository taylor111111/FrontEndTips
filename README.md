# FrontEndTips

前端开发经验与技巧合集 📘  
记录在实际项目中遇到的细节优化与最佳实践。

---

## 📂 目录

### 📐 The Complete Mathematics of Front‑End Frameworks

> 一套从 **UI = State Machine** 到 **Fiber = 可中断拓扑遍历** 的完整数学化前端框架理论。

- [Chapter 1：UI = 状态机的数学本质](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_1.md)
- [Chapter 2：State × Action = UI 状态空间](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_2.md)
- [Chapter 3：Reducer、纯函数与可证明系统](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_3.md)
- [Chapter 4：UI 枚举、可达状态与有限状态机](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_4.md)
- [Chapter 4.x：Chapter 4 的数学补充（UI 枚举深化）](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_4_x.md)
- [Chapter 5：副作用 = 世界输入的数学化（Effect Boundary）](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_5.md)
- [Chapter 6：Fiber = 拓扑 + 调度 + 离散时间](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_6.md)
- [Chapter 6.1：Fiber = 可中断的拓扑排序](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_6_1.md)
- [Chapter 6.2：Dynamic Root Selection（动态根选择）](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_6_2.md)
- [Chapter 6.3：Dynamic Root Selection 的形式化模型](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_6_3.md)
- [Chapter 6.4：当 root 动态改变时，Fiber traversal 如何改变](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_6_4.md)
- [Chapter 6.5：Fiber、调度与优先级系统](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_6_5.md)
- [Chapter 6.6：Fiber 总结（数学视角）](The%20Complete%20Mathematics%20of%20Front-End%20Frameworks/chapter_6_6.md)

---

### 🧭 UI 对齐与视觉优化
- [InfoTipIcon 文字与图标对齐技术说明](docs/InfoTipIcon_Alignment_Wiki.md)
- [图标基线对齐与 translateY 微调原理说明](docs/Icon_Baseline_and_translateY.md)
- [图标与文字的基线对齐：从视觉误差到完美修正](docs/Icon_Baseline_Perfect_Alignment.md)
- [全局 Scrollbar 样式与策略 Wiki](docs/global-scrollbar-strategy.md)

### 🔁 A Complete Guide to useEffect 读书笔记

- [Chapter 1：useEffect 的本质与心智模型（Snapshot & Synchronization）](A%20Complete%20Guide%20to%20useEffect/chapter_1.md)
- [Chapter 2：从 useEffect 提炼的工程实践规则](A%20Complete%20Guide%20to%20useEffect/chapter_2.md)
- [Chapter 3：useEffect 的典型业务场景与必须使用的情况](A%20Complete%20Guide%20to%20useEffect/chapter_3.md)

---

### 🧱 AMD & CMD → 模块化开发 → 懒加载 → 工程问题

> 从 **模块依赖图（Graph）** 出发，系统性讲清  
> **模块如何划分、如何避免循环依赖、懒加载如何设计、工程问题如何兜底**。

- [Chapter 1：从 AMD / CMD 看“模块化”的本质](modular-architecture/chapter_1.md)
- [Chapter 2：懒加载带来的工程问题与处理方式](modular-architecture/chapter_2.md)
- [Chapter 3：懒加载的四条“硬标准”](modular-architecture/chapter_3.md)
- [Chapter 4：分层结构与依赖图设计（domain / ui / infra / features / app）](modular-architecture/chapter_4.md)
- [Chapter 5：模块依赖图 → 懒加载子图 → UX 状态机 → Chunk 结构](modular-architecture/chapter_5.md)
- [Chapter 6：一张结构图，回答模块拆分 / 循环依赖 / 懒加载设计](modular-architecture/chapter_6.md)

### ⚙️ 性能与工程化
- （待补充）Webpack 打包优化
- （待补充）懒加载与路由分片策略

### 🧩 组件开发
- （待补充）表单组件封装规范
- （待补充）Tooltip 与交互延迟优化

### 🧠 前端思维与经验
- （待补充）状态管理与副作用思考
- （待补充）设计一致性与开发落地
