# react 把树做成一个「可中断的拓扑排序」

> 把树做成一个「可中断的拓扑排序」。
>
> 实质在说：
> **React 把「整棵组件树的遍历」变成了一条「可以随时暂停和继续的线性任务序列」。**

---

## 1. 先说普通的「树 + 拓扑顺序」

对一棵树来说，本来就有一种天然的拓扑顺序：

> 父节点一定在子节点被访问。

最典型的是 DFS 前序遍历：

```
      A
     / \
    B   C
   / \   \
  D   E   F

前序 DFS 顺序：A, B, D, E, C, F
```

这条线性的顺序，本质上就是：

> 对树做了一次拓扑排序（topological order）：
> 父一定排在子之前，没有“先儿子再父亲”的情况。

在传统递归 DFS 里，这条顺序是**一次性跑完的**：

```js
function walk(node) {
  // 1. 处理当前节点
  doWork(node)

  // 2. 递归处理子树
  for (const child of node.children) {
    walk(child)
  }
}
walk(root)
```

这个过程有几个特点：

* 一旦开始 `walk(root)`，中途不能暂停
* 调用栈是隐式的（用 JS 的函数栈）
* 不能说：

  > “我先处理一半树，先让 UI 响应一下，再回来”

这是传统渲染的 **「一口气渲染完」**。

---

## 2. React 想要的是：这条遍历顺序要「可中断」

React 想要的是：

> 这条 A, B, D, E, C, F 的遍历顺序，
> **不要一次性跑完，而是变成一串小任务**。

比如：

```
A, B, D, E, C, F
```

被拆成：

* A
* B
* D
* E
* C
* F

每一个都是一个 **WorkUnit**。

你可以：

* 跑几个就停一下
* 下次从上次停的地方继续

也就是说：

* 把“树的遍历” → 变成一个 **可暂停的迭代器**
* 每一步只处理 **一个 Fiber 节点**
* 每处理一个节点，都可以：

    * 检查时间片是否用完
    * 看有没有更高优先级的任务插队
    * 决定要不要把控制权还给浏览器

伪代码会变成这样：

```js
let nextUnitOfWork = rootFiber

function workLoop(deadline) {
  while (nextUnitOfWork && deadline.timeRemaining() > 0) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork)
  }

  // 如果还有 work 没做完，下次从 nextUnitOfWork 接着来
  if (nextUnitOfWork) {
    requestIdleCallback(workLoop)
  } else {
    commitRoot()
  }
}
```

关键点是：

> `performUnitOfWork` **只处理一个 Fiber 节点**，
> 并返回「下一个要处理的 Fiber」。

---

## 3. 问题来了：「下一个要处理的节点是谁？」

如果我们用递归 DFS：

* “下一个是谁”是**藏在调用栈里**的
* JS 本身不允许你随时暂停、恢复调用栈

Fiber 的 trick 是：

> **把“调用栈”摊平到数据结构本身。**

每个 Fiber 节点都有三个关键指针：

```ts
type FiberNode = {
  child: FiberNode | null    // 第一个子节点
  sibling: FiberNode | null  // 下一个兄弟节点
  return: FiberNode | null   // 父节点
  // ... 其他属性
}
```

现在我们不用函数栈了，而是靠这三个指针，

> 在树上做一个 **“手动 DFS”**。

典型的 Fiber 遍历逻辑（简化版）：

```js
function performUnitOfWork(fiber) {
  // 1. 在“向下”阶段处理当前 fiber（beginWork）
  beginWork(fiber)

  // 2. 优先走 child（深入子树）
  if (fiber.child) return fiber.child

  // 3. 没有 child，尝试走 sibling（兄弟）
  let next = fiber
  while (next) {
    completeWork(next)      // “向上”阶段：收尾
    if (next.sibling) {
      return next.sibling  // 切到兄弟分支
    }
    next = next.return     // 回到父节点，继续向上找
  }

  // 4. 没有父也没有兄弟，遍历结束
  return null
}
```

---

## 4. 这串顺序，就是「树的可中断拓扑排序」

如果把刚才的示意树跑一遍：

```
树结构：
A
├─ B
│  ├─ D
│  └─ E
└─ C
   └─ F
```

Fiber 遍历顺序（带阶段）：

```
A (begin)
B (begin)
D (begin) → D (complete)
E (begin) → E (complete)
B (complete)
C (begin)
F (begin) → F (complete)
C (complete)
A (complete)
```

你可以把每一个“访问节点的时刻”，都当成一个**离散任务**：

```
[A.begin],
[B.begin],
[D.begin], [D.complete],
[E.begin], [E.complete],
[B.complete],
[C.begin],
[F.begin], [F.complete],
[C.complete],
[A.complete]
```

任何一步都可以：

* 跑到 `D.complete` 停
* 下次从 `E.begin` 继续
* 或者插入别的高优任务

这条「线性任务序列」就是：

> **对整棵树的一种拓扑排序（父先于子），
> 但被拆成了一连串可暂停的离散 steps。**

这就是：

> 把树做成一个「可中断的拓扑排序」。

---

## 5. 直观类比：把 DFS 变成「随时可停的楼层巡逻」

想象你在一栋大楼里巡检：

* 这栋楼 = 组件树
* 每个房间 = 一个 Fiber

普通 DFS：

* 一口气从顶楼走到地下室，不停

Fiber 式巡检：

* 每进入 / 离开一个房间，都是一步 step
* 每走几步就看看：

    * 有没有急事（高优任务）
    * 有没有别的楼层要先处理
* 有的话，先退出当前巡检，下次再回来

为了做到「随时能继续」，你需要：

* 记住「我现在在哪个房间」（当前 Fiber）
* 记住「我下一步去哪」（child / sibling / return）

这正是 Fiber 的三个指针 + `nextUnitOfWork`。

---

## 6. 总结成一句你可以写进笔记里的话

普通 DFS：

> 用语言调用栈做隐式拓扑遍历，一旦开始，中途不能停。

Fiber：

> 把树结构用 `child / sibling / return` 摊平成一条**显式的拓扑遍历序列**，
> 每个 Fiber 节点访问都变成一个独立的 WorkUnit，
> 调度器可以在任意 WorkUnit 之间暂停、切换、恢复。

**这就是：**

> **React 把树做成一个「可中断的拓扑排序」。**
