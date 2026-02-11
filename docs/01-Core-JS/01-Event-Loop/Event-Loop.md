---
title: "Event Loop: microtask vs macrotask"
category: "Core JS"
tags: ["frontend", "event-loop", "async"]
difficulty: "easy"
status: "learning"
created: 2026-02-10
source: ["https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop"]
---

# Event Loop: microtask vs macrotask

## TL;DR / 一句话总结
- EN: Promises schedule microtasks that run after the current call stack, before the next macrotask (e.g., timers).
- CN: Promise 的回调是**微任务**：当前同步栈清空后立刻执行，优先级高于定时器等**宏任务**。

## Interview answer (60s) / 面试 60 秒回答
- EN: JavaScript runs on a single call stack. The event loop picks a macrotask (like a timer or IO callback), runs it to completion, then drains the microtask queue (Promise reactions, queueMicrotask, MutationObserver). Only after microtasks are drained does it move to the next macrotask. That’s why `Promise.then` runs before `setTimeout(..., 0)`.
- CN：JS 单线程执行，同步代码跑在 call stack 上。事件循环每次取一个宏任务执行完，然后**清空微任务队列**（Promise.then/catch/finally、queueMicrotask、MutationObserver 等），清空后才进入下一个宏任务，所以 `Promise.then` 通常会比 `setTimeout(...,0)` 先执行。

## Code snippet / 可手写代码
```ts
console.log('A');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve().then(() => console.log('promise'));

console.log('B');
// A
// B
// promise
// timeout
```

## Common pitfalls / 常见坑
- EN: Thinking `await` makes things synchronous (it still yields back to the event loop).
- CN: 误以为 `await` 会阻塞线程；实际上它会把后续代码放进微任务继续跑。

## Follow-ups / 常见追问
- Q: What’s a “floating promise” and why is it bad?
  - A: A promise you neither `await` nor `return` → unhandled rejections / race bugs.
- Q: `requestAnimationFrame` vs `setTimeout`?
  - A: rAF runs before the next paint; timers are macrotasks.
