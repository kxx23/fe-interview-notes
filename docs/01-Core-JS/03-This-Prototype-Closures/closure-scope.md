---
title: "Scope & Closures"
category: "Core JS"
tags: ["frontend", "closure", "scope"]
difficulty: "easy"
status: "learning"
created: 2026-02-10
source: [
  "https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures",
  "https://developer.mozilla.org/en-US/docs/Glossary/Scope"
]
---

# Scope & Closures（作用域与闭包）

## TL;DR / 一句话总结
- EN: A closure is when a function “remembers” variables from its lexical scope even after the outer function has returned.
- CN: 闭包就是函数“记住”其**词法作用域**里的变量；外层函数返回后，这些变量仍可被访问。

## Interview answer (60s) / 面试 60 秒回答
- EN: JavaScript uses lexical scoping: the scope chain is determined by where code is written. A closure happens when an inner function captures variables from its outer scope. This enables patterns like factories and private state. The tradeoff is memory/lifecycle: captured variables can keep objects alive longer, so you must clean up references (e.g., remove event listeners) to avoid leaks.
- CN：JS 是**词法作用域**（写在哪决定作用域链）。当内层函数引用外层变量时，就形成闭包。它常用于工厂函数、私有状态、回调等。代价是生命周期：被捕获的变量可能让对象更久不释放，所以要注意清理引用（比如移除事件监听）避免泄漏。

## Key points / 关键点
- EN:
  - Lexical scope ≠ dynamic scope.
  - Captured variables are by reference (shared), not by value copy.
- CN:
  - 词法作用域：作用域链在编写时就确定。
  - 捕获是“引用共享”，多个闭包可能共享同一个变量。

## Common pitfalls / 常见坑
- CN: `for (var i...)` + 异步回调导致 i 都变成最终值；用 `let` 或 IIFE。
- EN: Keeping closures around in long-lived handlers causes leaks if not cleaned up.

## Code snippet / 可手写代码
```ts
function makeCounter() {
  let n = 0;
  return {
    inc() { n++; return n; },
    get() { return n; },
  };
}

const c = makeCounter();
c.inc(); // 1
c.inc(); // 2
c.get(); // 2
```

### 经典坑：var in loop
```ts
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// 3 3 3

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);
}
// 0 1 2
```

## Follow-ups / 常见追问
- Q: 为什么 `let` 能解决 loop 问题？
  - A: `let` 每次迭代创建新的块级绑定。
- Q: 闭包一定会内存泄漏吗？
  - A: 不一定；泄漏是“该释放未释放”。闭包只是延长生命周期，关键在于你是否清理引用。
