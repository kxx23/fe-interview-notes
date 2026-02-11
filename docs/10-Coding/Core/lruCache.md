---
title: "Implement LRU Cache"
category: "Core"
difficulty: "medium"
patterns: ["Map", "design"]
created: 2026-02-10
---

# Implement LRU Cache

## Notes
For interview: simplest JS solution uses `Map` insertion order.

## Solution (TS)
```ts
export class LRUCache<K, V> {
  private map = new Map<K, V>();
  constructor(private cap: number) {}

  get(key: K): V | undefined {
    if (!this.map.has(key)) return undefined;
    const val = this.map.get(key)!;
    this.map.delete(key);
    this.map.set(key, val);
    return val;
  }

  set(key: K, val: V) {
    if (this.map.has(key)) this.map.delete(key);
    this.map.set(key, val);
    if (this.map.size > this.cap) {
      const firstKey = this.map.keys().next().value;
      this.map.delete(firstKey);
    }
  }
}
```

## Follow-ups
- implement with doubly linked list + hashmap
- TTL support
