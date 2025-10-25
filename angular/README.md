# Angular Modern Reactivity

_Understanding Angular's new signal-based reactivity and why it's a game changer._

## Overview

Angular 16+ introduced a massive shift in how we handle reactivity: **signals**. For years, Angular developers relied heavily on RxJS observables, `BehaviorSubject`, and lifecycle hooks to manage state and side effects. It worked, but it was complex, verbose, and had a steep learning curve.

The new signal-based reactivity system is Angular's answer to making reactive programming simpler, more intuitive, and performant by default. Think of it as Angular saying: "What if state updates were just... obvious?"

---

## The Whiteboard Metaphor

Imagine you're working in an office with a team:

**The Old Way (RxJS Observables):**
- Everyone passes paper memos through tubes between desks (streams)
- You have to subscribe to each tube to get updates
- You need to remember to unsubscribe or papers pile up (memory leaks)
- Combining info from multiple tubes requires complex plumbing (`combineLatest`, `merge`, etc.)
- It works, but it's a lot of ceremony

**The New Way (Signals):**
- There's a shared whiteboard in the center of the room
- Anyone can write a sticky note on the board
- Everyone can see changes immediately (no subscription needed)
- Computed notes automatically update when their source notes change
- Effects are like a janitor who automatically cleans up when notes change

---

## What's New in Angular Signals

### Three Core Primitives

**1. `signal()` - Writable State**
- Simple, mutable state container
- Replaces `BehaviorSubject` in most cases
- No subscription needed to read values

**2. `computed()` - Derived State**
- Automatically recalculates when dependencies change
- Replaces `combineLatest` and manual derived state patterns
- Memoized by default (only recalculates when needed)

**3. `effect()` - Side Effects**
- Runs automatically when signals it reads change
- Replaces `subscribe()` and many lifecycle hook patterns
- Automatically tracks dependencies

---

## Topics

### [Quick Reference Cheatsheet](./cheatsheet/README.md)

Bite-sized quick reference for modern Angular basics: new control flow syntax, standalone components, dependency injection, routing, and common patterns.

**What you'll learn:**
- Modern control flow (`@if`, `@for`, `@switch`)
- Standalone components and `inject()`
- Signal-based inputs and outputs
- Two-way binding and the "banana-in-a-box" syntax
- HTTP requests and common patterns
- Migration from old Angular to modern Angular

**Key insight:** If you're learning Angular now, start here. This is the modern way.

---

### [Observables: Async Data Streams](./observables/README.md)

Understanding RxJS Observables, the async powerhouse behind Angular's HTTP, Router, and reactive Forms.

**What you'll learn:**
- What Observables are (the newsletter metaphor)
- Common RxJS operators (`map`, `filter`, `switchMap`, `debounceTime`)
- The AsyncPipe and avoiding memory leaks
- When to use Observables vs Signals
- Real-world patterns (search with debounce, polling, retries)

**Key insight:** Observables aren't going away. Use them for async operations, Signals for state.

---

### [Signals: The New State Primitive](./signals/README.md)

Compare `signal()` to the old `BehaviorSubject` pattern.

**What you'll learn:**
- Creating and updating writable signals
- Reading signal values in templates and components
- Why signals are simpler than observables for local state
- When to still use observables
- Migration patterns from RxJS to signals

**Key insight:** If you're using `BehaviorSubject` just to hold state and `.next()` to update it, you probably want a signal instead.

---

### [Computed: Derived State Made Easy](./computed/README.md)

Compare `computed()` to RxJS operators like `combineLatest` and manual derived state.

**What you'll learn:**
- Creating computed signals that depend on other signals
- Automatic dependency tracking (no manual wiring!)
- Memoization and performance benefits
- Chaining computed signals
- Migration from `combineLatest` patterns

**Key insight:** Computed signals are like Excel formulas. They automatically update when their inputs change, and they don't recalculate unnecessarily.

---

### [Effects: Side Effects Without the Ceremony](./effect/README.md)

Compare `effect()` to `subscribe()` and lifecycle hooks like `ngOnInit`.

**What you'll learn:**
- Running side effects when signals change
- Automatic cleanup (no more manual unsubscribe!)
- Effect timing and execution guarantees
- When to use effects vs. computed
- Migration from `subscribe()` and lifecycle hook patterns

**Key insight:** Effects are for actions (logging, API calls, localStorage), while computed is for calculations. If it returns a value, use computed. If it does something, use effect.

---

## Quick Comparison Table

| Task | Old Way (RxJS) | New Way (Signals) |
|------|----------------|-------------------|
| **Hold state** | `BehaviorSubject` + `.next()` | `signal()` + `.set()` or `.update()` |
| **Read state** | `.subscribe()` or `async` pipe | `mySignal()` or `mySignal()` in template |
| **Derived state** | `combineLatest()` + manual logic | `computed(() => ...)` |
| **Side effects** | `.subscribe()` + `takeUntil()` | `effect(() => ...)` |
| **Cleanup** | Manual `unsubscribe()` or `takeUntil()` | Automatic |

---

## Should You Migrate Everything to Signals?

**Short answer:** No, not yet. But start using signals for new features.

**When to use signals:**
- Local component state (replace `BehaviorSubject`)
- Derived state (replace `combineLatest`)
- Simple side effects (replace basic `subscribe()` patterns)

**When to stick with RxJS:**
- Complex async operations (HTTP, WebSockets, timers)
- Event streams with operators like `debounceTime`, `throttle`, `switchMap`
- Libraries that return observables (Angular HttpClient, etc.)

**The sweet spot:** Use signals for state, RxJS for async operations. Angular provides `toSignal()` and `toObservable()` to bridge between them.

---

## Migration Strategy

```typescript
// 1. Start with new components - use signals from the start
@Component({...})
export class NewComponent {
  count = signal(0);  // ✅ Signal
  double = computed(() => this.count() * 2);  // ✅ Computed
}

// 2. Gradually convert existing simple state
@Component({...})
export class LegacyComponent {
  // ❌ Old way
  // count$ = new BehaviorSubject<number>(0);

  // ✅ New way
  count = signal(0);
}

// 3. Keep RxJS for complex async operations
@Component({...})
export class HybridComponent {
  searchTerm = signal('');  // ✅ Signal for state

  // ✅ Keep RxJS for complex async + operators
  results$ = toObservable(this.searchTerm).pipe(
    debounceTime(300),
    switchMap(term => this.api.search(term))
  );

  // ✅ Convert back to signal for template
  results = toSignal(this.results$, { initialValue: [] });
}
```

---

## Quick Navigation

```
angular/
├── README.md (you are here)
├── cheatsheet/
│   └── README.md
├── observables/
│   └── README.md
├── signals/
│   └── README.md
├── computed/
│   └── README.md
└── effect/
    └── README.md
```

---

## Resources

- [Angular Signals Documentation](https://angular.io/guide/signals)
- [Angular Signals RFC](https://github.com/angular/angular/discussions/49090)
- [RxJS Interop](https://angular.io/guide/rxjs-interop) - `toSignal()` and `toObservable()`
- [Angular Blog: Signal APIs](https://blog.angular.io/angular-v16-is-here-4d7a28ec680d)

---

_Making reactivity obvious: From streams and subscriptions to signals and sticky notes._ 🌭

← [Back to Home](../README.md) | [Cheatsheet](./cheatsheet/README.md) | [Observables](./observables/README.md) | [Signals](./signals/README.md) | [Computed](./computed/README.md) | [Effect](./effect/README.md)
