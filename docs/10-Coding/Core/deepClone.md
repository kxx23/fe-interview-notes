---
title: "Implement deepClone"
category: "Core"
difficulty: "medium"
patterns: ["recursion", "hash map"]
created: 2026-02-10
---

# Implement deepClone

## Notes
Interview scope: handle objects/arrays + circular references. Mention that full spec clone is complex (Date/RegExp/Map/Set/TypedArray).

## Solution (TS)
```ts
export function deepClone<T>(obj: T, seen = new Map<any, any>()): T {
  if (obj === null || typeof obj !== 'object') return obj;
  if (seen.has(obj)) return seen.get(obj);

  const out: any = Array.isArray(obj) ? [] : {};
  seen.set(obj, out);

  for (const key of Object.keys(obj as any)) {
    out[key] = deepClone((obj as any)[key], seen);
  }
  return out;
}
```

## Follow-ups
- preserve prototype? property descriptors?
- handle Map/Set/Date?
