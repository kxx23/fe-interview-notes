---
title: "Implement promisePool (limit concurrency)"
category: "Async"
difficulty: "medium"
patterns: ["concurrency control", "promise", "queue"]
created: 2026-02-10
---

# Implement promisePool (limit concurrency)

## Problem (EN)
Given an array of functions `tasks` where each returns a Promise, and a number `k`, run at most `k` tasks concurrently. Resolve when all tasks finish.

## 题目（中文）
给一组返回 Promise 的任务函数 `tasks`，以及并发数 `k`，要求同一时间最多跑 `k` 个任务；全部完成后 resolve。

## Approach
- Idea: create `k` workers; each worker pulls the next task index until exhausted.
- Complexity: scheduling O(n), in-flight O(k).

## Solution (TS)
```ts
export async function promisePool<T>(tasks: Array<() => Promise<T>>, k: number): Promise<T[]> {
  if (k <= 0) throw new Error('k must be >= 1');
  const results: T[] = new Array(tasks.length);
  let next = 0;

  async function worker() {
    while (true) {
      const i = next++;
      if (i >= tasks.length) return;
      results[i] = await tasks[i]();
    }
  }

  const workers = Array.from({ length: Math.min(k, tasks.length) }, () => worker());
  await Promise.all(workers);
  return results;
}
```

## Follow-ups
- Fail fast vs collect all errors?
- Support cancellation (AbortController)?
- Preserve result order (current does)?
