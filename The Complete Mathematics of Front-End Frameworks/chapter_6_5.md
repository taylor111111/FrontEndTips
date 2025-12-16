🌈 《动态 root* 形成的渲染路径平铺图（Cross-frame Timeline）》

— 跨时间片的 Fiber 执行路径可视化

我们用之前那棵树：

```code
      R
     / \
    A   B
   / \   \
  C   D   E
```

在渲染过程中，会跨越多个 frame：

```code
Frame 1   Frame 2   Frame 3   Frame 4   Frame 5
```

每个 frame 内部会运行一个 work loop，执行 `prefix(DFS_root*)`。

---

🟦 图例（Legend）

```code
[ ] = 当前时间片正在执行的 Fiber 节点
root* = 当前 traversal 的根节点（动态变化）
→ = Fiber DFS 中的 nextUnitOfWork 路径
… = 未执行 / 跳过 / 等待调度
```

---

✨ 场景设计：一个真实的并发渲染过程

事件顺序：

1. Frame 1：正常渲染 root (R)
2. Frame 2：继续 DFS，遇到用户在 C 输入 → root* 切换为 C
3. Frame 3：优先处理 C 子树（高优）
4. Frame 4：恢复处理原本未完成的 B 子树
5. Frame 5：完成渲染

这个顺序非常贴近 React 真实行为。

---

✨ 完整平铺图开始

（这是可以直接贴到你的书里的最终图）

```code
==============================================================
Dynamic Root Selection — Cross-frame Traversal Timeline (Fiber DFS)
==============================================================

Tree:
        R
       / \
      A   B
     / \   \
    C   D   E

--------------------------------------------------------------
Frame 1                         (root* = R, normal rendering)
--------------------------------------------------------------

R.begin → A.begin → C.begin
[R ] → [A ] → [C ]                time slice ends here
                    ↑ pause here (nextUnitOfWork = C.complete)

--------------------------------------------------------------
Frame 2                         (root* = R, continue DFS)
--------------------------------------------------------------

C.complete → D.begin → D.complete → A.complete
[C ] → [D ] → [D ] → [A ]
                        ↑ event in C triggers high-priority lane
                        root* switches to C

--------------------------------------------------------------
Frame 3                         (root* = C, high-priority work)
--------------------------------------------------------------

root* = C   (new traversal start)
C.begin → C.complete
[C ] → [C ]          C subtree stabilized

Scheduler: "OK, high-priority lane done. Restore previous work."

--------------------------------------------------------------
Frame 4                         (root* = R, resume old DFS path)
--------------------------------------------------------------

Resume at previous pause chain (nextUnitOfWork = B.begin)
B.begin → E.begin → E.complete → B.complete
[B ] → [E ] → [E ] → [B ]        finish B subtree

--------------------------------------------------------------
Frame 5                         (root* = R, finish)
--------------------------------------------------------------

A.complete → R.complete
[A ] → [R ]                      final commit

======================== END OF CROSS-FRAME SEQUENCE ========================
```

---

🧠 用一句话解释这个图：

> root* 可以跨帧变化。
> 每次 root* 变化都会开启一段新的“局部 DFS”。
> 完成后再恢复之前未完成的 DFS。
> 最终渲染结果是所有这些局部 DFS 的拼接。

---

⭐ 这张图告诉你三件最重要的事实：

① 很多 frame 可以从不同 root* 开始遍历

例如 Frame 3 的 root* 不再是 R，而是 C。

这就是并发渲染的基础。

② traversal 是多段式的，而不是一次 DFS

原本应该是：

```code
R → A → C → D → B → E
```

但实际执行的是：

```code
Frame1: R → A → C
Frame2: C → D → A   (pause at B)
Frame3: C           (high priority flush)
Frame4: B → E
Frame5: A → R       (final complete)
```

这就是所谓的：

> “分段式、可中断的拓扑序列”。

③ React 并发模式 ≠ “同时渲染多个树”

而是：

> 根据 root* 选择不同入口点分次执行 DFS。

root* 是改变一切的源头。
