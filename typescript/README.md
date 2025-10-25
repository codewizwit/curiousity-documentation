# TypeScript Deep Dives

_Making sense of types, one footgun at a time._

## Overview

TypeScript adds static typing to JavaScript, catching errors at compile time instead of runtime. But with great types comes great confusion, especially around concepts like `null` vs `undefined`, asynchronous code patterns, and type narrowing.

This section breaks down the tricky parts with practical examples and metaphors that stick.

---

## Topics

### [null vs undefined: The Toilet Paper Metaphor](./null-vs-undefined/README.md)

Understanding the difference between `null` and `undefined` through the universal experience of toilet paper availability.

**Key concepts:**
- When to use `null` vs `undefined`
- Type narrowing and guards
- Modern operators (`?.`, `??`, `!`)
- `strictNullChecks` configuration
- Common pitfalls and patterns

**Metaphor:** Toilet paper holders
- `0` = Empty roll (defined but nothing left)
- `undefined` = No roll on holder (never set up)
- `null` = Empty cardboard tube (intentionally empty)

---

### [Async/Await: The Hot Dog Stand](./async-await/README.md)

Understanding asynchronous code with the hot dog stand metaphor.

**Key concepts:**
- Synchronous vs asynchronous execution
- Evolution: callbacks → promises → async/await
- Sequential vs parallel patterns
- Error handling with try/catch
- TypeScript typing for async functions
- Common mistakes and best practices

**Metaphor:** Hot dog stand orders
- Synchronous = Blocking the line while cooking
- Asynchronous = Get a number, wait while doing other things
- Sequential await = Order items one at a time (slow)
- Parallel await = Order everything at once (fast)

---

## Quick Navigation

```
typescript/
├── README.md (you are here)
├── null-vs-undefined/
│   ├── README.md
│   └── null-undefined-meme.png
└── async-await/
    └── README.md
```

---

## Coming Soon

Topics I'm planning to explore:

- **Type Guards & Narrowing** - Runtime type checking patterns
- **Generics** - Reusable type-safe functions and components
- **Union & Intersection Types** - Combining types effectively
- **Utility Types** - Partial, Pick, Omit, and friends
- **Type Inference** - Letting TypeScript do the work
- **`any` vs `unknown`** - When to use each (and when to avoid)

---

## Resources

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [Type Challenges](https://github.com/type-challenges/type-challenges)
- [Total TypeScript](https://www.totaltypescript.com/)

---

_Understanding TypeScript: One metaphor at a time._ 🌭

← [Back to Home](../README.md) | [null vs undefined](./null-vs-undefined/README.md) | [async/await](./async-await/README.md)
