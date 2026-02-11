---
title: "Promise combinators: all / allSettled / race / any"
category: "Core JS"
tags: ["frontend", "async", "promise"]
difficulty: "easy"
status: "learning"
created: 2026-02-10
source: ["https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise"]
---

# Promise combinators: all / allSettled / race / any

## Spaced Repetition
- Created: 2026-02-10
- Last reviewed: 2026-02-10
- Next review: 2026-02-11
- Score (0-3): 1

Checklist:
- [ ] R1 (+1d)
- [ ] R2 (+3d)
- [ ] R3 (+7d)
- [ ] R4 (+14d)
- [ ] R5 (+30d)
- [ ] R6 (+60d)

## TL;DR / 一句话总结
- EN: Promise combinators coordinate multiple async operations with different failure/settle semantics.
- CN: 这四个 API 用来“组合多个异步任务”，区别核心在于**成功条件/失败条件/是否等待全部结束**。

## Interview answer (60s) / 面试 60 秒回答
- EN: `all` is fail-fast and fulfills only if all fulfill (keeps input order). `allSettled` always fulfills with per-task status. `race` settles with the first settled promise (could reject). `any` fulfills on the first fulfillment and rejects only if all reject (AggregateError).
- CN：`all` 要全成功才成功，任一失败就 fail-fast；结果顺序按输入顺序。`allSettled` 等全部结束并返回每个任务的状态报告。`race` 谁先结束用谁（可能先失败）。`any` 取第一个成功，只有全失败才抛 AggregateError。

## Key points / 关键点
- EN:
  - All accept an iterable; non-promises are treated as fulfilled values.
  - No built-in cancellation: others keep running.
- CN:
  - 都接收 iterable；普通值会被当成已 fulfilled。
  - 都不会自动取消其它任务（需要 AbortController/自定义取消协议）。

## Common pitfalls / 常见坑
- EN:
  - Assuming `Promise.all` returns results by completion order (it doesn't).
  - Forgetting to handle rejected entries in `allSettled`.
- CN:
  - 误以为 `all` 按完成顺序返回（实际按输入顺序）。
  - `allSettled` 需要你自己分流处理 rejected。

## Code snippet / 可手写代码
```ts
async function withTimeout<T>(p: Promise<T>, ms: number): Promise<T> {
  const timeout = new Promise<T>((_, reject) =>
    setTimeout(() => reject(new Error('timeout')), ms)
  ) as unknown as Promise<T>;
  return Promise.race([p, timeout]);
}
```

## Follow-ups / 常见追问
- Q: How to cancel remaining requests?
  - A: Use `AbortController` (fetch) or a custom cancellation token.
- Q: `any` vs `race`?
  - A: `race` = first settled (success or failure), `any` = first success.

## Quick comparison
| API | Success | Failure | Wait all? |
|---|---|---|---|
| all | all fulfill | any reject | no (fail-fast) |
| allSettled | always fulfills with report | rarely rejects | yes |
| race | first settled outcome | first settled is reject | no |
| any | first fulfillment | all reject (AggregateError) | no |
