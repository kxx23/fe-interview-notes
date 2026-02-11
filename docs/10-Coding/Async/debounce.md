---
title: "Implement debounce"
category: "Async"
difficulty: "easy"
patterns: ["timer", "closure"]
created: 2026-02-10
---

# Implement debounce

## Problem
Return a debounced function that delays invoking `fn` until after `wait` ms have elapsed since the last time it was called.

## Solution (TS)
```ts
export function debounce<T extends (...args: any[]) => any>(fn: T, wait: number) {
  let timer: ReturnType<typeof setTimeout> | null = null;
  return function (this: any, ...args: Parameters<T>) {
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), wait);
  };
}
```

## Follow-ups
- leading/trailing options
- cancel/flush
