# Roadmap（KB：中级｜中国公司｜React｜TS=7｜30–60min/天）

目标：6 周建立“**能说清楚 + 能手写 + 有项目故事**”的面试输出。

> 时间策略：
> - 平日 30–60 分钟：优先 **八股口述 + 1 个小手写题**（比刷很多题更有效）
> - 周末如有空再加 1 次 mock（30–45 分钟）

## Week 1 — JS 核心（中级最常考）
- Promise 组合 / async-await
- Event loop（microtask/macrotask）
- this / bind / call / apply
- closure & scope
- 原型链（讲得清楚即可）
- types & coercion（常见坑）

交付：
- 6 篇 topic 笔记（EN/CN 60 秒回答 + 1 个可手写例子）
- 6 个 handwrite 小题（promisePool / debounce / throttle / deepClone / eventEmitter / LRU）

## Week 2 — React 基础原理 + 常见坑（中国面试高频）
- render 触发条件、re-render 原因
- hooks：deps、stale closure、useMemo/useCallback 取舍
- keys / reconciliation
- 状态管理：local vs global、Context 性能坑

交付：
- 4 篇 React 笔记 + 1 份 hooks 坑点 checklist
- 2 个手写：实现 useDebounce/useThrottle hook；实现简版 usePrevious

## Week 3 — 浏览器 + 网络（讲清楚即可，别背太细）
- 渲染流程（layout/paint/composite）
- 缓存（Cache-Control/ETag）
- CORS + cookie/SameSite
- XSS/CSRF/CSP（只要能解释与防法）

交付：
- 4 篇笔记 + 1 张“缓存策略”小抄

## Week 4 — TS 工程化表达（你 TS=7：补齐面试口述）
- narrowing / discriminated unions
- generics（常见写法与约束）
- utility types（Pick/Omit/Record/ReturnType…）

交付：
- 3 篇笔记 + 10 个 TS snippet（可默写）

## Week 5 — 性能（可量化的故事）
- Web Vitals（LCP/CLS/INP）
- bundle 优化：code splitting、按需加载
- 长任务/渲染卡顿定位（Performance panel / React profiler）

交付：
- 2 篇笔记 + 1 份“性能排查流程”

## Week 6 — 项目表达 + Mock 冲刺
- 准备 2 个项目故事（STAR/CARE：中文主，英文备）
- 3 次 mock（coding×2 + React deep dive×1）

交付：
- 2 个 project story one-pager
- 3 份 mock 复盘（改进点<=3条/次）
