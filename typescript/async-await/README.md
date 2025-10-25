# Async/Await in TypeScript

_Understanding asynchronous code with hot dogs._

## Overview

Async/await is JavaScript's way of handling operations that take time: fetching data from a server, reading files, or waiting for user input. Instead of blocking everything while you wait, async/await lets you write asynchronous code that looks synchronous.

The best way to understand this? Think about ordering from a hot dog stand.

---

## The Hot Dog Stand Metaphor

### Synchronous (Blocking) Code - The Bad Way

Imagine a hot dog stand where the vendor does everything one at a time, and nobody else can order until the current order is completely done:

```typescript
// ❌ Synchronous - Everything blocks
function orderHotDog() {
  console.log('Customer 1: I want a hot dog');

  // Vendor starts cooking (blocks everyone)
  cookHotDog();  // Takes 5 minutes

  console.log('Customer 1: Got my hot dog!');

  // Only NOW can customer 2 order
  console.log('Customer 2: I want a hot dog');
  cookHotDog();  // Another 5 minutes
  console.log('Customer 2: Got my hot dog!');
}

// Customer 2 waits 5 minutes doing nothing
// Customer 3 waits 10 minutes doing nothing
// The line never moves!
```

**The Problem:** Everyone is stuck waiting. The vendor can't take new orders while cooking. Customers can't do anything else while waiting.

---

### Asynchronous with Callbacks - The Messy Way

Now the vendor gives you a number and calls you when it's ready. You can walk away and do other things:

```typescript
// ⚠️  Callbacks - Works but gets messy
function orderHotDog(callback: (hotdog: string) => void) {
  console.log('Customer 1: Order placed, got number 42');

  // Vendor starts cooking, gives you a callback
  cookHotDog((hotdog) => {
    console.log('Customer 1: My number was called!');

    // Want to add condiments? Another callback!
    addMustard(hotdog, (hotdogWithMustard) => {

      // Want to add ketchup too? Another callback!
      addKetchup(hotdogWithMustard, (finalHotdog) => {

        // Want fries with that? Another callback!
        orderFries((fries) => {
          console.log('Customer 1: Finally eating!');
        });
      });
    });
  });
}

// This is "Callback Hell" or "Pyramid of Doom"
```

**The Problem:** Code gets deeply nested and hard to read. Error handling is complicated.

---

### Asynchronous with Promises - Better

Promises are like getting a receipt that promises you'll get your hot dog:

```typescript
// ✅ Promises - Much better
function orderHotDog(): Promise<string> {
  console.log('Customer 1: Order placed, got receipt');

  return cookHotDog()
    .then(hotdog => {
      console.log('Customer 1: Hot dog ready!');
      return addMustard(hotdog);
    })
    .then(hotdogWithMustard => {
      return addKetchup(hotdogWithMustard);
    })
    .then(finalHotdog => {
      return orderFries();
    })
    .then(fries => {
      console.log('Customer 1: Finally eating!');
    })
    .catch(error => {
      console.log('Something went wrong:', error);
    });
}
```

**Better!** But still a bit awkward with all the `.then()` chains.

---

### Asynchronous with Async/Await - The Best Way

Async/await makes asynchronous code read like synchronous code, but without blocking:

```typescript
// ✨ Async/Await - Clean and readable
async function orderHotDog() {
  console.log('Customer 1: Order placed');

  // Wait for hot dog (but don't block other customers!)
  const hotdog = await cookHotDog();
  console.log('Customer 1: Got hot dog');

  // Wait for mustard
  const hotdogWithMustard = await addMustard(hotdog);

  // Wait for ketchup
  const finalHotdog = await addKetchup(hotdogWithMustard);

  // Wait for fries
  const fries = await orderFries();

  console.log('Customer 1: Finally eating!');
}

// Meanwhile, other customers can order!
async function placeOrders() {
  orderHotDog();  // Customer 1
  orderHotDog();  // Customer 2 (doesn't wait for Customer 1!)
  orderHotDog();  // Customer 3
}
```

**Perfect!** Reads top-to-bottom like synchronous code, but doesn't block.

---

## The Rules of Async/Await

### Rule 1: `async` Functions Always Return Promises

```typescript
async function getHotDog() {
  return 'Hot Dog';
}

// Even though we returned a string, it's wrapped in a Promise
const promise = getHotDog();
console.log(promise);  // Promise { 'Hot Dog' }

// To get the value, use await or .then()
promise.then(hotdog => console.log(hotdog));  // 'Hot Dog'
```

It's like how the hot dog stand gives you a receipt (Promise) that represents your future hot dog.

### Rule 2: `await` Only Works Inside `async` Functions

```typescript
// ❌ Can't use await at the top level (in most environments)
const hotdog = await getHotDog();  // Error!

// ✅ Must be inside an async function
async function main() {
  const hotdog = await getHotDog();  // Works!
}
```

You can only "wait" for your order if you're actually at the hot dog stand (inside an async function).

### Rule 3: `await` Pauses the Function, Not the Whole Program

```typescript
async function customer1() {
  console.log('Customer 1: Ordering...');
  const hotdog = await cookHotDog();  // Waits here
  console.log('Customer 1: Eating!');
}

async function customer2() {
  console.log('Customer 2: Ordering...');
  const hotdog = await cookHotDog();  // Waits here
  console.log('Customer 2: Eating!');
}

// Both start ordering at the same time
customer1();
customer2();

// Output:
// Customer 1: Ordering...
// Customer 2: Ordering...
// (both wait ~5 seconds)
// Customer 1: Eating!
// Customer 2: Eating!
```

Each customer waits for their own order, but they don't block each other!

---

## Real-World Examples

### Example 1: Fetching Data (API Request)

```typescript
// The old way: Callback hell
function getUserData(userId: string, callback: (data: User) => void) {
  fetchUser(userId, (user) => {
    fetchUserPosts(user.id, (posts) => {
      fetchPostComments(posts[0].id, (comments) => {
        callback({ user, posts, comments });
      });
    });
  });
}

// The new way: Async/await
async function getUserData(userId: string): Promise<UserData> {
  const user = await fetchUser(userId);
  const posts = await fetchUserPosts(user.id);
  const comments = await fetchPostComments(posts[0].id);

  return { user, posts, comments };
}
```

### Example 2: Sequential vs Parallel (Important!)

**Sequential - Like ordering items one at a time:**

```typescript
async function getFullMeal() {
  console.log('Starting order...');

  const hotdog = await cookHotDog();    // Wait 5 seconds
  const fries = await cookFries();      // Wait 3 seconds
  const drink = await pourDrink();      // Wait 1 second

  console.log('Got everything!');
  // Total time: 9 seconds (5 + 3 + 1)
}
```

**Parallel - Like ordering everything at once:**

```typescript
async function getFullMeal() {
  console.log('Starting order...');

  // Start all three at the same time!
  const [hotdog, fries, drink] = await Promise.all([
    cookHotDog(),   // Starts immediately
    cookFries(),    // Starts immediately
    pourDrink()     // Starts immediately
  ]);

  console.log('Got everything!');
  // Total time: 5 seconds (the longest item)
}
```

This is like telling the hot dog stand "I want all three" and they make them simultaneously!

### Example 3: Error Handling with Try/Catch

```typescript
async function orderHotDog() {
  try {
    const hotdog = await cookHotDog();
    const withMustard = await addMustard(hotdog);
    console.log('Enjoying my hot dog!');
    return withMustard;
  } catch (error) {
    // Maybe they ran out of hot dogs
    console.log('Order failed:', error);

    // Fallback: get a burger instead
    return await cookBurger();
  } finally {
    // This runs no matter what
    console.log('Thanks for coming!');
  }
}
```

---

## Common Patterns

### Pattern 1: Waiting for Multiple Things (Promise.all)

```typescript
// Wait for ALL orders to complete
async function getOrders() {
  const orders = await Promise.all([
    getHotDog(),
    getFries(),
    getDrink()
  ]);

  console.log('All orders ready:', orders);
}
```

**Use when:** You need everything before continuing.

### Pattern 2: Racing (Promise.race)

```typescript
// Use whichever vendor is fastest
async function getFood() {
  const food = await Promise.race([
    hotDogStand1.getHotDog(),
    hotDogStand2.getHotDog()
  ]);

  console.log('Got food from fastest stand:', food);
}
```

**Use when:** You want the first result and don't care about the rest.

### Pattern 3: AllSettled (Handle Failures Gracefully)

```typescript
// Try to get multiple items, but don't fail if one fails
async function getOptionalItems() {
  const results = await Promise.allSettled([
    getHotDog(),      // Might succeed
    getFries(),       // Might fail (sold out)
    getDrink()        // Might succeed
  ]);

  results.forEach((result, index) => {
    if (result.status === 'fulfilled') {
      console.log(`Item ${index}: Got it!`, result.value);
    } else {
      console.log(`Item ${index}: Failed`, result.reason);
    }
  });
}
```

**Use when:** You want to try multiple things and handle each success/failure individually.

---

## TypeScript-Specific Tips

### Typing Async Functions

```typescript
// Return type is automatically wrapped in Promise
async function getHotDog(): Promise<string> {
  return 'Hot Dog';  // TypeScript knows this becomes Promise<string>
}

// You can also be explicit
async function getHotDogExplicit(): Promise<string> {
  const hotdog: string = await cookHotDog();
  return hotdog;
}

// Generic async functions
async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url);
  return response.json();
}

const user = await fetchData<User>('/api/user');
```

### Async Function Types

```typescript
// Type for async function
type AsyncFunction = () => Promise<string>;

const getHotDog: AsyncFunction = async () => {
  return 'Hot Dog';
};

// Or use the full signature
type HotDogGetter = () => Promise<string>;
```

---

## Common Mistakes

### Mistake 1: Forgetting `await`

```typescript
// ❌ BAD - Returns a Promise, not the value
async function getHotDog() {
  const hotdog = cookHotDog();  // Missing await!
  console.log(hotdog);  // Promise { <pending> }
}

// ✅ GOOD
async function getHotDog() {
  const hotdog = await cookHotDog();
  console.log(hotdog);  // 'Hot Dog'
}
```

It's like looking at your receipt instead of waiting for your actual hot dog!

### Mistake 2: Using `await` in Loops Unnecessarily

```typescript
// ❌ SLOW - Processes orders one at a time (sequential)
async function processOrders(customers: string[]) {
  for (const customer of customers) {
    await serveCustomer(customer);  // Waits for each one
  }
}
// Time: 5 seconds × 10 customers = 50 seconds

// ✅ FAST - Processes all orders at once (parallel)
async function processOrders(customers: string[]) {
  await Promise.all(
    customers.map(customer => serveCustomer(customer))
  );
}
// Time: 5 seconds (all at once)
```

### Mistake 3: Not Handling Errors

```typescript
// ❌ BAD - Unhandled promise rejection
async function getHotDog() {
  const hotdog = await cookHotDog();  // What if this fails?
  return hotdog;
}

// ✅ GOOD - Handle errors
async function getHotDog() {
  try {
    const hotdog = await cookHotDog();
    return hotdog;
  } catch (error) {
    console.error('Failed to get hot dog:', error);
    throw error;  // Or handle it
  }
}
```

---

## The Hot Dog Stand State Machine

```typescript
type OrderState =
  | { status: 'ordering'; item: string }
  | { status: 'cooking'; item: string; timeRemaining: number }
  | { status: 'ready'; item: string }
  | { status: 'delivered'; item: string }
  | { status: 'failed'; error: string };

async function processOrder(item: string): Promise<OrderState> {
  // Start ordering
  let state: OrderState = { status: 'ordering', item };
  console.log('Order placed:', state);

  // Cooking
  state = { status: 'cooking', item, timeRemaining: 5 };
  console.log('Cooking:', state);
  await wait(5000);

  // Ready
  state = { status: 'ready', item };
  console.log('Ready:', state);

  // Delivered
  state = { status: 'delivered', item };
  console.log('Delivered:', state);

  return state;
}
```

---

## Quick Reference

```typescript
// Declare async function
async function myFunction() { }

// Await a promise
const result = await somePromise();

// Parallel execution
const results = await Promise.all([promise1, promise2, promise3]);

// Race (first to finish)
const winner = await Promise.race([promise1, promise2]);

// Handle all results (even failures)
const results = await Promise.allSettled([promise1, promise2]);

// Error handling
try {
  const result = await riskyOperation();
} catch (error) {
  console.error('Failed:', error);
} finally {
  console.log('Cleanup');
}

// Type annotations
async function getData(): Promise<string> {
  return await fetchData();
}
```

---

## Resources

- [MDN - async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
- [MDN - await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
- [TypeScript Handbook - Promises](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-7.html#asyncawait)
- [JavaScript.info - Async/await](https://javascript.info/async-await)

---

_Understanding async code: You don't block the line while waiting for your hot dog._ 🌭

← [Back to TypeScript](../README.md) | [Back to Home](../../README.md) | [null vs undefined](../null-vs-undefined/README.md)
