# Algorithms & Problem Solving: When Efficiency Started to Matter (2020)

_The moment I understood why "it works" isn't always good enough_

---

## Context

**When:** 2020 (Bootcamp - Computer Science fundamentals & algorithms module)
**What I was learning:** Data structures, Big O notation, algorithm complexity, performance optimization
**Why it was hard:** I could make code work, but understanding WHY one solution was better than another required a completely different way of thinking. It wasn't about syntax anymore — it was about efficiency, scalability, and mathematical analysis.
**Where I was:** Transitioning from "making things work" to "making things work well"

**The challenge:** Shifting from "does it work?" to "how fast does it work at scale?"

---

## The Notes

These 5 pages capture me learning to analyze code performance, understand algorithm complexity, and think about scalability before I even write code.

### Page 1: Benchmarking Performance & Algorithms
![Benchmarking Performance](./02-benchmarking-performance-algorithms.jpeg)

**What's happening here:**
"O(1) = O(n) = O(n²)"

Notes on how to measure algorithm performance, comparing constant time, linear time, and quadratic time.

**What I notice now:**
This was the introduction to thinking about **time complexity**. Not "how long does this take on my laptop" but "how does runtime grow as input size increases?" That's the fundamental question of algorithm analysis.

---

### Page 2: Search Patterns
![Search Patterns](./03-search-patterns.jpeg)

**What's happening here:**
Introduction to common search patterns and algorithms for finding data efficiently.

Notes on different approaches to searching through data structures.

**What I notice now:**
I was learning that searching isn't just "loop through everything." There are smarter approaches: binary search cuts the problem in half each time. Hash tables give constant-time lookups. The right search pattern depends on how your data is organized.

---

### Page 3: Bubble Sort
![Bubble Sort](./04-bubble-sort.jpeg)

**What's happening here:**
Notes on the bubble sort algorithm - how it works by repeatedly comparing adjacent elements and swapping them if they're in the wrong order.

Diagrams showing the sorting process step by step.

**What I notice now:**
Bubble sort was my first sorting algorithm. It's simple to understand (compare neighbors, swap if needed, repeat), but inefficient at scale (O(n²)). Learning it taught me that "simple to understand" doesn't mean "good to use in production."

---

### Page 4: Bubble Sort Complexity Analysis
![Bubble Sort Complexity](./05-bubble-sort-complexity.jpeg)

**What's happening here:**
Analyzing the Big O complexity of bubble sort - understanding why it's O(n²) and when that matters.

Breaking down the nested loops and counting comparisons.

**What I notice now:**
This was where Big O became concrete. Bubble sort has nested loops: for each item, you compare it to every other item. That's n × n = O(n²). On a 10-item array? 100 operations. On a 1000-item array? 1,000,000 operations. Now the math matters.

**This was the shift from "it works" to "it scales."**

---

### Page 5: Space-Time Complexity & Big O
![Space-Time Complexity](./06-space-time-complexity-big-o.jpeg)

**What's happening here:**
Notes on the relationship between time complexity and space complexity, analyzed through Big O notation.

Understanding that optimization isn't just about speed — it's about the tradeoff between memory usage and processing time.

**What I notice now:**
**Space-time tradeoff** is everywhere in engineering: you can make things faster by using more memory (caching, lookup tables), or use less memory by doing more computation on the fly. There's no free lunch — just informed choices about which resource (time or space) you're willing to spend.

---

## The Aha Moment

**Page 4 — Bubble Sort Complexity Analysis**

Seeing the **concrete example** of bubble sort's O(n²) complexity made everything click:

Bubble sort has two nested loops:
```
for each item in array:
    for each other item in array:
        compare and swap if needed
```

That's **n × n = n² operations.**

- 10 items → 100 operations (fine)
- 100 items → 10,000 operations (getting slow)
- 1,000 items → 1,000,000 operations (unusable)

The breakthrough: **Big O isn't about exact runtime. It's about how performance changes as input grows.**

An O(n²) algorithm might be faster than O(n) for small inputs. But at scale? The quadratic algorithm becomes unusable while the linear algorithm keeps working.

**Efficiency isn't about speed today. It's about scalability tomorrow.**

---

## How I'd Explain This Now

**Then (2020):**
"Big O notation measures how fast an algorithm runs."

**Now (2025):**
"Big O notation describes how an algorithm's performance changes as the input size grows — it's a measure of scalability, not speed.

Think of it like comparing routes on a road trip:
- **O(1)**: Teleportation — always instant, no matter the distance
- **O(log n)**: Highway with shortcuts — each exit cuts remaining distance in half
- **O(n)**: Straight highway — time doubles when distance doubles
- **O(n²)**: City streets with traffic — every additional mile adds exponentially more time

You pick your route based on distance (input size). For a short trip, city streets are fine. For cross-country? You need a highway.

Same with algorithms: the 'best' algorithm depends on your input size and constraints. A 'slow' algorithm with low memory usage might beat a 'fast' algorithm that requires tons of RAM."

**The road trip metaphor started here** — trying to make Big O concrete instead of abstract.

---

## What This Taught Me

**Think before you code.**

Before learning algorithms, I'd jump straight to implementation. After understanding Big O, I started asking:
- How big will this dataset get?
- How often will this run?
- Is this a one-time script or production code?

**Algorithm analysis isn't academic — it's practical.**

Nested loops on a 10-item array? No problem. On a 10,000-item array? Your app freezes. Understanding complexity means you can spot the problem before you ship it.

**Tradeoffs, not perfection.**

There's no "best" algorithm in a vacuum:
- Fast + memory-heavy? Great if you have RAM.
- Slow + memory-efficient? Perfect for constrained environments.
- O(n) with complex logic? Maybe O(n²) with simpler code is better.

Engineering is choosing the right tradeoff for your constraints.

---

## Connection to Today

The algorithm thinking from these notes shows up everywhere:
- Optimizing database queries (avoid N+1 queries)
- Choosing the right data structure (Map vs Array)
- Analyzing frontend performance (avoiding unnecessary re-renders)
- Understanding when to cache vs. recompute

**The instinct to ask "how does this scale?" started here.**

---

_From 2020 bootcamp notes: Computer Science & Algorithms module. Part of [The Archives](../README.md)._ 🌭

← [Back to Archives](../README.md) | [Back to Home](../../README.md) | [JavaScript](../javascript/README.md)
