# frontend_senior_market_profile

> 本目录用于系统性整理：  
> **高级前端工程师在真实工程与真实面试中，会反复遇到、但经常被错误提问和评估的问题空间。**

这里不是 API 笔记，也不是框架说明。  
而是一份 **工程问题的“边界图”**。

---

## 为什么会有这个目录

在实际工作和面试中，我反复遇到这样一类问题：

- 问题本身是真实存在的  
- 但问题没有被清晰定义  
- 面试官不知道合理的解空间在哪里  
- 责任边界被无限外推到前端工程师身上  
- 最终演变为：
  - 消耗精力的发散追问
  - 无法被验证的主观判断

这个目录的目标是：

- 把工程问题本身说清楚  
- 把前端工程师的责任边界画清楚  
- 区分：
  - 工程问题
  - 架构问题
  - 产品方案问题
- 为面试与实际讨论准备**安全、可收敛、不耗能的表达方式**
- 让工程师能够长期存活，而不是靠硬扛

---

## 核心理念

> **高级工程师的价值，不在于吞下多少复杂度，  
而在于能否准确识别、定义、并控制复杂度。**

本目录关注的不是「你会不会写」，而是：

- 你是否知道自己在解决什么问题  
- 你是否知道哪些问题不该由你单独解决  
- 你是否能在复杂系统中保持理性与边界感  

---

## 文件结构说明

### 二、核心工程问题空间（重点）

这些文件讨论的是**真实工程中无法回避的问题**，而不是某个框架的使用方法。

- **[http_concurrency_engineering.md](./http_concurrency_engineering.md)**  
  HTTP 并发管理问题的工程本体。

- **[http_concurrency_engineering_playbook.md](./http_concurrency_engineering_playbook.md)**  
  一个完整的工程问题拆解清单，包括：
  - 去重（Deduplication）
  - 取消（Cancellation）
  - 竞态控制（Race Condition）
  - 缓存（Caching）
  - 过期与刷新（Stale / Refetch）
  - 共享状态（Server State）
  - 重试与退避（Retry / Backoff）
  - 资源回收（GC / Memory）
  - 状态可观测性（Loading / Error）

- **[promise_engineering_problem_space.md](./promise_engineering_problem_space.md)**  
  Promise 在工程中的真实问题空间，而非语法技巧。

- **[promise_common_pitfalls_with_examples.md](./promise_common_pitfalls_with_examples.md)**  
  Promise 在真实工程中最常见的误用场景（附带代码示例）。

- **[promise_common_pitfalls_with_fix_answers.md](./promise_common_pitfalls_with_fix_answers.md)**  
  面试与工程讨论中，对 Promise 常见坑的“修复式”安全回答。

- **[Promise_vs_react_query_catch.md](./Promise_vs_react_query_catch.md)**  
  Promise.catch 与 React Query 错误模型的本质差异。

- **[Promise_vs_ReactQuery_FallbackEngineering.md](./Promise_vs_ReactQuery_FallbackEngineering.md)**  
  回退 / 降级 / 重试策略为何不该由 Promise 承担。

- **[error_retry_and_fallback_engineering.md](./error_retry_and_fallback_engineering.md)**  
  错误分类、重试、退避与降级的工程级问题空间。

- **[search_race_conditions_playbook.md](./search_race_conditions_playbook.md)**  
  搜索竞态问题的完整工程拆解（Promise / AbortController / React Query / XState / 产品方案）。

- **[understanding_querykey.md](./understanding_querykey.md)**  
  React Query 中 queryKey 的工程语义与并发控制能力。

---

### 一、面试与市场现实认知

- **[bad_interview_questions.md](./bad_interview_questions.md)**  
  记录那些典型的：
  - 无边界
  - 无验证标准
  - 本身就不工程化  
  的面试问题模式。

- **[frontend_senior_market_profile.md](./frontend_senior_market_profile.md)**  
  市场口径中的“高级前端工程师”画像，与真实工程需求之间的落差。

- **[frontend_senior_interview_map.md](./frontend_senior_interview_map.md)**  
  常见高级前端面试问题的真实意图与落点。

- **[frontend_senior_survival_guide.md](./frontend_senior_survival_guide.md)**  
  在高复杂度、低话语权环境中的工程生存策略。

---

### 三、React 工程边界问题

用于区分 React 行为本身、工程实践边界，以及面试中的安全表达。

- **[react_strict_mode_effects.md](./react_strict_mode_effects.md)**  
  Strict Mode 在开发环境下的真实行为。

- **[strict_mode_interview_safe_answers.md](./strict_mode_interview_safe_answers.md)**  
  面试中关于 Strict Mode 的安全、收敛型回答方式。

- **[setState_render_boundary.md](./setState_render_boundary.md)**  
  状态更新与渲染职责的边界说明。

---

### 四、Hooks：工程使用 vs 面试噪音

每个 Hook 都从三个层面拆解：

1. 真实工程使用场景  
2. 执行模型 / 心智模型  
3. 面试安全表达  

- **[useEffect_execution_model.md](./useEffect_execution_model.md)**  
- **[useEffect_interview_safe_answers.md](./useEffect_interview_safe_answers.md)**

- **[useLayoutEffect_real_world_patterns.md](./useLayoutEffect_real_world_patterns.md)**  
- **[useLayoutEffect_interview_safe_answers.md](./useLayoutEffect_interview_safe_answers.md)**

- **[useRef_real_world_patterns.md](./useRef_real_world_patterns.md)**  
- **[useRef_interview_safe_answers.md](./useRef_interview_safe_answers.md)**

- **[useCallback_useMemo_interview_safe_answers.md](./useCallback_useMemo_interview_safe_answers.md)**

---

## 这个目录不是做什么的

- 不是 React API 速查表  
- 不是刷题仓库  
- 不是“如何在面试中表现优秀”的技巧总结  
- 不是技术布道材料  

---

## 这个目录是写给谁的

写给这些工程师：

- 做过真实项目  
- 知道复杂度从哪里来  
- 不再相信“API 熟练度 = 工程能力”  
- 希望：
  - 把工程问题讲清楚
  - 把责任边界守住
  - 把精力用在真正有价值的地方  

如果你也有这种感觉：

> “这个问题是对的，但这样问是不对的。”

那这个目录，就是为你写的。

---

## 最后一句

前端工程如今往往承担着：

- 很高的工程复杂度  
- 很低的决策权  
- 很模糊的责任边界  

这个目录不是为了解决所有问题，  
而是为了 **保持清醒、稳定、可持续地工作**。

不靠硬扛。  
不靠燃烧。  
而是靠**清晰的工程认知**。
