# [FE卡] Event loop: microtask vs macrotask

## 中文60秒回答（3-6句，面试可直接念）
JavaScript 在浏览器里是单线程执行的：同步代码先跑完，异步回调靠事件循环排队执行。一次“tick”里，先执行一个 macrotask（比如整段 script、setTimeout 回调、I/O），执行过程中产生的 microtask（主要是 Promise.then / queueMicrotask）会被放进微任务队列。当前 macrotask 结束后，事件循环会“清空”所有 microtask（一直跑到队列为空），然后才会进入下一次 macrotask。直觉上：microtask 像“立刻插队的收尾工作”，macrotask 像“下一轮大任务”。这也解释了为什么 Promise.then 通常会比 setTimeout(…,0) 更早执行。

## EN 60s answer (3-6 sentences, ready to say)
JavaScript runs on a single thread, so async callbacks are coordinated by the event loop. In each tick, the engine executes one macrotask (e.g., a script chunk, a setTimeout callback, I/O), and during that execution it may enqueue microtasks (mainly Promise reactions via `.then/.catch/.finally` or `queueMicrotask`). After the current macrotask finishes, the event loop drains the microtask queue completely before moving on to the next macrotask. Intuitively, microtasks are “immediate post-work” that runs before the next big task. That’s why `Promise.resolve().then(...)` usually runs before `setTimeout(..., 0)`.

## 现在看不懂就看这里（直觉解释/类比/最小例子/常见错误）
- 类比：macrotask = 一节课；microtask = 下课前必须做完的收尾（擦黑板/点名）。你不能开始下一节课（下一个 macrotask）之前把收尾做完（drain microtasks）。
- 最小例子（预测输出顺序）：
  ```js
  console.log('A');
  setTimeout(() => console.log('T'), 0);
  Promise.resolve().then(() => console.log('P'));
  console.log('B');
  // A B P T
  ```
- 常见错误：以为“setTimeout 0 就是立刻”。其实它是“下一轮 macrotask”，还要等当前 macrotask 结束 + microtasks 清空。

## Pitfalls（至少5条，含“为什么错”）
1) **把 Promise.then 当成“和 setTimeout 一样的异步”**：错在优先级不同；microtask 会在当前 macrotask 结束后立刻清空，常常更早执行。
2) **认为每次 tick 只执行一个 microtask**：错在“drain”语义；事件循环会一直跑 microtask 直到队列为空，可能导致长时间占用线程。
3) **用无限 Promise 链制造“饿死”UI/定时器**：错在 microtask 会持续插队，导致渲染/输入/timeout 迟迟得不到机会（starvation）。
4) **在浏览器里把渲染时机和 microtask 顺序混为一谈**：错在渲染通常发生在 macrotask 之间（并且受多种条件影响）；microtask 会先清空，可能推迟下一次渲染。
5) **Node.js 里照搬浏览器结论**：错在 Node 还有 `process.nextTick`、不同阶段（timers/poll/check 等），细节不同；面试要先说明“我先讲浏览器模型，Node 细节略有差异”。
6) **以为 async/await 会创建新的线程**：错在 await 只是把后续代码拆成 microtask（Promise continuation），仍在同一线程由事件循环调度。

## Handwrite（1题，20-40行 TS/JS，带简短注释）
题目：实现一个 `defer(fn)`：优先用 microtask（queueMicrotask/Promise），否则退化到 macrotask（setTimeout）。并写一个 demo 证明顺序。

```ts
// Prefer microtask to run "right after current call stack".
export function defer(fn: () => void): void {
  if (typeof queueMicrotask === 'function') {
    queueMicrotask(fn);
    return;
  }

  // Fallback: Promise microtask (works in most modern JS runtimes)
  if (typeof Promise !== 'undefined') {
    Promise.resolve().then(fn);
    return;
  }

  // Last resort: macrotask
  setTimeout(fn, 0);
}

// Demo: predict the output order
console.log('sync-1');
setTimeout(() => console.log('timeout'), 0); // macrotask

defer(() => console.log('defer'));          // microtask (usually)
Promise.resolve().then(() => console.log('promise'));
console.log('sync-2');

// Expected in browsers: sync-1, sync-2, defer, promise, timeout
// (microtasks drain before the next macrotask)
```

## Follow-ups（面试官追问 + 你怎么答，至少5条）
1) **Q: microtask 里再创建 microtask 会怎样？**
   A: 会继续进 microtask 队列；因为会 drain 到空，所以会在同一轮里继续执行，直到队列真的为空。
2) **Q: setTimeout(…,0) 为什么不是 0ms 就执行？**
   A: 它只是“最早下一轮 timers 阶段”；还要等当前任务结束、微任务清空、以及最小延迟/调度开销。
3) **Q: async/await 对事件循环的影响？**
   A: `await` 之后的代码本质是 Promise continuation，通常以 microtask 形式恢复执行；所以常见顺序是 await 后会在当前 macrotask 结束后、下一个 macrotask 前执行。
4) **Q: requestAnimationFrame 属于哪类？**
   A: 它是浏览器渲染前的回调，通常在下一次绘制前触发；它不是标准的 microtask/macrotask 二分法里的一个简单成员，但可理解为“渲染节拍”相关的回调。
5) **Q: 如何避免 microtask 导致的卡顿/饿死？**
   A: 不要无界递归/长链 microtask；必要时把工作切片，让出控制权（例如用 `setTimeout`/`MessageChannel`/`scheduler.postTask` 等把部分工作放到后续 macrotask）。
6) **Q: 怎么解释浏览器和 Node 的差异而不跑题？**
   A: 先给出浏览器主流模型；补一句“Node 有更多阶段以及 nextTick 更高优先级”，如果需要我可以展开。

## 30分钟怎么练（一个最小流程：读→复述→手写→自测）
1) **读（5min）**：记住一句话：*每个 macrotask 结束后会 drain microtasks*。
2) **复述（5min）**：用“上课/收尾”类比，把 microtask vs macrotask 说成 4 句中文 + 4 句英文。
3) **手写（10min）**：不看卡片写出 `defer(fn)`（含 microtask 优先 + fallback）和 demo。
4) **自测（10min）**：自己出 3 段代码（混合 console.log / Promise.then / setTimeout），先写预测顺序，再跑一遍验证；错了就回到“drain microtasks”这条规则。
