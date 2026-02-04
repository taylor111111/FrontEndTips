

# React StrictMode 下的 effect 行为（工程记录）

> 这是一份 **工程记录文档**，不是面试答案。
> 目的只有一个：  
> **解释为什么在 React 18 开发环境 + StrictMode 下，会看到“多一次 render / effect”。**

---

## 一、问题背景（真实现象）

在 React 18 + 开发环境 + StrictMode 下，很多人会观察到：

```tsx
function Comp() {
  console.log('render');

  useEffect(() => {
    console.log('effect');
    return () => console.log('cleanup');
  }, []);

  return null;
}
```

控制台输出类似：

```
render
render
effect
cleanup
effect
```

这常常引发误解：
- `useEffect` 会不会触发 render？
- React 是不是偷偷重渲染？
- 代码是不是写错了？

---

## 二、核心结论（先给答案）

> **这不是组件逻辑问题，也不是 useEffect 的语义问题。  
> 这是 React StrictMode 在开发环境下的“副作用检测机制”。**

---

## 三、StrictMode 实际做了什么（准确描述）

在 **React 18 开发环境** 中，StrictMode 会**刻意**让组件经历一次完整的生命周期压力测试：

```
mount
  ├─ render
  ├─ commit
  └─ effect
unmount（模拟）
  └─ cleanup
mount（再次）
  ├─ render
  ├─ commit
  └─ effect
```

关键点：

- 是 **两次 mount**
- 不是一次 render + 重跑 effect
- cleanup 是 **刻意插入的**

---

## 四、为什么 React 要这样做？

StrictMode 的目标不是“改变行为”，而是 **暴露问题**。

它试图帮助你发现：

- effect 是否依赖了不稳定的外部状态
- effect cleanup 是否正确
- effect 是否假设“只会执行一次”
- effect 是否存在隐藏副作用（比如重复注册）

这些问题，在未来的并发特性下都会变成 bug。

---

## 五、必须明确区分的三件事（非常重要）

### 1️⃣ useEffect 本身不会触发 render

> **触发 render 的永远是：state / props / context 变化。**

`useEffect(() => { console.log() })`  
不会触发 render。

---

### 2️⃣ 多一次 render ≠ 业务逻辑多跑一次

StrictMode 下的多执行：
- 只存在于 **开发环境**
- 不存在于 **生产环境**
- 不代表真实用户行为

---

### 3️⃣ 这是框架行为，不是组件行为

- 不应该“修复”
- 不应该“规避”
- 只需要 **理解它**

---

## 六、工程上的正确态度

这类问题：

- 不需要写额外代码处理
- 不需要加 flag 判断
- 不需要用 ref 去“挡住第二次执行”

正确做法只有一个：

> **写出“可重复执行、可安全清理”的副作用。**

---

## 七、为什么这是工程问题，而不是面试问题

原因很简单：

- 这是 **开发工具行为**
- 和业务逻辑无关
- 和 API 熟练度无关
- 展开讨论只会制造噪音

在面试中：
- 即使你完全理解
- 也没有必要展开说明
- 回答反而可能引发无意义追问

---

## 八、一个实用的记忆锚点

> **StrictMode 的额外执行  
> 是一次“副作用体检”，不是“真实运行路径”。**

---

## 九、总结（一句话）

> **如果你在 StrictMode 下看到 effect / render 多执行一次，  
> 这是 React 在帮你发现问题，而不是你的代码有问题。**
