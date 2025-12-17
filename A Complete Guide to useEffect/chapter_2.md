## 10. One Effect Per Event Source：按“事件来源”拆 Effect

### 核心思想

- 一个 Effect 只负责一个事件源：
  - 浏览器事件
  - WebSocket 通道
  - 某个数据订阅
  - 某类 side-effect

### 最佳实践

- 不要写“混合型 Effect”：

```js
useEffect(() => {
  // resize
  window.addEventListener('resize', onResize);
  // scroll
  window.addEventListener('scroll', onScroll);
  // 还顺便发个请求
  fetchData();

  return () => {
    window.removeEventListener('resize', onResize);
    window.removeEventListener('scroll', onScroll);
  };
}, []);
```

- 改成：

```js
useEffect(() => {
  window.addEventListener('resize', onResize);
  return () => window.removeEventListener('resize', onResize);
}, [onResize]);

useEffect(() => {
  window.addEventListener('scroll', onScroll);
  return () => window.removeEventListener('scroll', onScroll);
}, [onScroll]);

useEffect(() => {
  fetchData();
}, [fetchData]);
```

---

## 11. Event‑specific Effects & 需要“事件思维”的 Effects

### 核心思想

- 有些副作用，本质上是“被某个动作触发的”，而不是“状态自然变化就该触发的”：
  - “点击按钮”触发的请求
  - “提交表单”触发的埋点

- 这类逻辑不要放在依赖 state 的 Effect 里，否则：
  - 只要 state 变了就会触发
  - 很容易造成重复请求 / 死循环

### 最佳实践

- 用“事件思维”处理：
  - 在 `onClick / onSubmit` 中直接触发；
  - 或者抽成 `useCallback`，由 handler 调用；
  - Effect 仅处理真正需要“根据结果状态同步到外部”的部分。

---

## 12. Resetting Effects：通过重建实例重建时间线

### 核心思想

- 有些行为的语义是“完全重新开始”，比如：
  - 打开一个弹窗编辑新的 item
  - 重新进入某个 wizard 流程

- 在 Dan 的视角里：

  > **“重建实例 = 重建时间线”**

- 可以直接用 `key` 强制 React 把子树当成一个全新的组件。

### 最佳实践

```jsx
{isOpen && (
  <EditDialog key={itemId} itemId={itemId} />
)}
```

- 每次 `itemId` 变化，`EditDialog` 整个子树被卸载 + 重建：
  - state 清零
  - Effect 重新跑

- 逻辑更贴近你说的：

  > “编辑一条新信息，语义上就是一个新的实例。”

---

## 13. 总结：从 useEffect 提炼出来的 React 实践规则

我帮你总结成可以挂在 README 顶部的一组 bullet：

1. Effect 是同步描述，不是生命周期脚本。
2. 每次渲染都是一张独立快照，Effect 只看到那张快照。
3. cleanup = 撤销上一个快照对外部世界的修改。
4. 依赖数组是 Effect 的 diff key，不能对依赖说谎。
5. 少写依赖的正确方式是重构，而不是删依赖。
6. 事件 → 状态更新，用 handler / useReducer 管；
   状态结果 → 副作用，用 Effect 管。
7. 一个 Effect 只负责一个事件源（或一个同步关系）。
8. 事件驱动逻辑用“事件思维”，不要强行塞进依赖型 Effect。
9. 需要“重开时间线”的场景，优先用 `key` 重建子树。
