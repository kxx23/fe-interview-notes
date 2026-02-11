# [FE卡] Event loop: microtask vs macrotask

## 中文60秒回答（3-6句，面试可直接念）
JS 在浏览器/Node 里是单线程执行，但通过事件循环把“同步代码、微任务、宏任务”按批次调度。每一轮 loop 会先跑完当前调用栈的同步代码，然后**清空 microtask 队列**（如 Promise.then/catch/finally、queueMicrotask、MutationObserver），再取一个 macrotask（如 setTimeout/setInterval、I/O、UI 事件、MessageChannel）执行。因为 microtask 会在渲染/下一次 macrotask 之前被全部执行，所以用 Promise.then 往往比 setTimeout(0) 更“快”。如果不断递归塞 microtask，会饿死渲染和 I/O，导致页面卡死。

## EN 60s answer (3-6 sentences, ready to say)
JavaScript runs on a single thread, and the event loop schedules work in phases: sync code, microtasks, then macrotasks. In each turn, the engine runs the current call stack, then **drains the microtask queue** (Promise callbacks, `queueMicrotask`, MutationObserver), and only then picks the next macrotask (timers, I/O, UI events, MessageChannel). That’s why `Promise.then(...)` typically runs before `setTimeout(..., 0)`. If you keep enqueueing microtasks in a loop, you can starve rendering and I/O and make the app feel frozen.

## 现在看不懂就看这里（直觉解释/类比/最小例子/常见错误）
- 类比：
  - **macrotask = 下一封大邮件**（一封一封处理）
  - **microtask = 这封邮件的“立刻要做的小便签”**（处理完这封邮件前必须把便签全做完）
- 最小例子（输出顺序）：
  ```js
  console.log('A');
  setTimeout(() => console.log('timeout'), 0);
  Promise.resolve().then(() => console.log('then'));
  console.log('B');
  // A B then timeout
  ```
- 常见误解：
  - “setTimeout(0) 会立刻执行”——不会，它只是把回调放到未来的 macrotask 队列里，最早也要等本轮同步 + microtask 清空之后。

## Pitfalls（至少5条，含“为什么错”）
1) **把 `then` 当成同步**：
   - 错在哪：`then` 回调永远异步（进 microtask）。
   - 为什么错：你会在逻辑上误判变量状态/日志顺序，产生竞态。
2) **用 `setTimeout(0)` 当“比 Promise 更快的 defer”**：
   - 错在哪：`setTimeout` 是 macrotask，通常晚于 microtask。
   - 为什么错：想“尽快执行”时反而延后，UI/状态更新顺序会乱。
3) **递归塞 microtask（microtask 泄洪）**：
   - 错在哪：不停 `queueMicrotask`/`Promise.then` 让 microtask 永远清不完。
   - 为什么错：渲染与输入事件要等 microtask 清空，页面会卡、掉帧、无法响应。
4) **混淆不同环境的细节（浏览器 vs Node）**：
   - 错在哪：Node 还有 `process.nextTick`、不同阶段的 I/O/timers；浏览器有渲染时机。
   - 为什么错：你背“固定顺序”会在另一个环境翻车。回答要讲“原则 + 常见实现”，别编绝对规则。
5) **在一个任务里做重 CPU 计算，期待事件循环救场**：
   - 错在哪：事件循环只能在你把控制权交回去后才调度。
   - 为什么错：长同步循环会阻塞一切，micro/macro 都没机会跑。
6) **以为 await 会“开新线程”**：
   - 错在哪：`await` 只是把后续代码拆到 microtask 继续执行。
   - 为什么错：错误处理/执行顺序会被你理解错（尤其 try/catch 与 finally）。

## Handwrite（1题，20-40行 TS/JS，带简短注释）
题目：实现一个 `schedule(fn)`，让 fn 在“本轮同步结束后、下一次 macrotask 之前”执行；并演示它比 `setTimeout` 更早。

```ts
// schedule: prefer microtask (Promise/queueMicrotask), fallback to setTimeout.
// Goal: run after current call stack, before next macrotask.

type Task = () => void;

export function schedule(fn: Task): void {
  if (typeof queueMicrotask === 'function') {
    queueMicrotask(fn);
    return;
  }
  // Promise microtask fallback
  if (typeof Promise !== 'undefined' && Promise.resolve) {
    Promise.resolve().then(fn);
    return;
  }
  // Last resort: macrotask
  setTimeout(fn, 0);
}

// Demo ordering
console.log('sync-1');
setTimeout(() => console.log('timeout'), 0);
schedule(() => console.log('scheduled')); // should print before timeout
console.log('sync-2');
```

## Follow-ups（面试官追问 + 你怎么答，至少5条）
1) Q：`await` 之后的代码属于 microtask 还是 macrotask？
   - A：`await` 等价于把后续放到 Promise 链上继续执行，通常进入 **microtask** 队列。
2) Q：为什么一直递归 `Promise.then` 会让页面不渲染？
   - A：因为每轮都会在进入下一次 macrotask/渲染前清空 microtask；如果 microtask 不断增长，渲染机会被饿死。
3) Q：`requestAnimationFrame` 在哪一层？
   - A：它不是典型 micro/macro 分类里的普通任务；它会在浏览器准备下一帧绘制前回调，常用于与渲染对齐。
4) Q：如何把很重的任务“拆开”避免阻塞？
   - A：把工作切片：每片做一点然后 `setTimeout`/`MessageChannel`/`requestIdleCallback` 让出控制权；或用 Web Worker。
5) Q：Node 里还有什么更“优先”的队列？
   - A：Node 有 `process.nextTick`（优先级很高，甚至可能饿死其他队列）以及不同阶段的 timers/I/O；但我会先讲清 microtask vs macrotask 的大原则。
6) Q：`Promise.resolve().then` 和 `queueMicrotask` 有什么区别？
   - A：都入 microtask；`queueMicrotask` 更语义化且不会受 Promise 链上的异常处理细节影响（但都要注意不要滥用导致饥饿）。

## 30分钟怎么练（一个最小流程：读→复述→手写→自测）
1) 读（5min）：把“每轮：sync → drain microtasks → next macrotask”默写成一句话。
2) 复述（5min）：用中文 60 秒回答录音/自讲一遍；再用英文 60 秒讲一遍。
3) 手写（10min）：不看答案手写 `schedule(fn)` + demo，跑出输出顺序。
4) 自测（10min）：自己再造 2 个小例子：
   - 例1：`async function` + `await` + `then` 混合，写出你预测的输出顺序再验证。
   - 例2：写一个递归 microtask，观察 UI 卡顿（注意别在真项目里干）。
