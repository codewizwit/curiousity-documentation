# Signals: The New State Primitive

_Trading observables for sticky notes._

## Overview

For years, Angular developers reached for `BehaviorSubject` when they needed mutable state. It worked, but it came with baggage: subscriptions, memory leaks, async pipes everywhere, and ceremony.

Angular signals flip the script. They're **simple, synchronous state containers** that components can read directly without subscribing. No pipes, no subscriptions, no cleanup. Just straightforward state.

---

## The Sticky Note Metaphor

**BehaviorSubject (Old Way):**
- You write a note and put it in a tube
- Everyone who wants to know what's on the note has to subscribe to the tube
- Every time you update the note, it gets sent through the tube to all subscribers
- You have to remember to unsubscribe or notes pile up in everyone's inbox (memory leak)

**Signal (New Way):**
- You write a sticky note and put it on the whiteboard
- Anyone can walk up and read it anytime
- When you update the note, everyone who needs it automatically sees the new value
- No subscriptions, no cleanup — just read when you need it

---

## The Old Way: BehaviorSubject

```typescript
import { Component, OnDestroy } from '@angular/core';
import { BehaviorSubject, Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

@Component({
  selector: 'app-counter',
  template: `
    <div>
      <p>Count: {{ count$ | async }}</p>
      <button (click)="increment()">Increment</button>
    </div>
  `
})
export class CounterComponent implements OnDestroy {
  // Create a BehaviorSubject to hold state
  private count$ = new BehaviorSubject<number>(0);
  private destroy$ = new Subject<void>();

  // Expose as observable
  readonly countObservable$ = this.count$.asObservable();

  increment() {
    // Update state with .next()
    this.count$.next(this.count$.value + 1);
  }

  // Need to manually clean up
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
    this.count$.complete();
  }
}
```

**Problems with this approach:**
- Lots of boilerplate (subjects, cleanup, async pipes)
- Easy to forget cleanup → memory leaks
- `.value` access feels awkward
- Hard to debug (async timing issues)
- Verbose for simple state

---

## The New Way: signal()

```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <div>
      <p>Count: {{ count() }}</p>
      <button (click)="increment()">Increment</button>
    </div>
  `
})
export class CounterComponent {
  // Create a signal
  count = signal(0);

  increment() {
    // Update state with .update()
    this.count.update(value => value + 1);
  }

  // No cleanup needed! 🎉
}
```

**Benefits:**
- Way less code (no subscriptions, no cleanup)
- Read values by calling the signal like a function: `count()`
- Update with `.set()` or `.update()`
- Synchronous and easy to debug
- No memory leaks by default

---

## Creating Signals

### Basic Signal Creation

```typescript
import { signal } from '@angular/core';

// Primitive values
const count = signal(0);
const name = signal('Alex');
const isLoggedIn = signal(false);

// Objects (use .update() to modify)
const user = signal({ id: 1, name: 'Alex' });

// Arrays
const todos = signal<string[]>([]);
```

### TypeScript Types

```typescript
// Explicit typing
const count = signal<number>(0);
const user = signal<User | null>(null);

// Inferred types work great
const name = signal('Alex');  // Signal<string>
const items = signal([1, 2, 3]);  // Signal<number[]>
```

---

## Reading Signal Values

### In Templates

```typescript
@Component({
  template: `
    <!-- Just call the signal like a function -->
    <p>Count: {{ count() }}</p>
    <p>User: {{ user().name }}</p>

    <!-- No async pipe needed! -->
    <p>Items: {{ items().length }}</p>
  `
})
export class MyComponent {
  count = signal(0);
  user = signal({ name: 'Alex', age: 30 });
  items = signal([1, 2, 3]);
}
```

### In Component Code

```typescript
export class MyComponent {
  count = signal(0);

  logCount() {
    // Read by calling the signal
    console.log('Current count:', this.count());
  }

  checkCount() {
    // Use in conditionals
    if (this.count() > 10) {
      console.log('Count is high!');
    }
  }
}
```

---

## Updating Signal Values

### Using `.set()` - Replace Entire Value

```typescript
export class MyComponent {
  count = signal(0);
  user = signal({ name: 'Alex', age: 30 });

  reset() {
    // Replace with new value
    this.count.set(0);
  }

  updateUser() {
    // Replace entire object
    this.user.set({ name: 'Sam', age: 25 });
  }
}
```

### Using `.update()` - Modify Based on Current Value

```typescript
export class MyComponent {
  count = signal(0);
  user = signal({ name: 'Alex', age: 30 });
  items = signal([1, 2, 3]);

  increment() {
    // Update based on current value
    this.count.update(current => current + 1);
  }

  incrementByAmount(amount: number) {
    this.count.update(current => current + amount);
  }

  updateUserAge() {
    // Update object properties
    this.user.update(current => ({
      ...current,
      age: current.age + 1
    }));
  }

  addItem(item: number) {
    // Update arrays
    this.items.update(current => [...current, item]);
  }

  removeFirstItem() {
    this.items.update(current => current.slice(1));
  }
}
```

**When to use which:**
- Use `.set()` when you have the complete new value
- Use `.update()` when you need the current value to calculate the next value

---

## Real-World Examples

### Example 1: Todo List

```typescript
import { Component, signal } from '@angular/core';

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

@Component({
  selector: 'app-todo-list',
  template: `
    <div>
      <input #input (keyup.enter)="addTodo(input.value); input.value = ''">
      <button (click)="addTodo(input.value); input.value = ''">Add</button>

      <ul>
        @for (todo of todos(); track todo.id) {
          <li>
            <input
              type="checkbox"
              [checked]="todo.completed"
              (change)="toggleTodo(todo.id)"
            >
            <span [style.text-decoration]="todo.completed ? 'line-through' : 'none'">
              {{ todo.text }}
            </span>
            <button (click)="removeTodo(todo.id)">Delete</button>
          </li>
        }
      </ul>

      <p>Total: {{ todos().length }}, Completed: {{ completedCount() }}</p>
    </div>
  `
})
export class TodoListComponent {
  todos = signal<Todo[]>([]);
  nextId = 0;

  // Computed property (we'll cover this more in computed/README.md)
  completedCount = computed(() =>
    this.todos().filter(t => t.completed).length
  );

  addTodo(text: string) {
    if (!text.trim()) return;

    this.todos.update(current => [
      ...current,
      { id: this.nextId++, text: text.trim(), completed: false }
    ]);
  }

  toggleTodo(id: number) {
    this.todos.update(current =>
      current.map(todo =>
        todo.id === id
          ? { ...todo, completed: !todo.completed }
          : todo
      )
    );
  }

  removeTodo(id: number) {
    this.todos.update(current =>
      current.filter(todo => todo.id !== id)
    );
  }
}
```

### Example 2: Form State

```typescript
import { Component, signal } from '@angular/core';

interface LoginForm {
  email: string;
  password: string;
}

@Component({
  selector: 'app-login',
  template: `
    <form (submit)="handleSubmit()">
      <input
        type="email"
        [value]="form().email"
        (input)="updateEmail($event)"
        placeholder="Email"
      >
      <input
        type="password"
        [value]="form().password"
        (input)="updatePassword($event)"
        placeholder="Password"
      >
      <button
        type="submit"
        [disabled]="isSubmitting()"
      >
        {{ isSubmitting() ? 'Logging in...' : 'Login' }}
      </button>

      @if (error()) {
        <p class="error">{{ error() }}</p>
      }
    </form>
  `
})
export class LoginComponent {
  form = signal<LoginForm>({ email: '', password: '' });
  isSubmitting = signal(false);
  error = signal<string | null>(null);

  updateEmail(event: Event) {
    const email = (event.target as HTMLInputElement).value;
    this.form.update(current => ({ ...current, email }));
  }

  updatePassword(event: Event) {
    const password = (event.target as HTMLInputElement).value;
    this.form.update(current => ({ ...current, password }));
  }

  async handleSubmit() {
    this.error.set(null);
    this.isSubmitting.set(true);

    try {
      await this.authService.login(this.form());
      // Success!
    } catch (err) {
      this.error.set('Login failed. Please try again.');
    } finally {
      this.isSubmitting.set(false);
    }
  }
}
```

---

## Signals vs BehaviorSubject: When to Use Which

### Use Signals When:

- ✅ You need local component state
- ✅ State updates are synchronous
- ✅ You want simple, straightforward reads/writes
- ✅ You're building new Angular components (16+)
- ✅ You want to avoid subscription management

**Example use cases:**
- Form state
- UI toggles (modals, dropdowns)
- Local filters and sorts
- Component-specific flags (loading, error states)

### Use BehaviorSubject/Observables When:

- ✅ You need complex async operations (debounce, throttle, retry)
- ✅ Working with streams of events
- ✅ Need RxJS operators (`switchMap`, `mergeMap`, etc.)
- ✅ Integrating with libraries that return observables
- ✅ Need late subscribers to receive old values over time

**Example use cases:**
- HTTP requests
- WebSocket streams
- Real-time data
- Search with debouncing
- Complex state machines

---

## Bridging Signals and Observables

Angular provides interop functions to convert between signals and observables.

### toSignal() - Observable to Signal

```typescript
import { Component } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';

@Component({...})
export class MyComponent {
  // Convert observable to signal
  user$ = this.userService.getCurrentUser();
  user = toSignal(this.user$, { initialValue: null });

  // Now use in template without async pipe
  // {{ user()?.name }}
}
```

### toObservable() - Signal to Observable

```typescript
import { Component, signal } from '@angular/core';
import { toObservable } from '@angular/core/rxjs-interop';

@Component({...})
export class SearchComponent {
  searchTerm = signal('');

  // Convert signal to observable for RxJS operators
  searchTerm$ = toObservable(this.searchTerm);

  results$ = this.searchTerm$.pipe(
    debounceTime(300),
    switchMap(term => this.api.search(term))
  );
}
```

---

## Common Patterns

### Pattern 1: Loading States

```typescript
export class DataComponent {
  isLoading = signal(false);
  data = signal<Data | null>(null);
  error = signal<string | null>(null);

  async loadData() {
    this.isLoading.set(true);
    this.error.set(null);

    try {
      const result = await this.api.fetchData();
      this.data.set(result);
    } catch (err) {
      this.error.set('Failed to load data');
    } finally {
      this.isLoading.set(false);
    }
  }
}
```

### Pattern 2: Toggle States

```typescript
export class ModalComponent {
  isOpen = signal(false);

  open() {
    this.isOpen.set(true);
  }

  close() {
    this.isOpen.set(false);
  }

  toggle() {
    this.isOpen.update(current => !current);
  }
}
```

### Pattern 3: Accumulating State

```typescript
export class CartComponent {
  items = signal<CartItem[]>([]);

  addItem(item: CartItem) {
    this.items.update(current => [...current, item]);
  }

  removeItem(itemId: string) {
    this.items.update(current =>
      current.filter(item => item.id !== itemId)
    );
  }

  updateQuantity(itemId: string, quantity: number) {
    this.items.update(current =>
      current.map(item =>
        item.id === itemId
          ? { ...item, quantity }
          : item
      )
    );
  }

  clear() {
    this.items.set([]);
  }
}
```

---

## Common Mistakes

### Mistake 1: Forgetting to Call the Signal

```typescript
// ❌ BAD - Reading signal as property
const value = this.count;  // This is the Signal object, not the value!

// ✅ GOOD - Call the signal to get value
const value = this.count();
```

### Mistake 2: Mutating Objects Directly

```typescript
// ❌ BAD - Mutating signal value directly
this.user().age = 31;  // Doesn't trigger updates!

// ✅ GOOD - Use .update() or .set()
this.user.update(current => ({ ...current, age: 31 }));
```

### Mistake 3: Over-Using Signals

```typescript
// ❌ BAD - Using signals for everything
debounceTime$ = toObservable(this.searchTerm).pipe(debounceTime(300));

// ✅ GOOD - Use observables for complex async operations
searchTerm$ = new Subject<string>();
results$ = this.searchTerm$.pipe(
  debounceTime(300),
  switchMap(term => this.api.search(term))
);
```

---

## Migration Checklist

Moving from `BehaviorSubject` to `signal()`:

- [ ] Replace `new BehaviorSubject(initialValue)` with `signal(initialValue)`
- [ ] Replace `.next(value)` with `.set(value)` or `.update(fn)`
- [ ] Replace `.value` with calling the signal: `mySignal()`
- [ ] Remove `| async` pipes from templates
- [ ] Delete `OnDestroy` cleanup code
- [ ] Remove `takeUntil` operators
- [ ] Update tests to call signals instead of subscribing

---

## Quick Reference

```typescript
// Create
const count = signal(0);
const user = signal<User | null>(null);

// Read
const value = count();
const name = user()?.name;

// Update
count.set(10);
count.update(current => current + 1);

// In templates
{{ count() }}
{{ user()?.name }}

// Convert to/from observables
const sig = toSignal(observable$);
const obs$ = toObservable(signal);
```

---

## Resources

- [Angular Signals Guide](https://angular.io/guide/signals)
- [RxJS Interop Guide](https://angular.io/guide/rxjs-interop)
- [Angular Blog: Introducing Signals](https://blog.angular.io/angular-v16-is-here-4d7a28ec680d)

---

_From streams and subscriptions to sticky notes and direct reads._
