

# Part 2：代數資料型別（ADT）與型別系統 —— FP 的數學骨架

> **本章目標：**讓你建立 *為什麼 FP 的資料結構比物件導向更可靠* 的深層理解。讓「代數資料型別」成為你日後寫演算法、寫 Redux、寫 React 的數學武器。

---

## 1. 什麼是 Algebraic Data Type（代數資料型別）？

ADT 不僅僅是「一種型別」，它是：

> **用代數方法組合型別，讓程式具備可推理性。**

兩種最核心 ADT：

### **1.1 Product Type（乘積型別）**

也叫 **結構型（Record）**。

數學上：

```
A × B
```

工程上：

```js
const user = { name: "Taylor", age: 27 }
```

含義：

* 想要一個 `user`，**兩者都得提供**（AND）
* 結構是固定的，可推理

→ Redux 的 `state`、React 的 `props` 本質上全部都是「乘積型別」。

### **1.2 Sum Type（和型別）**

也叫 **Union / Variant / Enum**。

數學上：

```
A + B
```

工程上（TypeScript）：

```ts
type Result = Success | Failure
```

含義：

* 值**只可能是其中之一**（OR）
* 必須被完整覆蓋處理（Exhaustiveness）

→ Redux 的 action、後端 protocol、錯誤處理，全都是「和型別」。

---

## 2. 為什麼 Product + Sum = 所有資料結構？

這是 ADT 最強大的地方：

> **任何資料結構，都能分解成 Product（×）和 Sum（+）。**

例如：React Component 的 props 描述：

```
ComponentProps = (固定結構) × (可選組態) × (callback)...
```

錯誤處理：

```
Result<T> = Ok(T) + Err(E)
```

Redux action：

```
Action = TYPE_A + TYPE_B + TYPE_C
```

FP 的資料模型如此強，是因為它把所有東西都化成「代數結構」。

---

## 3. Pattern Matching：為什麼 FP 代碼好讀？

Sum Type 讓你可以這樣解構資料：

```ts
switch (result.type) {
  case "Ok": return render(result.value)
  case "Err": return showError(result.error)
}
```

這是：

* 完整覆蓋每一個可能性
* 不會漏
* 不會錯
* 天然可推理

因此 FP 語言普遍可讀性更高。

React 的 JSX 其實也有匹配概念：

```jsx
{status === "loading" && <Spinner />}
{status === "error" && <Error />}    
{status === "ok" && <Dashboard />}
```

這就是 **Sum Type → UI 的天然映射**。

---

## 4. 型別系統的作用：控制副作用與保證正確性

FP 型別系統不是「限制」，而是：

> **用數學保證你的程式“永遠不會寫錯一類 bug”。**

例如：

* Product Type 保證 props 結構完整
* Sum Type 保證 action 有限且可控
* 泛型（Generics）保證運算過程類型一致

FP 的型別= *約束很少，但穩定性極高*。

---

## 5. 工程視角：TS 如何偷偷在推動 FP？

TypeScript 讓 JavaScript 慢慢變成半 FP 語言：

### 5.1 Union Type → Sum Type

```ts
type Status = "loading" | "error" | "ok"
```

完全 FP。

### 5.2 Record → Product Type

```ts
type User = { name: string, age: number }
```

完全 FP。

### 5.3 Discriminated Union

```ts
type Action = 
  | { type: "ADD"; value: number }
  | { type: "RESET" }
```

這就是 Redux Action 的數學形式。

---

## 6. FP 型別系統的真正力量：可組合性（Composability）

FP 強在：

> **資料結構可以被代數地組合，而不破壞可推理性。**

例如：

```ts
type ApiResponse<T> = Result<T> | Timeout | NetworkErr
```

你的處理函數仍然保持：

```ts
response → UI
```

結構清晰，不會被「例外情況」擊破。

---

## 7. 這章如何幫助你之後走算法 / 可視化 / 科研？

ADT 是：

* **狀態機的數學語言**
* **DP 狀態定義的最佳模型**
* **所有 UI 的核心抽象**
* **所有協議與資料格式的源頭**
* **科研寫作必備的形式化語言**

Taylor，你在做 DP 視覺化時：

* 狀態空間是 Product Type
* 子問題選擇是 Sum Type

你自然會用 ADT 的方式建構整個解決方案。

這是你科研能力的底層组件。

---
