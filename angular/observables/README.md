# Angular Observables

_Asynchronous data streams that power Angular apps._

## Overview

Observables are everywhere in Angular. HTTP requests, router events, form value changes, user interactions. If it happens over time, it's probably an Observable.

Think of Observables like a newsletter subscription: you subscribe to get updates, you receive values as they come in, and you can unsubscribe when you're done. Unlike Promises (which resolve once), Observables can emit multiple values over time.

---

## The Newsletter Metaphor

**Promise:**
You order a book online. You get one notification when it arrives. Done.

**Observable:**
You subscribe to a newsletter. You get emails every week. You can unsubscribe anytime, or the newsletter might complete (stop sending emails).

```typescript
// Promise: single value, one-time deal
fetch('/api/user').then(data => console.log(data));

// Observable: stream of values over time
userUpdates$.subscribe(data => console.log(data));
```

**Key differences:**
- Promises execute immediately
- Observables are lazy (only execute when subscribed)
- Promises resolve once
- Observables can emit multiple times

---

## Observables vs Promises: When to Use Each

Both handle asynchronous operations, but they're designed for different scenarios.

### Promises: One and Done

**Use Promises when:**
- You need a single value (one API response, one file read)
- The operation completes once
- You don't need to cancel the operation
- You're working with native browser APIs (`fetch`, `async/await`)

**Example:**
```typescript
// Fetch API returns a Promise
async function getUser() {
  const response = await fetch('/api/user');
  const user = await response.json();
  return user; // Single value, done
}
```

**The vibe:** Set it and forget it. Fire off the request, get your response, move on.

### Observables: Ongoing Streams

**Use Observables when:**
- You need multiple values over time (WebSocket messages, user clicks, form changes)
- You want to cancel the operation (unsubscribe)
- You need to transform data with operators (`map`, `filter`, `debounce`)
- You're working with Angular's HttpClient (it returns Observables)
- You need fine-grained control over async operations

**Example:**
```typescript
// Angular HttpClient returns an Observable
this.http.get('/api/user')
  .pipe(
    map(user => user.name), // Transform the data
    catchError(() => of('Guest')) // Handle errors gracefully
  )
  .subscribe(name => console.log(name));
```

**The vibe:** Ongoing conversation. Subscribe to updates, transform them as they come, unsubscribe when done.

### Side-by-Side Comparison

```typescript
// ============================================
// PROMISE EXAMPLE
// ============================================

// Executes immediately when created
const userPromise = fetch('/api/user').then(res => res.json());

// Can only handle once
userPromise.then(user => console.log(user));
userPromise.then(user => console.log(user)); // Reuses cached result

// Can't cancel (request already sent)
// No operators (map, filter, etc.)

// ============================================
// OBSERVABLE EXAMPLE
// ============================================

// Doesn't execute until subscribed (lazy)
const user$ = this.http.get('/api/user');

// Each subscription can be independent
const sub1 = user$.subscribe(user => console.log(user));
const sub2 = user$.subscribe(user => console.log(user)); // Separate request!

// Can cancel anytime
sub1.unsubscribe(); // Stops the request if still in flight

// Rich operator library
user$.pipe(
  map(user => user.name),
  filter(name => name.length > 3),
  debounceTime(300)
).subscribe(name => console.log(name));
```

### Converting Between Them

**Observable to Promise:**
```typescript
import { firstValueFrom } from 'rxjs';

// Convert Observable to Promise (takes first value)
const user = await firstValueFrom(this.http.get('/api/user'));
```

**Promise to Observable:**
```typescript
import { from } from 'rxjs';

// Convert Promise to Observable
const user$ = from(fetch('/api/user').then(res => res.json()));
user$.subscribe(user => console.log(user));
```

### Real-World Decision Tree

**"I need to fetch user data on page load"**
→ Either works, but Observable gives you retry/error handling operators

**"I need to listen to every keystroke in a search box"**
→ Observable (multiple values, need `debounceTime`)

**"I need to call a third-party library that returns Promises"**
→ Keep the Promise or convert with `from()`

**"I need to cancel a request when user navigates away"**
→ Observable (can unsubscribe)

**"I need to combine data from 3 different API calls"**
→ Observable with `combineLatest()` or `forkJoin()`

**"I'm using async/await"**
→ Promise (or convert Observable with `firstValueFrom()`)

### The Angular Reality

In Angular, you'll mostly use **Observables** because:
- `HttpClient` returns Observables
- Router events are Observables
- Reactive Forms emit Observables
- RxJS operators make complex async logic clean

But you can always convert to Promises when needed with `firstValueFrom()` or `lastValueFrom()`.

**Bottom line:** Promises are great for simple, one-off async operations. Observables shine when you need control, composability, and streams of values over time.

---

## Creating Observables

### From HTTP Requests (Most Common in Angular)

```typescript
import { Component, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-users',
  template: `<p>Check the console</p>`
})
export class UsersComponent {
  http = inject(HttpClient);

  loadUsers() {
    // This returns an Observable
    this.http.get('https://api.example.com/users')
      .subscribe({
        next: (users) => console.log('Received:', users),
        error: (err) => console.error('Error:', err),
        complete: () => console.log('Request complete!')
      });
  }
}
```

**What's happening:**
- `next` - Called for each emitted value
- `error` - Called if something goes wrong
- `complete` - Called when the Observable finishes (HTTP requests complete after one emission)

### Creating Your Own Observable

```typescript
import { Observable } from 'rxjs';

const countdown$ = new Observable<number>(subscriber => {
  let count = 5;

  const interval = setInterval(() => {
    subscriber.next(count);
    count--;

    if (count < 0) {
      subscriber.complete();
      clearInterval(interval);
    }
  }, 1000);

  // Cleanup function (runs on unsubscribe)
  return () => clearInterval(interval);
});

// Subscribe to see it in action
countdown$.subscribe({
  next: (value) => console.log(value), // 5, 4, 3, 2, 1, 0
  complete: () => console.log('Liftoff!')
});
```

**The `$` naming convention:** Observables often have a `$` suffix (like `users$`, `clicks$`) to indicate they're streams.

---

## Common RxJS Operators

Operators transform, filter, and combine Observables. They're like pipes in a plumbing system: data flows through, gets modified, and comes out the other side.

### `map` - Transform Values

```typescript
import { map } from 'rxjs/operators';

this.http.get<User[]>('/api/users')
  .pipe(
    map(users => users.map(u => u.name)) // Extract just the names
  )
  .subscribe(names => console.log(names)); // ['Alice', 'Bob', 'Charlie']
```

**Think of it like:** Array `.map()`, but for streams.

### `filter` - Skip Unwanted Values

```typescript
import { filter } from 'rxjs/operators';

clicks$
  .pipe(
    filter(event => event.button === 0) // Only left clicks
  )
  .subscribe(event => console.log('Left click!'));
```

### `tap` - Side Effects (Debugging)

```typescript
import { tap } from 'rxjs/operators';

this.http.get('/api/data')
  .pipe(
    tap(data => console.log('Data received:', data)), // Log without changing it
    map(data => data.results)
  )
  .subscribe(results => console.log(results));
```

**Use `tap` for:** Logging, debugging, triggering side effects without modifying the stream.

### `catchError` - Handle Errors Gracefully

```typescript
import { catchError } from 'rxjs/operators';
import { of } from 'rxjs';

this.http.get('/api/users')
  .pipe(
    catchError(error => {
      console.error('Failed to load users:', error);
      return of([]); // Return empty array as fallback
    })
  )
  .subscribe(users => console.log(users)); // Won't break if API fails
```

### `switchMap` - Switch to a New Observable

```typescript
import { switchMap } from 'rxjs/operators';

// When search term changes, cancel previous request and start new one
searchTerm$
  .pipe(
    switchMap(term => this.http.get(`/api/search?q=${term}`))
  )
  .subscribe(results => console.log(results));
```

**The metaphor:** You're listening to one radio station (Observable). When you change the channel (new value), you immediately switch to the new station and stop listening to the old one.

**Use `switchMap` for:** Search boxes, autocomplete, any time you want to cancel the previous request.

### `debounceTime` - Wait Before Emitting

```typescript
import { debounceTime } from 'rxjs/operators';

searchInput$
  .pipe(
    debounceTime(300) // Wait 300ms after user stops typing
  )
  .subscribe(value => console.log('Search for:', value));
```

**Use `debounceTime` for:** Search inputs, resize events, anything you don't want to trigger on every keystroke.

### `combineLatest` - Combine Multiple Observables

```typescript
import { combineLatest } from 'rxjs';

const user$ = this.http.get('/api/user');
const settings$ = this.http.get('/api/settings');

combineLatest([user$, settings$])
  .subscribe(([user, settings]) => {
    console.log('Got both:', user, settings);
  });
```

**When to use:** You need data from multiple sources before proceeding.

---

## The AsyncPipe (Your New Best Friend)

The AsyncPipe subscribes to Observables in your template and automatically unsubscribes when the component is destroyed. No memory leaks, no manual cleanup.

### Without AsyncPipe (Manual Subscription)

```typescript
import { Component, OnInit, OnDestroy, inject } from '@angular/core';
import { Subscription } from 'rxjs';
import { UserService } from './user.service';

@Component({
  selector: 'app-users',
  template: `
    @if (users) {
      @for (user of users; track user.id) {
        <p>{{ user.name }}</p>
      }
    }
  `
})
export class UsersComponent implements OnInit, OnDestroy {
  userService = inject(UserService);
  users: User[] = [];
  subscription?: Subscription;

  ngOnInit() {
    this.subscription = this.userService.getUsers()
      .subscribe(users => this.users = users);
  }

  ngOnDestroy() {
    this.subscription?.unsubscribe(); // Don't forget this!
  }
}
```

### With AsyncPipe (Automatic)

```typescript
import { Component, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { UserService } from './user.service';

@Component({
  selector: 'app-users',
  standalone: true,
  imports: [CommonModule],
  template: `
    @if (users$ | async; as users) {
      @for (user of users; track user.id) {
        <p>{{ user.name }}</p>
      }
    }
  `
})
export class UsersComponent {
  userService = inject(UserService);
  users$ = this.userService.getUsers(); // Keep as Observable
}
```

**What `async` does:**
- Subscribes to `users$`
- Updates the template when values emit
- Unsubscribes automatically when component is destroyed
- Handles loading states cleanly

**Best practice:** Use AsyncPipe whenever possible. It prevents memory leaks and simplifies your code.

---

## Unsubscribing (Avoiding Memory Leaks)

If you subscribe manually, you must unsubscribe. Otherwise, the subscription keeps running even after the component is destroyed, causing memory leaks.

### Pattern 1: Store Subscriptions

```typescript
import { Component, OnDestroy } from '@angular/core';
import { Subscription } from 'rxjs';

export class MyComponent implements OnDestroy {
  private subscription = new Subscription();

  ngOnInit() {
    this.subscription.add(
      this.someObservable$.subscribe(data => console.log(data))
    );

    this.subscription.add(
      this.anotherObservable$.subscribe(data => console.log(data))
    );
  }

  ngOnDestroy() {
    this.subscription.unsubscribe(); // Unsubscribes from all
  }
}
```

### Pattern 2: `takeUntilDestroyed()`

```typescript
import { Component, inject } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-example'
})
export class ExampleComponent {
  private destroyRef = inject(DestroyRef);

  ngOnInit() {
    this.someObservable$
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe(data => console.log(data));
    // Automatically unsubscribes when component is destroyed
  }
}
```

**Modern Angular recommendation:** Use `takeUntilDestroyed()` or AsyncPipe. They handle cleanup automatically.

---

## Real-World Patterns

### Search with Debounce

```typescript
import { Component, signal } from '@angular/core';
import { toObservable } from '@angular/core/rxjs-interop';
import { debounceTime, switchMap } from 'rxjs/operators';

@Component({
  selector: 'app-search',
  template: `
    <input [value]="searchTerm()" (input)="searchTerm.set($event.target.value)">

    @if (results$ | async; as results) {
      @for (result of results; track result.id) {
        <p>{{ result.name }}</p>
      }
    }
  `
})
export class SearchComponent {
  searchTerm = signal('');
  searchTerm$ = toObservable(this.searchTerm);

  results$ = this.searchTerm$.pipe(
    debounceTime(300), // Wait for user to stop typing
    switchMap(term => this.http.get(`/api/search?q=${term}`))
  );
}
```

### Polling (Repeated Requests)

```typescript
import { interval, switchMap } from 'rxjs';

// Fetch data every 5 seconds
const poll$ = interval(5000).pipe(
  switchMap(() => this.http.get('/api/status'))
);

poll$.subscribe(status => console.log('Current status:', status));
```

### Retry Failed Requests

```typescript
import { retry } from 'rxjs/operators';

this.http.get('/api/data')
  .pipe(
    retry(3) // Retry up to 3 times on failure
  )
  .subscribe({
    next: data => console.log(data),
    error: err => console.error('Failed after 3 retries:', err)
  });
```

---

## Common Gotchas

### 1. Forgetting to Subscribe

```typescript
// This does NOTHING
this.http.get('/api/users'); // Observable created but not subscribed

// This works
this.http.get('/api/users').subscribe(users => console.log(users));
```

**Observables are lazy.** They don't execute until you subscribe.

### 2. Subscribing in a Loop

```typescript
// Bad: Creates multiple subscriptions
@for (user of users; track user.id) {
  {{ userDetails$ | async }}
}

// Good: Subscribe once, map the data
users$ = this.http.get('/api/users');
```

### 3. Not Unsubscribing

```typescript
// Memory leak!
ngOnInit() {
  this.interval$.subscribe(val => console.log(val));
  // Never unsubscribes, keeps running forever
}

// Fixed: Use AsyncPipe or takeUntilDestroyed()
```

---

## Observables vs Signals

Angular is moving toward Signals for state management, but Observables aren't going anywhere. Here's when to use each:

**Use Signals for:**
- Component state (loading, error, data)
- Simple reactive values
- Synchronous updates

**Use Observables for:**
- Asynchronous events (HTTP, WebSockets, timers)
- Complex stream transformations (debounce, retry, switchMap)
- Events over time (clicks, inputs, scroll)

**Best practice:** Convert Observables to Signals when needed:
```typescript
import { toSignal } from '@angular/core/rxjs-interop';

users$ = this.http.get('/api/users');
users = toSignal(users$, { initialValue: [] }); // Signal from Observable
```

---

## Quick Reference

| Operator | What It Does | Use Case |
|----------|--------------|----------|
| `map` | Transform values | Extract properties, format data |
| `filter` | Skip values | Only even numbers, only clicks |
| `tap` | Side effects | Logging, debugging |
| `catchError` | Handle errors | Provide fallback values |
| `switchMap` | Switch Observables | Search, cancel previous request |
| `debounceTime` | Wait before emitting | Search input, resize events |
| `combineLatest` | Combine multiple streams | Wait for multiple API calls |
| `retry` | Retry on error | Network requests |
| `takeUntilDestroyed` | Auto-unsubscribe | Component cleanup |

---

## Resources

- [RxJS Official Docs](https://rxjs.dev)
- [RxJS Operator Decision Tree](https://rxjs.dev/operator-decision-tree)
- [Learn RxJS](https://www.learnrxjs.io)
- [Angular Signals](../signals/README.md)

---

_Observables: the streams that make Angular flow._ 🌭

← [Back to Home](../../README.md) | [Angular Overview](../README.md) | [Signals](../signals/README.md)
