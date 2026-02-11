---
title: "Promises fundamentals"
category: "Core JS"
tags: ["frontend", "async", "event-loop", "promise"]
difficulty: "medium"
status: "learning"
created: 2026-02-10
source: [
  "https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise",
  "https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop"
]
---

# Promises fundamentals

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
- EN: A Promise represents a future value and enables composable async flows via chaining.
- CN: Promise 表示“未来才拿到的值/错误”，用 then/catch/finally 组合异步流程。

## Interview answer (60s) / 面试 60 秒回答
- EN: A promise starts pending and settles once (fulfilled/rejected). `then` returns a new promise so chains are composable; returning a value fulfills the next link, throwing rejects it, and returning a promise adopts its state. Handlers run as microtasks.
- CN：Promise 从 pending 开始，只会 settle 一次（fulfilled/rejected）。`then` 返回新 Promise：返回值→下一个 fulfilled；throw→下一个 rejected；返回 Promise→下一个“接管”它的状态。`then/catch/finally` 回调属于微任务。

## Code snippet / 可手写代码
```ts
Promise.resolve(1)
  .then(x => x + 1)
  .then(x => Promise.resolve(x + 1))
  .then(console.log);
```

## Common pitfalls / 常见坑
- EN: Using `forEach(async ...)` and expecting it to await.
- CN: 用 `forEach(async () => ...)` 以为会串行 await（不会）。用 `for...of` 或 `Promise.all`。
