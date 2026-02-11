---
title: "this binding rules"
category: "Core JS"
tags: ["frontend", "this"]
difficulty: "easy"
status: "learning"
created: 2026-02-10
source: ["https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this"]
---

# this binding rules

## TL;DR / 一句话总结
- EN: In JS, `this` is determined by **how a function is called**, except arrow functions which capture `this` lexically.
- CN: `this` 取决于**调用方式**；箭头函数没有自己的 this，会捕获外层 this。

## Interview answer (60s) / 面试 60 秒回答
- EN: There are common binding rules: (1) `new` binding: `this` is the new instance. (2) explicit binding: `call/apply/bind` sets `this`. (3) implicit binding: `obj.fn()` makes `this=obj`. (4) default binding: plain `fn()` uses `undefined` in strict mode (or global in sloppy mode). Arrow functions don’t have their own `this`; they close over the surrounding `this`.
- CN：常见四条：① new 调用：this 指向新实例；② call/apply/bind 显式绑定；③ obj.fn() 隐式绑定 this=obj；④ 直接 fn() 默认绑定：严格模式是 undefined。箭头函数无 this，使用外层 this。

## Code snippet / 可手写代码
```ts
'use strict';

function f() { return this; }
const obj = { x: 1, f };

console.log(obj.f() === obj); // true (implicit)
console.log(f() === undefined); // true (default in strict)

const g = f.bind(obj);
console.log(g() === obj); // true (explicit)

const arrow = () => this;
```

## Common pitfalls / 常见坑
- CN: 把方法解构出来调用会丢 this：`const { f } = obj; f();`
- EN: `this` is lost when you pass methods as callbacks unless you bind.

## Follow-ups / 常见追问
- `bind` 和 `call/apply` 区别？（bind 返回新函数；call/apply 立即调用）
- React 里为什么 class 组件要 bind？（方法作为回调时 this 丢失）
