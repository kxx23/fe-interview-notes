---
title: "Type Coercion (== vs ===, toString/toNumber)"
category: "Core JS"
tags: ["frontend", "types", "coercion"]
difficulty: "easy"
status: "learning"
created: 2026-02-10
source: [
  "https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness",
  "https://developer.mozilla.org/en-US/docs/Glossary/Type_coercion"
]
---

# Type Coercion（类型转换）

## TL;DR / 一句话总结
- EN: Prefer `===`; `==` triggers coercion rules that can surprise you (e.g., `'' == 0` is true).
- CN: 优先用 `===`；`==` 会触发隐式类型转换，很多边界很反直觉（如 `'' == 0` 为 true）。

## Interview answer (60s) / 面试 60 秒回答
- EN: JavaScript has implicit coercion, mainly via `ToPrimitive`, `ToNumber`, and `ToString`. `===` compares without coercion (except NaN semantics), while `==` applies coercion rules (null/undefined special case, number/string conversion, boolean to number, objects to primitives). In interviews, I say: use `===` by default; understand a few key corner cases for debugging.
- CN：JS 有隐式转换，核心是 `ToPrimitive/ToNumber/ToString`。`===` 不做隐式转换（NaN 例外），`==` 会按规则转换（null/undefined 特判、字符串/数字互转、boolean 转数字、对象先转原始值）。工程里默认用 `===`，但要知道常见坑便于排查 bug。

## Key rules (must know) / 必背规则
- `null == undefined` is true; but neither equals anything else.
- `NaN` is not equal to anything, including itself (`Number.isNaN`).
- `false` in `==` tends to coerce to 0.
- Objects in `==` are coerced via `valueOf`/`toString` (ToPrimitive).

## Common gotchas / 常见坑（背这 8 条就够了）
```ts
'' == 0          // true
'0' == 0         // true
0 == false       // true
'\n\t' == 0      // true
null == undefined// true
null == 0        // false
[] == 0          // true   ([] -> '' -> 0)
[1] == 1         // true   ([1] -> '1' -> 1)
```

## Prefer / 建议
- Use `===` unless you explicitly want `null == undefined` style checks.
- For NaN: `Number.isNaN(x)`.

## Follow-ups / 常见追问
- Q: `Object.is` vs `===`?
  - A: `Object.is(NaN, NaN)` is true; `Object.is(+0, -0)` is false; `===` is opposite for those edge cases.
