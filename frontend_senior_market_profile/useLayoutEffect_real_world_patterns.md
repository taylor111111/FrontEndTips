

# useLayoutEffect 的真实工程使用场景（非面试版）

> 这不是一篇 API 对照文章。
> 这是一篇 **只用于工程判断** 的 useLayoutEffect 使用说明。
>
> 一句话原则：
> **useLayoutEffect 用来阻止用户“看到错误的一帧”。**

---
  
## 一、先给结论（工程判断优先）

在真实工程中：

- **99% 的场景用 useEffect**
- **只有在“首帧视觉必须一致”时，才允许使用 useLayoutEffect**

useLayoutEffect 不是更高级的 useEffect，
它是一个 **更危险、但必要的同步钩子**。

---
  
## 二、React + 浏览器的真实时间线（工程版）

这是判断 useLayoutEffect 是否合理的唯一基础：

```
React render（计算 Virtual DOM）
      ↓
Commit 阶段（更新真实 DOM）
      ↓
useLayoutEffect（同步执行，阻塞）
      ↓
Browser layout / paint（用户看到画面）
      ↓
useEffect（异步执行）
```

关键分界线只有一条：

> **浏览器 paint 之前 vs 之后**

---
  
## 三、useLayoutEffect 的正当工程场景（非常少）

### 场景 1：读取布局并立刻修正（避免闪烁）

#### 问题本质

- 你 **必须读取 DOM 尺寸**
- 并且 **必须在用户看到之前同步修正**

如果用 useEffect：
- 用户会先看到一帧错误布局
- 再被“修正”

#### 示例

```tsx
function Tooltip({ targetRef }) {
  const tooltipRef = useRef<HTMLDivElement | null>(null);

  useLayoutEffect(() => {
    if (!tooltipRef.current || !targetRef.current) return;

    const rect = targetRef.current.getBoundingClientRect();
    tooltipRef.current.style.top = `${rect.bottom + 8}px`;
  }, []);

  return <div ref={tooltipRef}>Tooltip</div>;
}
```

#### 工程判断

- ❌ 允许闪烁 → 用 useEffect
- ✅ 不允许闪烁 → 才能用 useLayoutEffect

---
  
## 四、useLayoutEffect 的反例（高频误用）

### ❌ 误用 1：当成“更快的 useEffect”

```tsx
useLayoutEffect(() => {
  fetchData();
}, []);
```

❌ 错误原因：

- 网络请求与 layout 无关
- 你只是无意义地阻塞了浏览器渲染

---
  
### ❌ 误用 2：修复时序焦虑

```tsx
useLayoutEffect(() => {
  setState(...);
}, []);
```

常见心理：
> “useEffect 不稳定，我用更早的钩子”

实际问题：
- 状态设计错误
- 时间线没有被正确拆分

useLayoutEffect **不是时序补丁工具**。

---
  
## 五、一个非常重要的工程原则

你可以用下面的问题强制自己做判断：

> **如果这段逻辑晚一帧执行，  
> 用户是否能看到“明显错误的 UI”？**

- 如果 **不能接受** → useLayoutEffect
- 如果 **可以接受** → useEffect

没有第三种情况。

---
  
## 六、与 useEffect 的关系（工程视角）

| 对比点 | useLayoutEffect | useEffect |
|------|----------------|-----------|
| 执行时机 | DOM 更新后，paint 前 | paint 后 |
| 是否阻塞渲染 | ✅ 是 | ❌ 否 |
| 工程风险 | 高 | 低 |
| 默认选择 | ❌ | ✅ |

useLayoutEffect 的存在不是为了“优化”，  
而是为了 **消除首帧错误状态**。

---
  
## 七、总结（请记住这一句）

> **useLayoutEffect 是 UI 的“最后一道同步防线”，  
> 不是常规副作用工具。**

如果你在犹豫要不要用它——  
**那答案几乎一定是：不要用。**
