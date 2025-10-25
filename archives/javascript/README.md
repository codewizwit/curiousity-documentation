# JavaScript Runtime: When I Finally Understood the Event Loop (2020)

_The moment asynchronous code stopped being magic_

---

## Context

**When:** 2020 (Bootcamp - JavaScript fundamentals & runtime module)
**What I was learning:** How JavaScript actually runs "under the hood" - the event loop, call stack, memory heap, callback queue
**Why it was hard:** JavaScript code looked simple, but understanding HOW it executed asynchronously required seeing the invisible machinery. Why doesn't `setTimeout(fn, 0)` run immediately? How can JavaScript do multiple things when it's single-threaded?
**Where I was:** Building async applications and hitting weird timing bugs I couldn't explain

**The challenge:** Understanding that JavaScript is single-threaded but can handle asynchronous operations through the event loop.

---

## The Notes

These 11 pages capture me learning how JavaScript actually executes code - from data structures and the event loop to advanced concepts like closures, functional programming, and module systems.

### Page 1: Data Structures - Stack, Queue, Heap
![Data Structures](./01-stack-queue-heap-overview.jpeg)

**What's happening here:**
"Module 1 Computer Science"

Notes on stack (LIFO), queue (FIFO), and heap data structures - the foundational concepts that make understanding the JavaScript runtime possible.

**What I notice now:**
The terminology clicked here: the **call stack** is literally a stack data structure. The **callback queue** is literally a queue. The **memory heap** isn't the heap data structure, but rather unstructured memory storage. Understanding data structures made the runtime concrete.

**Starting with the fundamentals - you can't understand the event loop until you understand what a stack and queue actually are.**

---

### Page 2: Big Picture - Event Loop Diagram
![Event Loop Diagram](./02-big-picture-event-loop-diagram.jpeg)

**What's happening here:**
"BIG PICTURE" diagram showing the JavaScript runtime components:
- **MEMORY HEAP** - where objects live
- **CALL STACK** - where function execution happens
- **WEB API** - browser-provided async capabilities (setTimeout, fetch, DOM)
- **CALLBACK QUEUE** - where callbacks wait their turn
- **EVENT LOOP** - the coordinator that checks if call stack is empty and moves callbacks from queue to stack

**What I notice now:**
This diagram was THE breakthrough. I needed to see all the pieces together: JavaScript itself (call stack + memory), the browser APIs that handle async work, and the event loop connecting them.

**The event loop isn't magic - it's a simple loop asking: "Is the call stack empty? Yes? Grab the next callback from the queue."**

---

### Page 3: Call Stack & Memory Heap
![Call Stack Memory](./03-call-stack-memory-heap.jpeg)

**What's happening here:**
Detailed diagrams of the call stack and memory heap structures.

Boxes showing how function calls stack on top of each other, and how objects are stored in the heap.

**What I notice now:**
I was visualizing execution: functions go on the stack (LIFO - last in, first out), run to completion, then pop off. Objects go in the heap. Understanding this made debugging stack traces make sense - the trace IS the call stack at the moment of error.

---

### Page 4: Async Behavior
![Async Behavior](./04-async-behavior.jpeg)

**What's happening here:**
Notes on asynchronous JavaScript behavior - how async operations work, common patterns, and understanding execution flow.

**What I notice now:**
These fundamentals connect: closures work because functions remember their scope (stored in memory heap). Async works because callbacks get queued. Everything fits together once you understand the runtime.

---

### Page 5: setTimeout
![setTimeout](./05-setTimeout.jpeg)

**What's happening here:**
Deep dive into setTimeout - how it works, timing behavior, and why it doesn't always behave as expected.

**What I notice now:**
I was building my mental model systematically: how does setTimeout actually work with the event loop? Why doesn't `setTimeout(fn, 0)` run immediately? Understanding this prevented so many timing bugs.

---

### Page 6: Common Problems & Solutions
![Common Problems Solutions](./06-common-problems-solutions.jpeg)

**What's happening here:**
Notes on common JavaScript problems and their solutions - async pitfalls, scope issues, timing bugs, and how to debug them.

**What I notice now:**
This was me documenting patterns I'd hit in real code. Modules have their own scope. Events propagate in specific ways. Understanding the "gotchas" meant I could write better code from the start.

---

### Page 7: The "this" Keyword & Prototypes
![This Keyword Prototype](./07-this-keyword-prototype.jpeg)

**What's happening here:**
Notes on JavaScript's "this" keyword, prototypes, constructors, and object-oriented patterns in JavaScript.

Diagrams showing the prototype chain and how "this" binding works in different contexts.

**What I notice now:**
**"this" is context, not identity.** Unlike other languages where "this" always means "the current object," JavaScript's "this" depends on HOW the function was called:
- Regular function call: `this` = global object (or undefined in strict mode)
- Method call: `this` = the object before the dot
- Constructor call: `this` = the new object being created
- Arrow function: `this` = inherited from surrounding scope

Understanding prototypes and "this" made object-oriented JavaScript finally make sense. The prototype chain is how JavaScript does inheritance without classes (pre-ES6). Every object has a hidden link to its prototype, creating a lookup chain.

---

### Page 8: Functional Programming vs OOP
![Functional Programming vs OOP](./08-functional-programming-vs-oop.jpeg)

**What's happening here:**
Notes on functional programming concepts compared to object-oriented programming - understanding different programming paradigms in JavaScript.

Exploring how functions can be treated as first-class citizens, immutability, and pure functions versus object-based patterns.

**What I notice now:**
**JavaScript is multi-paradigm by design.** OOP organizes code around objects with state and methods. Functional programming organizes code around pure functions and immutable data. JavaScript supports both, and mixing them intelligently is what makes modern JavaScript powerful. Understanding when to use `.map()` vs a for loop, when to favor immutability vs stateful objects, made me a more flexible developer.

---

### Page 9: Closures
![Closures](./09-closures.jpeg)

**What's happening here:**
Notes on JavaScript closures - how functions can "remember" and access variables from their outer scope even after the outer function has returned.

Understanding lexical scope and how closures enable patterns like private variables and factory functions.

**What I notice now:**
**Closures are about memory, not magic.** When a function is created, it captures a reference to its surrounding scope. This enables powerful patterns: module pattern (private state), callbacks that remember context, and functional programming techniques. Every JavaScript developer uses closures constantly (every event handler, every callback). Understanding them explicitly transformed debugging from "why doesn't this work?" to "oh, the closure captured the wrong variable."

---

### Page 10: Event Delegation
![Event Delegation](./10-event-delegation.jpeg)

**What's happening here:**
Notes on event delegation - attaching a single event listener to a parent element instead of many listeners to individual child elements.

Understanding how events bubble up through the DOM and how to use this for more efficient event handling.

**What I notice now:**
**Event delegation is performance optimization through understanding the DOM.** Instead of attaching 100 click listeners to 100 buttons, attach one listener to their parent and check `event.target` to see which child triggered it. This works because events bubble up from child to parent. Understanding event delegation prevented performance issues in dynamic UIs where elements are constantly added/removed, and made frameworks like React's synthetic event system finally make sense.

---

### Page 11: Modules - Import, Export & Require
![Modules Import Export](./11-modules-import-export.jpeg)

**What's happening here:**
Questions about JavaScript module systems:
- `module.exports` = & `object.3`
- How does `require()` work?
- Difference between `require()` and `import`
- External dependencies and package management

**What I notice now:**
**Modules are how JavaScript scales beyond one file.** CommonJS (`require`/`module.exports`) came first for Node.js. ES6 modules (`import`/`export`) are the modern standard. Understanding both was essential because legacy code uses CommonJS while new code uses ES6 modules. The key insight: modules have their own scope, with no more global namespace pollution. Each file is isolated, explicitly exporting what others can use. This makes code maintainable, testable, and prevents the "everything depends on everything" nightmare.

---

## The Aha Moment

**Page 2: The Big Picture Event Loop Diagram**

The breakthrough came from this visual:

```
CALL STACK → executing code (one thing at a time)
     ↓
   Empty?
     ↓
EVENT LOOP checks → CALLBACK QUEUE has waiting callbacks?
     ↓                       ↓
   Yes                     Move callback to call stack
```

**The key insight: JavaScript is single-threaded, but the browser isn't.**

When you call `setTimeout(fn, 1000)`:
1. **Call stack**: Registers the timer with the Web API, continues executing
2. **Web API**: Counts down 1000ms in the background (browser C++ code, not JavaScript)
3. **After 1000ms**: Callback goes into the callback queue
4. **Event loop**: Waits for call stack to be empty, then moves callback from queue to stack
5. **Call stack**: Finally runs your callback

**Why `setTimeout(fn, 0)` doesn't run immediately:**
Even with 0ms delay, the callback goes to the queue. If the call stack is busy, the callback waits. "0ms" means "as soon as the stack is clear," not "right now."

This explained every weird async timing bug I'd hit!

---

## How I'd Explain This Now

**Then (2020):**
"JavaScript is asynchronous and can do multiple things at once."

**Now (2025):**
"JavaScript itself is single-threaded - it can only do one thing at a time via the call stack. But the browser provides Web APIs (setTimeout, fetch, DOM events) that run outside JavaScript in native browser code.

Think of it like a restaurant:
- **Call stack** = the chef (can only cook one dish at a time)
- **Web APIs** = the oven timer, dishwasher, other equipment (run independently)
- **Callback queue** = orders waiting for the chef
- **Event loop** = the manager checking: 'Chef free? Here's the next order.'

The chef (JavaScript) is single-threaded, but the kitchen (browser) has multiple things happening. When the oven timer beeps (async operation completes), the dish doesn't interrupt the chef - it joins the queue. When the chef finishes the current dish (call stack empties), the manager (event loop) hands over the next order (callback).

This is why JavaScript can handle async operations without threads: it delegates the waiting to the browser, then processes results one at a time."

**The restaurant metaphor started here**, trying to explain how single-threaded code handles concurrency.

---

## What This Taught Me

**Visualizing invisible systems makes them debuggable.**

The event loop, call stack, and memory heap are invisible when you're writing code. But once I could see them (through diagrams), debugging made sense:
- Stack overflow? Too many nested function calls.
- Callback not running? Call stack never emptied.
- Memory leak? Objects in the heap not getting garbage collected.

**Asynchronous !== concurrent.**

JavaScript handles async operations sequentially through the event loop. It's not running callbacks in parallel - it's managing them in order. Understanding this prevented race conditions and timing bugs.

**The runtime shapes the code patterns.**

Why do we use promises and async/await? Because callbacks in queues are hard to reason about. Promises and async/await are syntax sugar over the same event loop mechanism - they just make it easier to write and read.

---

## Connection to Today

The JavaScript runtime concepts from these notes show up everywhere:
- Understanding why `await` pauses execution (waits for promise resolution before continuing)
- Debugging infinite loops (call stack never empties, event loop never runs)
- Optimizing performance (keeping the call stack clear for responsive UIs)
- Writing async code that doesn't race (understanding execution order)

**The instinct to visualize how code executes started here.**

---

_From 2020 bootcamp notes: JavaScript runtime & fundamentals. Part of [The Archives](../README.md)._ 🌭

← [Back to Archives](../README.md) | [Back to Home](../../README.md) | [TypeScript](../../typescript/README.md)
