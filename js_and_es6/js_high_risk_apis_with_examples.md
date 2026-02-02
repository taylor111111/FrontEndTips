# JS 中真正“值钱”的高风险 API（附工程级示例）

> 本文目标不是“列 API”，而是明确：  
> **哪些 JS API 一旦用错，代价极高，且在真实工程中频繁出现。**

判断标准只有三个：
- 高频使用
- 极易被误用
- 一旦出错，排查成本高、影响范围大

---

## 1️⃣ Array.prototype.sort（原地修改）

### ❌ 常见误解
> sort 会返回一个新数组

### ✅ 实际语义
```js
arr.sort() // 会直接修改 arr 本身
```

### 💥 事故示例
```js
const list = [{ id: 1 }, { id: 2 }];
const sorted = list.sort((a, b) => b.id - a.id);

// 你以为只是 sorted 变了
// 实际 list 也被修改
```

### 🧠 工程结论
- `sort` 是 **原地 mutation**
- 在 state / props / cache 数据上使用极其危险

### ✅ 安全替代
```js
const sorted = [...list].sort(...)
```

---

## 2️⃣ Array.prototype.reverse（原地修改）

### 💥 事故示例
```js
const list1 = [{ k: 1 }, { k: 2 }];
const list2 = [...list1].reverse();

list2[0].k = 3;

// list1[1].k === 3
```

### 🧠 工程结论
- reverse + 浅拷贝 = **引用共享陷阱**
- 极容易引发“幽灵修改”

---

## 3️⃣ Object.assign（浅拷贝，不是合并）

### ❌ 常见误解
> Object.assign 是深拷贝

### 💥 事故示例
```js
const a = { conf: { dark: true } };
const b = Object.assign({}, a);

b.conf.dark = false;
// a.conf.dark === false
```

### 🧠 工程结论
- 只拷贝第一层
- 内部对象仍然共享引用

---

## 4️⃣ 展开运算符 `...`（看起来安全，其实不是）

### 💥 事故示例
```js
const state = { user: { name: 'A' } };
const next = { ...state };

next.user.name = 'B';
// state.user.name === 'B'
```

### 🧠 工程结论
- `...` 只是 **语法糖版浅拷贝**
- 在嵌套结构中非常危险

---

## 5️⃣ Array.prototype.map（以为是纯函数，其实不是）

### ❌ 常见误用
```js
list.map(item => {
  item.count += 1;
  return item;
});
```

### 💥 后果
- map 被用成了 mutation 工具
- 所有引用方都会被影响

### ✅ 正确姿势
```js
list.map(item => ({
  ...item,
  count: item.count + 1
}));
```

---

## 6️⃣ JSON.parse(JSON.stringify())（伪深拷贝）

### ❌ 风险点
- Date / Map / Set / undefined 丢失
- 循环引用直接炸
- 精度问题

### 🧠 工程结论
> 这是 **调试期临时工具**，不是工程方案

---

## 7️⃣ Array.prototype.splice（最危险的数组 API）

### 💥 事故示例
```js
const list = [1, 2, 3];
const removed = list.splice(1, 1);

// list === [1, 3]
```

### 🧠 工程结论
- splice = 删除 + 插入 + mutation
- 在任何共享数据结构中都极其危险

---

## 8️⃣ 引用传参（不是 API，但必须警惕）

### 💥 事故示例
```js
function update(config) {
  config.enabled = false;
}

update(sharedConfig);
```

### 🧠 工程结论
- JS 函数参数是 **引用传递**
- 函数 = 隐式修改点

---

## 9️⃣ setTimeout（时间线事故）

### ❌ 常见误解
> setTimeout 只是“延迟执行”，和当前逻辑是连续的

### 💥 事故示例（React 中极其常见）
```js
useEffect(() => {
  setTimeout(() => {
    setCount(count + 1);
  }, 1000);
}, []);
```

### 💥 实际问题
- 回调函数捕获的是 **旧的 count**
- 定时器运行在 **独立时间线**
- 状态永远停留在初始值

### ✅ 工程正确姿势
```js
setCount(c => c + 1);
```

### 🧠 工程结论
> `setTimeout` / `setInterval` 拥有 **独立时间线**，  
> 永远不要假设它“跟当前 render 同步”。

---

## 🔟 Promise.then（链断事故）

### ❌ 常见误解
> then 只是顺序写法

### 💥 事故示例
```js
fetchData()
  .then(res => {
    process(res);
  })
  .then(result => {
    // result === undefined
  });
```

### 💥 实际问题
- 忘记 return
- Promise 链被 **悄悄截断**
- 后续逻辑静默失效

### ✅ 正确写法
```js
fetchData()
  .then(res => {
    return process(res);
  })
  .then(result => {
    // 正常使用
  });
```

### 🧠 工程结论
> Promise 链不是语法糖，  
> **每一段 then 都必须明确返回值**。

---

## 1️⃣1️⃣ Promise.all（误解极多）

### ❌ 常见误解
> 只要有一个成功就算成功

### ✅ 实际语义
- 任意一个 Promise reject
- 整个 Promise.all **立即 reject**

### 💥 事故示例
```js
Promise.all([
  fetchA(),
  fetchB(), // reject
  fetchC()
]);
```

结果：
- fetchA / fetchC 即使成功
- 结果仍然整体失败

### 🧠 工程结论
> `Promise.all` 是 **强一致并发工具**，  
> 不是容错工具。

---

## 1️⃣2️⃣ forEach（不能 await）

### ❌ 常见误用
```js
items.forEach(async item => {
  await save(item);
});
```

### 💥 实际问题
- forEach 不等待 Promise
- 错误无法捕获
- 执行顺序不可控

### ✅ 正确姿势
```js
for (const item of items) {
  await save(item);
}
```

或：

```js
await Promise.all(items.map(save));
```

### 🧠 工程结论
> **forEach ≠ 异步控制结构**

---

## 1️⃣3️⃣ Date（时间基准错误）

### ❌ 常见误用
```js
const base = new Date(); // 页面初始化时

function last7Days() {
  return [base, addDays(base, 7)];
}
```

### 💥 真实事故
- 页面长时间不刷新
- 第二天点击“近 7 天”
- 时间仍然从昨天开始算
- QA 难以复现

### 🧠 工程结论
> **时间必须在“使用时获取”，  
> 而不是在“初始化时缓存”。**
