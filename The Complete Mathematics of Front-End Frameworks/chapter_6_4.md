

## 当 root 动态改变时，Fiber 的 traversal 路线是如何改变的

---

## 一、基础结构：一棵组件树（Fiber Tree）

我们先画一棵简单的树（6 个节点）：

```
        R
       / \
      A   B
     / \   \
    C   D   E
```

* R = 全局 root
* A、B = 子树 root
* C、D、E = 叶子节点

---

## 二、普通 DFS（无中断、无调度）：固定 traversal

普通递归 DFS 的 traversal（begin-only 简化版）：

```
R → A → C → D → B → E
```

图示箭头：

```
R
↓
A → C
↓   ↓
B   D
↓
E
```

* 一条固定路径
* 从 R 开始
* 不可中断、不可插队

---

## 三、Fiber：动态 root* = 动态 traversal 起点

我们来模拟不同 root* 的情况。

---

### 【场景 1】root* = R（正常情况：从全局 root 开始）

Traversal：

```
R → A → C → D → B → E
```

图示：

```
[Start: R]
R → A → C → D → B → E
```

---

### 【场景 2】高优更新出现在 C（局部 root = C）

例如：用户在 C 节点触发 onChange → 高优 lane → root* = C。

Fiber traversal 立即从 C 开始，而不是从 R 开始。

```
Traversal:
C → (backtrack) → D → (backtrack) → A.complete
```

图示箭头（从 C 开始）：

```
        R
       / \
      A   B
     / \   \
    →C   D   E
        ↑
   traversal 从 C 开始，走 C → D，再返回 A
```

注意：

> 不是遍历整棵树，而是：
> 只遍历 C 子树 + 必要的祖先链（C → A）。

---

### 【场景 3】root* 替换成 B（例如：Suspense fallback 或 B 子树变高优）

调度器选择：

```
root* = B
```

Traversal 立即切换：

```
Traversal:
B → E
```

图示：

```
        R
       / \
      A   →B
     / \     \
    C   D     →E
```

* 路径变短
* 直接进入 B 子树

---

### 【场景 4】root* 在渲染途中被“切换”（最高难度）

假设当前：root* = R，正在遍历 B 时，A 子树突然出现高优更新（如用户输入）。

#### 流程：

1. 开始执行 root = R 的 DFS：

```
R → A → C → D → B
              ↑ 正在遍历这里
```

2. 输入事件触发：update on A → root* = A

3. 调度器中断当前 traversal：

```
(stop at B)
```

4. 从新 root（A）重新开始 traversal：

```
A → C → D
```

图示：

```
Step 1（原 traversal）：
R → A → C → D → B ...

Step 2（调度切换 root* = A）：
        R
       / \
      A(*) B
     / \
    C   D

Traversal jump:
A → C → D
```

你可以看到：

> 中断发生在 B，但 traversal 从 A 重新开始。
> 这是 root 动态切换带来的“重规划路径”。

---

## 最终整合图（最适合放入《第六卷》）

完整展示 root* 不同 → traversal 起点不同 → 整条 DFS 路径改变：

```
        R
       / \
      A   B
     / \   \
    C   D   E

---------------------------------
Case 1: root* = R
Traversal:
R → A → C → D → B → E

---------------------------------
Case 2: root* = C
Traversal:
C → (up) A → (up) R → then maybe D ...

---------------------------------
Case 3: root* = B
Traversal:
B → E → (up) R

---------------------------------
Case 4: During R traversal, event in A:
Original:
R → A → C → D → B ...

Switch root*: A
New traversal:
A → C → D
---------------------------------
```

可见：

> root* 改变 = traversal 路径改变

---

## 你可以把图旁边的金句写成：

> Fiber 渲染不是对树的一个固定 DFS，
> 而是从多个可能的 root 中动态选择一个，
> 再对该子树执行局部 DFS。
>
> **root 的改变会导致整个 traversal 路线重写。**
