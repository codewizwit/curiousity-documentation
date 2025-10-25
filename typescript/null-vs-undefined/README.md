# TypeScript Deep Dives

_Making sense of types, one footgun at a time._

## Overview

TypeScript adds static typing to JavaScript, catching errors at compile time instead of runtime. But with great types comes great confusion, especially around concepts like `null` vs `undefined`, type narrowing, and the dreaded `any`.

This documentation breaks down the tricky parts with practical examples and metaphors that stick.

---

## null vs undefined: The Toilet Paper Metaphor

The best way to understand the difference between `null` and `undefined` is through the universal experience of toilet paper availability.

![null vs undefined toilet paper meme](./null-undefined-meme.png)

### The Visual Guide

**`0` (empty but defined)**
- Toilet paper holder exists
- Roll is completely used up (empty tube)
- You can count exactly zero sheets left
- Defined value, just happens to be zero

**`undefined` (not set up yet)**
- No toilet paper holder at all
- Nothing was ever installed
- The value was never assigned
- Doesn't exist in this context

**`null` (intentionally empty)**
- Toilet paper holder exists
- There's a cardboard tube (empty roll)
- Someone used all of it and left the tube
- Intentionally set to "nothing"

### In Code

```typescript
// 0 - Empty but defined (zero sheets counted)
let tpSheets = 0;
console.log(tpSheets);  // 0
console.log(typeof tpSheets);  // "number"

// undefined - Never set up (no holder installed)
let tpHolder: { sheets: number } | undefined;
console.log(tpHolder);  // undefined
console.log(typeof tpHolder);  // "undefined"

// null - Intentionally empty (empty tube left behind)
let tpRoll: { sheets: number } | null = null;
console.log(tpRoll);  // null
console.log(typeof tpRoll);  // "object" (JavaScript quirk!)
```

---

## The Rules of null and undefined

### When to Use `undefined`

Use `undefined` when:
- A value hasn't been initialized yet
- A function doesn't return anything
- An optional parameter wasn't provided
- A property doesn't exist on an object

```typescript
// Optional parameters default to undefined
function greet(name?: string) {
  console.log(name);  // undefined if not provided
}

greet();  // undefined

// Function returns undefined implicitly
function doNothing() {
  // no return
}

console.log(doNothing());  // undefined

// Accessing non-existent property
const user = { name: 'Alex' };
console.log(user.age);  // undefined
```

### When to Use `null`

Use `null` when:
- You want to explicitly represent "no value"
- You're clearing a previously set value
- An API returns null to indicate absence
- You need to differentiate between "not set yet" and "intentionally empty"

```typescript
// Explicitly represent no value
let currentUser: User | null = null;

// After logout, clear the user
function logout() {
  currentUser = null;  // Intentionally set to nothing
}

// API response pattern
interface ApiResponse {
  data: string | null;  // null means no data, not an error
  error: string | null;  // null means no error
}

const success: ApiResponse = { data: 'Hello', error: null };
const failure: ApiResponse = { data: null, error: 'Not found' };
```

---

## The Toilet Paper State Machine

Here's how toilet paper states map to TypeScript values:

```typescript
type ToiletPaperState =
  | { status: 'full', sheets: number }
  | { status: 'empty', sheets: 0 }
  | { status: 'no-roll', sheets: undefined }
  | { status: 'empty-tube', sheets: null }
  | { status: 'no-holder', sheets: never };

// Full roll
const full: ToiletPaperState = { status: 'full', sheets: 100 };

// Used up - 0 sheets but roll is still there
const empty: ToiletPaperState = { status: 'empty', sheets: 0 };

// No roll on the holder - undefined
const noRoll: ToiletPaperState = { status: 'no-roll', sheets: undefined };

// Empty cardboard tube - null
const emptyTube: ToiletPaperState = { status: 'empty-tube', sheets: null };

// Checking the state
function checkToiletPaper(state: ToiletPaperState) {
  if (state.sheets === 0) {
    console.log('Empty but defined - roll is done');
  } else if (state.sheets === undefined) {
    console.log('No roll installed - undefined');
  } else if (state.sheets === null) {
    console.log('Empty cardboard tube left - null');
  } else {
    console.log(`You have ${state.sheets} sheets left`);
  }
}
```

---

## Common Pitfalls

### Pitfall 1: Checking for Both

Always check for both `null` and `undefined` if you're not sure which you'll get:

```typescript
// ❌ BAD - Only checks for undefined
function getName(user?: User) {
  if (user === undefined) {
    return 'Guest';
  }
  return user.name;
}

// ✅ GOOD - Checks for both
function getName(user?: User | null) {
  if (user == null) {  // Checks both null and undefined (loose equality)
    return 'Guest';
  }
  return user.name;
}

// ✅ ALSO GOOD - Explicit checks
function getName(user?: User | null) {
  if (user === null || user === undefined) {
    return 'Guest';
  }
  return user.name;
}

// ✅ BEST - Nullish coalescing
function getName(user?: User | null) {
  return user?.name ?? 'Guest';
}
```

### Pitfall 2: `typeof null` Returns "object"

This is a JavaScript bug that can't be fixed for backwards compatibility:

```typescript
console.log(typeof null);       // "object" (bug!)
console.log(typeof undefined);  // "undefined"

// ❌ DON'T use typeof to check for null
if (typeof value === 'object') {
  // This includes null!
}

// ✅ DO use strict equality
if (value === null) {
  // Correct null check
}
```

### Pitfall 3: Default Function Parameters

```typescript
// undefined triggers default, but null doesn't
function greet(name: string = 'Guest') {
  console.log(`Hello, ${name}`);
}

greet(undefined);  // "Hello, Guest" (default is used)
greet(null);       // "Hello, null" (default is NOT used)

// Better: Use nullish coalescing
function greet(name?: string | null) {
  console.log(`Hello, ${name ?? 'Guest'}`);
}

greet(undefined);  // "Hello, Guest"
greet(null);       // "Hello, Guest"
greet('');         // "Hello, " (empty string is kept)
```

---

## Type Narrowing with null and undefined

TypeScript can narrow types based on null/undefined checks:

```typescript
function processValue(value: string | null | undefined) {
  // Type: string | null | undefined

  if (value === null) {
    // Type: null
    console.log('Value is explicitly null');
    return;
  }

  if (value === undefined) {
    // Type: undefined
    console.log('Value is undefined');
    return;
  }

  // Type: string (narrowed!)
  console.log(value.toUpperCase());
}

// Shorter version with nullish check
function processValue(value: string | null | undefined) {
  if (value == null) {
    // Type: null | undefined
    console.log('Value is nullish');
    return;
  }

  // Type: string (narrowed!)
  console.log(value.toUpperCase());
}
```

---

## Modern TypeScript Operators

### Optional Chaining (`?.`)

Safely access nested properties that might be null or undefined:

```typescript
interface BathroomSetup {
  toiletPaperHolder?: {
    roll?: {
      sheets: number;
    } | null;
  };
}

const bathroom: BathroomSetup = {};

// ❌ Old way - multiple checks
if (bathroom.toiletPaperHolder &&
    bathroom.toiletPaperHolder.roll &&
    bathroom.toiletPaperHolder.roll.sheets) {
  console.log(bathroom.toiletPaperHolder.roll.sheets);
}

// ✅ New way - optional chaining
console.log(bathroom.toiletPaperHolder?.roll?.sheets);  // undefined
```

### Nullish Coalescing (`??`)

Provide fallback values only for null or undefined (not for `0`, `''`, or `false`):

```typescript
const sheets = 0;

// ❌ Logical OR treats 0 as falsy
console.log(sheets || 100);  // 100 (wrong!)

// ✅ Nullish coalescing only checks null/undefined
console.log(sheets ?? 100);  // 0 (correct!)

// Real-world example
function getSheetCount(count?: number | null): number {
  return count ?? 100;  // Use 100 only if count is null or undefined
}

console.log(getSheetCount(0));         // 0 (keeps zero)
console.log(getSheetCount(null));      // 100 (uses default)
console.log(getSheetCount(undefined)); // 100 (uses default)
```

### Non-null Assertion (`!`)

Tell TypeScript "trust me, this won't be null or undefined":

```typescript
function getSheetCount(bathroom: BathroomSetup): number {
  // ❌ TypeScript error: Object is possibly 'undefined'
  // return bathroom.toiletPaperHolder.roll.sheets;

  // ⚠️  Use with caution - runtime error if actually null/undefined
  return bathroom.toiletPaperHolder!.roll!.sheets;

  // ✅ Better: Provide fallback
  return bathroom.toiletPaperHolder?.roll?.sheets ?? 0;
}
```

---

## strictNullChecks Configuration

TypeScript's `strictNullChecks` flag changes how null and undefined are handled:

```json
// tsconfig.json
{
  "compilerOptions": {
    "strictNullChecks": true  // Recommended!
  }
}
```

**With `strictNullChecks: false` (old behavior):**
```typescript
let name: string = null;  // ✅ Allowed (bad!)
let age: number = undefined;  // ✅ Allowed (bad!)
```

**With `strictNullChecks: true` (strict mode):**
```typescript
let name: string = null;  // ❌ Type 'null' is not assignable to type 'string'
let age: number = undefined;  // ❌ Type 'undefined' is not assignable to type 'number'

// You must explicitly allow null/undefined
let name: string | null = null;  // ✅ OK
let age: number | undefined = undefined;  // ✅ OK
```

---

## Real-World Patterns

### Pattern 1: Nullable Return Types

```typescript
// Find user by ID
function findUser(id: string): User | null {
  const user = database.get(id);
  return user ?? null;  // Return null if not found
}

// Usage
const user = findUser('123');
if (user === null) {
  console.log('User not found');
} else {
  console.log(user.name);  // Type narrowed to User
}
```

### Pattern 2: Optional Configuration

```typescript
interface Config {
  timeout?: number;  // undefined if not provided
  retries?: number;
}

function makeRequest(config: Config = {}) {
  const timeout = config.timeout ?? 5000;  // Default 5 seconds
  const retries = config.retries ?? 3;     // Default 3 retries

  console.log(`Timeout: ${timeout}ms, Retries: ${retries}`);
}

makeRequest();  // Uses all defaults
makeRequest({ timeout: 10000 });  // Custom timeout, default retries
makeRequest({ timeout: 0 });  // 0 is valid! Uses 0, not default
```

### Pattern 3: State Management

```typescript
interface AppState {
  currentUser: User | null;  // null when logged out
  loadingUser: boolean;
  userError: string | undefined;  // undefined when no error
}

const initialState: AppState = {
  currentUser: null,  // Explicitly no user
  loadingUser: false,
  userError: undefined,  // No error yet
};

// After login
const loggedInState: AppState = {
  currentUser: { id: '123', name: 'Alex' },
  loadingUser: false,
  userError: undefined,
};

// After error
const errorState: AppState = {
  currentUser: null,
  loadingUser: false,
  userError: 'Failed to fetch user',
};
```

---

## Quick Reference

```typescript
// Checking for null/undefined
value === null          // Strict null check
value === undefined     // Strict undefined check
value == null           // Checks both (loose equality)
value == undefined      // Checks both (loose equality)

// Type guards
if (value !== null && value !== undefined) { /* value is defined */ }
if (value != null) { /* value is defined (both null and undefined) */ }

// Modern operators
value ?? defaultValue   // Nullish coalescing
object?.property        // Optional chaining
object!.property        // Non-null assertion (use sparingly)

// TypeScript types
type Nullable<T> = T | null;
type Optional<T> = T | undefined;
type Maybe<T> = T | null | undefined;
```

---

## Resources

- [TypeScript Handbook - null and undefined](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#null-and-undefined)
- [MDN - Nullish Coalescing](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)
- [MDN - Optional Chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)

---

_Understanding the difference between "no toilet paper" and "no roll on the holder," one bug at a time._ 🌭

← [Back to Home](../../README.md) | [TypeScript Main](../README.md) | [Async/Await](../async-await/README.md)
