---
title: "Implement EventEmitter"
category: "Core"
difficulty: "easy"
patterns: ["map", "pub-sub"]
created: 2026-02-10
---

# Implement EventEmitter

## Solution (TS)
```ts
type Handler<T = any> = (payload: T) => void;

export class EventEmitter {
  private map = new Map<string, Set<Handler>>();

  on(event: string, handler: Handler) {
    if (!this.map.has(event)) this.map.set(event, new Set());
    this.map.get(event)!.add(handler);
    return () => this.off(event, handler);
  }

  off(event: string, handler: Handler) {
    this.map.get(event)?.delete(handler);
  }

  emit(event: string, payload?: any) {
    for (const h of this.map.get(event) ?? []) h(payload);
  }
}
```

## Follow-ups
- once()
- listener order
- memory leak prevention
