---
title: "Implement throttle"
category: "Async"
difficulty: "easy"
patterns: ["timer", "closure"]
created: 2026-02-10
---

# Implement throttle

## Problem
Return a throttled function that invokes `fn` at most once per `wait` ms.

## Solution (TS)
```ts
export function throttle<T extends (...args: any[]) => any>(fn: T, wait: number) {
  let last = 0;
  return function (this: any, ...args: Parameters<T>) {
    const now = Date.now();
    if (now - last >= wait) {
      last = now;
      return fn.apply(this, args);
    }
  };
}
```

## Follow-ups
- trailing execution
- using timestamps vs timer
