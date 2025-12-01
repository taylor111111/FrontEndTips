```html
React Timeline Model（核心哲学）
│
├── 1. 渲染快照（Snapshot）
│     │
│     ├── 每次 render = 一个静态快照
│     ├── 快照包含：
│     │     • UI 结构
│     │     • props（稳定输入）
│     │     • state（当前值）
│     │     • memo/cache（当前缓存）
│     │     • effect dependencies（当前订阅）
│     └── “世界暂停” → 记录当前场景
│
├── 2. 时间线（Instance = Snapshot Sequence）
│     │
│     ├── 实例 ≠ 对象
│     ├── 实例 = 多个快照按时间排列
│     │     render #1 → #2 → #3 → …
│     ├── 表示“组件的生命周期”
│     └── “时间线连续”非常重要
│
├── 3. Identity（组件的“身份”）
│     │
│     ├── identity = 时间线的指针
│     ├── 由 key、fiber、位置共同定义
│     ├── identity 相同 → 时间线继续
│     └── identity 改变 → 开启新时间线
│
├── 4. Reset（重建实例 = 重建时间线）
│     │
│     ├── 触发条件：
│     │     • key 改变
│     │     • 组件卸载 → 再挂载
│     │     • 路由切换
│     │     • 条件渲染从 false→true
│     ├── reset 的结果：
│     │     • state 被清空
│     │     • effect 清理
│     │     • 订阅解除
│     │     • 从“快照#1”开始
│     └── reset = 全新宇宙
│
├── 5. Diff（在同一时间线上前进）
│     │
│     ├── 不满足 reset → diff
│     ├── diff 仅更新变化的部分
│     ├── diff 保留：
│     │     • state
│     │     • memo cache
│     │     • effect identity
│     └── diff = 同一宇宙下一帧
│
├── 6. Effects（时间线中的副作用）
│     │
│     ├── effect 属于“快照#N”记录的订阅
│     ├── effect 在每一帧的变化逻辑：
│     │     • deps 没变 → 不重新订阅
│     │     • deps 变了 → 清理旧的 → 安装新的
│     ├── effect 永远属于“某一帧”
│     └── effect 是处理“外部世界变化”的方法：
│           DOM、事件、网络、localStorage…
│
├── 7. Event Model（事件驱动时间线跳转）
│     │
│     ├── UI 事件导致：
│     │     • state 更新
│     │     • 触发下一 render（下一帧）
│     │
│     ├── “事件函数”属于当前快照
│     │     • props 中的 handler
│     │     • useCallback 输出的 handler
│     │
│     └── 事件链：事件 → setState → 下一快照
│
└── 8. 开发者语义（你需要决定 diff / reset）
        │
        ├── 当语义是“继续编辑当前对象” → diff
        │     state, effect, cache 全部沿用
        │
        ├── 当语义是“这是一条全新的信息” → reset
        │     key 改变 → 新时间线
        │
        └── React 不知道你的语义  
            必须由你告诉 React：  
            **“这条时间线继续”还是“开一条新的”**

```

