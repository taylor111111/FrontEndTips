# 1. 从 AMD / CMD 看“模块化”的本质

先把术语抽象掉，只保留底层结构：

> 模块化 = 把一大坨脚本拆成有名字的节点（module），  
> 再用依赖边（import / require）把它们连起来。

也就是你熟悉的：

```text
Graph = (Modules, Dependencies)
```

---

## 1.1 AMD（RequireJS 一派）

核心特征：

```js
define(['dep1', 'dep2'], function (dep1, dep2) {
  // ...
})
```

- 依赖前置声明：一开始就列出所有依赖
- 天然异步：浏览器环境下，加载 JS 文件，加载完再执行回调
- 适合浏览器：因为浏览器加载脚本本来就是异步

数学视角：

**AMD 在模块定义时就给出了节点的入边列表，loader 根据依赖图做一次拓扑加载。**

---

## 1.2 CMD（SeaJS 一派）

CMD 大致长这样：

```js
define(function (require, exports, module) {
  const a = require('./a')
  // 用到的时候再 require
})
```

- 依赖就近：用到哪里，哪里 require
- 设计目标：更贴近 CommonJS 写法
- Loader 做更多运行时分析：在执行阶段才知道需要加载什么

数学视角：

**CMD 的依赖图是在「执行时」逐步暴露的，不像 AMD 在定义时就静态列好。**

---

你可以把两者总结成一句话：

- **AMD**：依赖图在「定义阶段」就显式给出  
- **CMD**：依赖图在「执行阶段」逐步显现  

这两种差异，后来被打包工具 / ES Module / dynamic import 吃干抹净了。

---

# 2. 从 AMD / CMD 演化到现在：模块化的三条主线

从历史演化来看，有三条主线同时在走：

1. **Node.js 世界**：CommonJS（同步 require）
2. **浏览器世界**：AMD / CMD（异步加载 + define / require）
3. **标准化世界**：ES Modules（import / export）

最终的收敛是：

- 语法层面：以 **ES Module** 为主（import / export）
- 打包层面：webpack / Rollup / esbuild 把所有模块变成一个「静态依赖图」
- 运行层面：通过「代码分割 + 懒加载」决定运行时加载哪个子图

你现在实际用到的东西是：

- `import / export`（语法）
- bundler 里的 `code splitting + dynamic import()`（工程策略）
- React 里的 `React.lazy / route-level 懒加载`（UI 层策略）

可以把它理解为：

> **AMD / CMD 做的事，现在基本由  
> “ESM + bundler + dynamic import” 合体完成。**

---

# 3. 模块化开发：工程上真正需要关心的几个点

从你的角度，不需要再纠结「AMD 和 CMD 的历史八卦」，更重要的是这几个问题：

---

## 3.1 模块边界怎么划？

几个实用的“脑子舒服”原则：

- **高内聚，低耦合**：一个 module 做一类事情
- **对外只暴露一个稳定的 API**
  - `index.ts` 作为模块门面
  - 内部文件是实现细节

有清晰分层：

- `domain`（业务领域）
- `ui`（组件）
- `infra`（网络、存储、配置）

你可以用我们前面 Fiber 的方式看它：

> 整个项目 = 模块图  
> 每个模块 = 一个子树 / 子图  
> 对外只暴露根节点 API。

---

## 3.2 循环依赖 / 初始化顺序的问题

这类问题最常见：

- A import B，B 又 import A → 运行时是 `undefined`
- 某个模块在被 import 时就执行副作用逻辑（注册、打 patch），导致初始化顺序非常敏感

工程上的应对方案：

1. **尽量把“数据结构定义”与“执行逻辑”拆开**
   - `config.ts`：纯对象 / 函数
   - `bootstrap.ts`：负责 wire up & 执行

2. **跨层调用时需要一个中间“service 层”**
   - 避免 UI 模块和 infra 模块互相 import 造成环

3. **避免在顶层做复杂副作用**
   - 顶层文件更适合定义“结构”，不适合做“动作”

你完全可以把这套看成 **“拓扑排序 + 消除环”** 的工程版本。

---

# 4. 懒加载：从“模块图”走向“按需加载”

懒加载本质是：

> 把“模块依赖图”切成多个子图，  
> 按时间 / 场景加载。

---

## 4.1 常见切法（前端工程里最常见三种）

### 1. 路由级切割（最常见）

- 每个页面 / route 对应一个 chunk
- 只加载当前 route 的 chunk
- React Router / Next.js / Vue Router 都有集成方案

---

### 2. 组件级切割（比如某些重组件）

- `React.lazy(() => import('./BigChart'))`
- 重 UI、图表、编辑器、富文本，都可以独立懒加载

---

### 3. 功能 / 工具级切割

- 比如一个只在特定交互场景下出现的 expensive util / SDK
- 用 `import()` 动态加载，减少首屏 bundle

---

## 4.2 用 AMD 思维看现在的 dynamic import

AMD 写法当年是：

```js
require(['chart'], function (chart) {
  // 用到时才加载 chart
})
```

现在：

```ts
const Chart = React.lazy(() => import('./Chart'))
```

本质是一模一样的：

- 把某个子图抽成「延迟计算」的节点
- 在图的某一条路径被访问时，才展开“这部分”

只是以前 loader 在运行时下载 JS，  
现在 bundler 在构建期把它切成 chunk + runtime loader。
