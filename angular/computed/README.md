# Computed: Derived State Made Easy

_Excel formulas for your Angular components._

## Overview

One of the most common patterns in Angular apps is **derived state**: values that depend on other values. Before signals, we reached for RxJS operators like `combineLatest`, `map`, and manual calculations. It worked, but it was verbose, error-prone, and easy to create unnecessary work.

`computed()` changes the game. It's a **memoized, automatically-updating value** that recalculates only when its dependencies change. Think of it like an Excel formula: when the cells it references change, it updates automatically. When nothing changes, it returns the cached result instantly.

---

## The Excel Spreadsheet Metaphor

**The Old Way (RxJS combineLatest):**
- You have cells A1, B1, C1 with values
- You want D1 to be the sum of A1 + B1 + C1
- You manually wire up subscriptions to watch A1, B1, and C1
- Every time ANY of them change, you recalculate and update D1
- You have to remember to unsubscribe from all three cells
- It's like writing code to update the formula every time

**The New Way (computed):**
- You write a formula in D1: `=A1+B1+C1`
- Excel automatically tracks which cells the formula depends on
- When A1, B1, or C1 change, D1 updates automatically
- When nothing changes, D1 just returns its cached value
- No manual tracking, no subscriptions, no cleanup

That's `computed()` in Angular signals.

---

## The Old Way: combineLatest

```typescript
import { Component, OnDestroy } from '@angular/core';
import { BehaviorSubject, combineLatest, Subject } from 'rxjs';
import { map, takeUntil } from 'rxjs/operators';

@Component({
  selector: 'app-cart',
  template: `
    <div>
      <p>Items: {{ itemCount$ | async }}</p>
      <p>Subtotal: \${{ subtotal$ | async }}</p>
      <p>Tax (10%): \${{ tax$ | async }}</p>
      <p>Total: \${{ total$ | async }}</p>
    </div>
  `
})
export class CartComponent implements OnDestroy {
  private items$ = new BehaviorSubject<CartItem[]>([]);
  private taxRate$ = new BehaviorSubject<number>(0.1);
  private destroy$ = new Subject<void>();

  // Derive item count
  itemCount$ = this.items$.pipe(
    map(items => items.length)
  );

  // Derive subtotal
  subtotal$ = this.items$.pipe(
    map(items => items.reduce((sum, item) => sum + item.price * item.quantity, 0))
  );

  // Derive tax - depends on subtotal AND taxRate
  tax$ = combineLatest([this.subtotal$, this.taxRate$]).pipe(
    map(([subtotal, rate]) => subtotal * rate)
  );

  // Derive total - depends on subtotal AND tax
  total$ = combineLatest([this.subtotal$, this.tax$]).pipe(
    map(([subtotal, tax]) => subtotal + tax)
  );

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**Problems:**
- Verbose: Every derived value needs `combineLatest` + `map`
- Manual dependency tracking: You list all dependencies explicitly
- Async pipes everywhere in templates
- Cleanup boilerplate
- Hard to debug (async timing issues)
- Can't easily use derived values in component code

---

## The New Way: computed()

```typescript
import { Component, signal, computed } from '@angular/core';

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

@Component({
  selector: 'app-cart',
  template: `
    <div>
      <p>Items: {{ itemCount() }}</p>
      <p>Subtotal: \${{ subtotal() }}</p>
      <p>Tax (10%): \${{ tax() }}</p>
      <p>Total: \${{ total() }}</p>
    </div>
  `
})
export class CartComponent {
  items = signal<CartItem[]>([]);
  taxRate = signal(0.1);

  // Computed values - automatically update when dependencies change
  itemCount = computed(() => this.items().length);

  subtotal = computed(() =>
    this.items().reduce((sum, item) => sum + item.price * item.quantity, 0)
  );

  tax = computed(() => this.subtotal() * this.taxRate());

  total = computed(() => this.subtotal() + this.tax());

  // No cleanup needed! 🎉
}
```

**Benefits:**
- Way less code (no `combineLatest`, no subscriptions)
- Automatic dependency tracking (you just read the signals you need)
- Synchronous and easy to debug
- Memoized by default (caches result until dependencies change)
- No memory leaks
- Can use computed values directly in code

---

## Creating Computed Signals

### Basic Computed

```typescript
import { signal, computed } from '@angular/core';

const count = signal(10);

// Computed automatically tracks dependencies
const double = computed(() => count() * 2);

console.log(double());  // 20

count.set(20);
console.log(double());  // 40 (automatically updated!)
```

### Multiple Dependencies

```typescript
const firstName = signal('Alex');
const lastName = signal('Smith');

// Automatically depends on both firstName and lastName
const fullName = computed(() =>
  `${firstName()} ${lastName()}`
);

console.log(fullName());  // "Alex Smith"

firstName.set('Sam');
console.log(fullName());  // "Sam Smith" (updated!)
```

### Chaining Computed Signals

```typescript
const price = signal(100);
const quantity = signal(3);
const taxRate = signal(0.1);

// First level: subtotal
const subtotal = computed(() => price() * quantity());

// Second level: tax (depends on subtotal)
const tax = computed(() => subtotal() * taxRate());

// Third level: total (depends on subtotal and tax)
const total = computed(() => subtotal() + tax());

console.log(total());  // 330

price.set(200);
console.log(total());  // 660 (all computed values update!)
```

---

## How Computed Works

### Automatic Dependency Tracking

```typescript
const a = signal(1);
const b = signal(2);
const c = signal(3);

// Computed automatically tracks that it depends on 'a' and 'b'
const sum = computed(() => a() + b());  // Reads a() and b()

// Changing 'a' or 'b' triggers recalculation
a.set(10);
console.log(sum());  // 12 (recalculated)

// Changing 'c' does NOT trigger recalculation (not a dependency)
c.set(100);
console.log(sum());  // 12 (still cached, c not used)
```

### Memoization (Caching)

```typescript
const items = signal([1, 2, 3, 4, 5]);

// Expensive computation
const sum = computed(() => {
  console.log('Computing sum...');
  return items().reduce((acc, n) => acc + n, 0);
});

console.log(sum());  // Logs "Computing sum...", returns 15
console.log(sum());  // Returns 15 (cached, no log!)
console.log(sum());  // Returns 15 (cached, no log!)

// Only recomputes when dependency changes
items.set([1, 2, 3, 4, 5, 6]);
console.log(sum());  // Logs "Computing sum...", returns 21
```

**Key insight:** Computed signals only recalculate when their dependencies change AND when the computed value is read. If no one reads it, it doesn't do unnecessary work.

---

## Real-World Examples

### Example 1: Filtered and Sorted List

```typescript
import { Component, signal, computed } from '@angular/core';

interface User {
  id: number;
  name: string;
  age: number;
  active: boolean;
}

@Component({
  selector: 'app-user-list',
  template: `
    <div>
      <input
        type="text"
        [value]="searchTerm()"
        (input)="searchTerm.set($any($event.target).value)"
        placeholder="Search users..."
      >

      <label>
        <input
          type="checkbox"
          [checked]="showActiveOnly()"
          (change)="showActiveOnly.set($any($event.target).checked)"
        >
        Show active only
      </label>

      <select
        [value]="sortBy()"
        (change)="sortBy.set($any($event.target).value)"
      >
        <option value="name">Name</option>
        <option value="age">Age</option>
      </select>

      <p>Showing {{ filteredUsers().length }} of {{ users().length }} users</p>

      <ul>
        @for (user of filteredUsers(); track user.id) {
          <li>
            {{ user.name }} ({{ user.age }})
            {{ user.active ? '✓' : '✗' }}
          </li>
        }
      </ul>
    </div>
  `
})
export class UserListComponent {
  // Source data
  users = signal<User[]>([
    { id: 1, name: 'Alice', age: 30, active: true },
    { id: 2, name: 'Bob', age: 25, active: false },
    { id: 3, name: 'Charlie', age: 35, active: true },
    { id: 4, name: 'Diana', age: 28, active: true },
  ]);

  // Filter/sort controls
  searchTerm = signal('');
  showActiveOnly = signal(false);
  sortBy = signal<'name' | 'age'>('name');

  // Computed: filtered and sorted users
  filteredUsers = computed(() => {
    let result = this.users();

    // Filter by search term
    const term = this.searchTerm().toLowerCase();
    if (term) {
      result = result.filter(user =>
        user.name.toLowerCase().includes(term)
      );
    }

    // Filter by active status
    if (this.showActiveOnly()) {
      result = result.filter(user => user.active);
    }

    // Sort
    const sortKey = this.sortBy();
    result = [...result].sort((a, b) => {
      if (sortKey === 'name') {
        return a.name.localeCompare(b.name);
      } else {
        return a.age - b.age;
      }
    });

    return result;
  });
}
```

**Key points:**
- `filteredUsers` automatically updates when ANY dependency changes
- Dependencies: `users()`, `searchTerm()`, `showActiveOnly()`, `sortBy()`
- No manual wiring - just read the signals you need
- Memoized - only recalculates when inputs change

---

### Example 2: Form Validation

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-signup-form',
  template: `
    <form>
      <input
        type="email"
        [value]="email()"
        (input)="email.set($any($event.target).value)"
        placeholder="Email"
      >
      @if (emailError()) {
        <p class="error">{{ emailError() }}</p>
      }

      <input
        type="password"
        [value]="password()"
        (input)="password.set($any($event.target).value)"
        placeholder="Password"
      >
      @if (passwordError()) {
        <p class="error">{{ passwordError() }}</p>
      }

      <input
        type="password"
        [value]="confirmPassword()"
        (input)="confirmPassword.set($any($event.target).value)"
        placeholder="Confirm Password"
      >
      @if (confirmPasswordError()) {
        <p class="error">{{ confirmPasswordError() }}</p>
      }

      <button
        type="submit"
        [disabled]="!isFormValid()"
      >
        Sign Up
      </button>

      @if (!isFormValid()) {
        <p class="hint">{{ validationSummary() }}</p>
      }
    </form>
  `
})
export class SignupFormComponent {
  email = signal('');
  password = signal('');
  confirmPassword = signal('');

  // Computed validation errors
  emailError = computed(() => {
    const email = this.email();
    if (!email) return 'Email is required';
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      return 'Invalid email format';
    }
    return null;
  });

  passwordError = computed(() => {
    const password = this.password();
    if (!password) return 'Password is required';
    if (password.length < 8) return 'Password must be at least 8 characters';
    if (!/[A-Z]/.test(password)) return 'Password must contain uppercase letter';
    if (!/[0-9]/.test(password)) return 'Password must contain a number';
    return null;
  });

  confirmPasswordError = computed(() => {
    const password = this.password();
    const confirm = this.confirmPassword();
    if (!confirm) return 'Please confirm password';
    if (password !== confirm) return 'Passwords do not match';
    return null;
  });

  // Computed form validity
  isFormValid = computed(() =>
    !this.emailError() &&
    !this.passwordError() &&
    !this.confirmPasswordError()
  );

  // Computed validation summary
  validationSummary = computed(() => {
    const errors = [
      this.emailError(),
      this.passwordError(),
      this.confirmPasswordError()
    ].filter(Boolean);

    return `${errors.length} error${errors.length !== 1 ? 's' : ''} remaining`;
  });
}
```

**Key points:**
- Each validation is a computed signal
- `isFormValid` depends on all validation signals
- Everything updates automatically as user types
- No manual validation orchestration needed

---

### Example 3: Dashboard Statistics

```typescript
import { Component, signal, computed } from '@angular/core';

interface Transaction {
  id: string;
  amount: number;
  type: 'income' | 'expense';
  category: string;
  date: Date;
}

@Component({
  selector: 'app-dashboard',
  template: `
    <div class="dashboard">
      <div class="stat-card">
        <h3>Total Income</h3>
        <p>\${{ totalIncome().toFixed(2) }}</p>
      </div>

      <div class="stat-card">
        <h3>Total Expenses</h3>
        <p>\${{ totalExpenses().toFixed(2) }}</p>
      </div>

      <div class="stat-card">
        <h3>Net Balance</h3>
        <p [class.positive]="netBalance() > 0" [class.negative]="netBalance() < 0">
          \${{ netBalance().toFixed(2) }}
        </p>
      </div>

      <div class="stat-card">
        <h3>Average Transaction</h3>
        <p>\${{ averageTransaction().toFixed(2) }}</p>
      </div>

      <div class="category-breakdown">
        <h3>Top Expense Categories</h3>
        <ul>
          @for (category of topExpenseCategories(); track category.name) {
            <li>
              {{ category.name }}: \${{ category.total.toFixed(2) }}
              ({{ category.percentage.toFixed(1) }}%)
            </li>
          }
        </ul>
      </div>
    </div>
  `
})
export class DashboardComponent {
  transactions = signal<Transaction[]>([
    { id: '1', amount: 1000, type: 'income', category: 'Salary', date: new Date() },
    { id: '2', amount: 200, type: 'expense', category: 'Groceries', date: new Date() },
    { id: '3', amount: 150, type: 'expense', category: 'Gas', date: new Date() },
    { id: '4', amount: 500, type: 'expense', category: 'Rent', date: new Date() },
  ]);

  // Computed: total income
  totalIncome = computed(() =>
    this.transactions()
      .filter(t => t.type === 'income')
      .reduce((sum, t) => sum + t.amount, 0)
  );

  // Computed: total expenses
  totalExpenses = computed(() =>
    this.transactions()
      .filter(t => t.type === 'expense')
      .reduce((sum, t) => sum + t.amount, 0)
  );

  // Computed: net balance (depends on totalIncome and totalExpenses)
  netBalance = computed(() =>
    this.totalIncome() - this.totalExpenses()
  );

  // Computed: average transaction
  averageTransaction = computed(() => {
    const txns = this.transactions();
    if (txns.length === 0) return 0;
    const total = txns.reduce((sum, t) => sum + t.amount, 0);
    return total / txns.length;
  });

  // Computed: category breakdown
  expensesByCategory = computed(() => {
    const expenses = this.transactions().filter(t => t.type === 'expense');
    const categoryMap = new Map<string, number>();

    expenses.forEach(txn => {
      const current = categoryMap.get(txn.category) || 0;
      categoryMap.set(txn.category, current + txn.amount);
    });

    return categoryMap;
  });

  // Computed: top expense categories (depends on expensesByCategory and totalExpenses)
  topExpenseCategories = computed(() => {
    const categories = this.expensesByCategory();
    const total = this.totalExpenses();

    return Array.from(categories.entries())
      .map(([name, amount]) => ({
        name,
        total: amount,
        percentage: total > 0 ? (amount / total) * 100 : 0
      }))
      .sort((a, b) => b.total - a.total)
      .slice(0, 5);  // Top 5
  });
}
```

**Key points:**
- Complex derived state with multiple layers
- `netBalance` depends on `totalIncome` and `totalExpenses`
- `topExpenseCategories` depends on `expensesByCategory` and `totalExpenses`
- Everything updates automatically when `transactions` changes
- Each computation is memoized (cached until dependencies change)

---

## Computed vs. Methods

You might wonder: why use `computed()` instead of just a method?

### Method (Not Cached)

```typescript
@Component({
  template: `
    <!-- Recalculates on EVERY change detection! -->
    <p>Total: {{ calculateTotal() }}</p>
  `
})
export class BadComponent {
  items = signal([1, 2, 3, 4, 5]);

  calculateTotal() {
    console.log('Calculating...');
    return this.items().reduce((sum, n) => sum + n, 0);
  }
}
```

**Problem:** `calculateTotal()` runs on every change detection cycle, even if `items` didn't change. Wasteful!

### Computed (Cached)

```typescript
@Component({
  template: `
    <!-- Only recalculates when items changes! -->
    <p>Total: {{ total() }}</p>
  `
})
export class GoodComponent {
  items = signal([1, 2, 3, 4, 5]);

  total = computed(() => {
    console.log('Calculating...');
    return this.items().reduce((sum, n) => sum + n, 0);
  });
}
```

**Better:** `total()` only recalculates when `items` changes. Cached otherwise!

---

## Common Patterns

### Pattern 1: Aggregations

```typescript
const numbers = signal([1, 2, 3, 4, 5]);

const sum = computed(() =>
  numbers().reduce((acc, n) => acc + n, 0)
);

const average = computed(() => {
  const arr = numbers();
  return arr.length > 0 ? sum() / arr.length : 0;
});

const max = computed(() =>
  Math.max(...numbers())
);
```

### Pattern 2: Conditional Logic

```typescript
const user = signal<User | null>(null);
const isAuthenticated = computed(() => user() !== null);
const isAdmin = computed(() => user()?.role === 'admin');
const canEdit = computed(() => isAuthenticated() && isAdmin());
```

### Pattern 3: Transformations

```typescript
const items = signal<Item[]>([]);

const itemNames = computed(() =>
  items().map(item => item.name)
);

const sortedItems = computed(() =>
  [...items()].sort((a, b) => a.name.localeCompare(b.name))
);
```

---

## Common Mistakes

### Mistake 1: Side Effects in Computed

```typescript
// ❌ BAD - Don't perform side effects in computed
const value = computed(() => {
  console.log('Computing...');  // ⚠️  This is OK (debugging)
  this.api.logEvent('computed');  // ❌ Side effect! Don't do this!
  return this.items().length;
});

// ✅ GOOD - Use effect() for side effects (see effect/README.md)
effect(() => {
  console.log('Items changed:', this.items().length);
  this.api.logEvent('items-changed');
});
```

**Rule:** Computed should be **pure functions**. No side effects, just calculations.

### Mistake 2: Mutating Inside Computed

```typescript
const items = signal([1, 2, 3]);

// ❌ BAD - Mutating the array
const sorted = computed(() => {
  const arr = items();
  arr.sort();  // ❌ Mutates the original array!
  return arr;
});

// ✅ GOOD - Create new array
const sorted = computed(() => {
  return [...items()].sort();
});
```

### Mistake 3: Over-Computing

```typescript
// ❌ BAD - Computing things that don't need computation
const firstName = signal('Alex');
const uppercaseFirstName = computed(() => firstName().toUpperCase());

// ✅ GOOD - Just use a method if it's trivial
const firstName = signal('Alex');
// In template: {{ firstName().toUpperCase() }}
```

**Rule:** Use `computed()` when the calculation is expensive or used in multiple places. For trivial transforms, just use expressions or methods.

---

## Migration from combineLatest

### Before (RxJS)

```typescript
import { combineLatest } from 'rxjs';
import { map } from 'rxjs/operators';

price$ = new BehaviorSubject(100);
quantity$ = new BehaviorSubject(2);
taxRate$ = new BehaviorSubject(0.1);

subtotal$ = combineLatest([this.price$, this.quantity$]).pipe(
  map(([price, quantity]) => price * quantity)
);

tax$ = combineLatest([this.subtotal$, this.taxRate$]).pipe(
  map(([subtotal, rate]) => subtotal * rate)
);

total$ = combineLatest([this.subtotal$, this.tax$]).pipe(
  map(([subtotal, tax]) => subtotal + tax)
);
```

### After (Signals)

```typescript
price = signal(100);
quantity = signal(2);
taxRate = signal(0.1);

subtotal = computed(() => this.price() * this.quantity());
tax = computed(() => this.subtotal() * this.taxRate());
total = computed(() => this.subtotal() + this.tax());
```

**Much simpler!**

---

## Quick Reference

```typescript
import { signal, computed } from '@angular/core';

// Create computed signal
const a = signal(10);
const b = signal(20);
const sum = computed(() => a() + b());

// Read computed value
console.log(sum());  // 30

// Dependencies automatically tracked
a.set(100);
console.log(sum());  // 120

// Memoized (cached until dependencies change)
console.log(sum());  // 120 (returns cached value)

// Chain computed signals
const double = computed(() => sum() * 2);
console.log(double());  // 240

// In templates
{{ sum() }}
{{ double() }}
```

---

## Resources

- [Angular Computed Signals](https://angular.io/guide/signals#computed-signals)
- [Angular Signals Guide](https://angular.io/guide/signals)
- [Angular Blog: Signals Deep Dive](https://blog.angular.io/angular-v16-is-here-4d7a28ec680d)

---

_From complex observable wiring to simple Excel-like formulas._ 🌭

← [Back to Angular](../README.md) | [Back to Home](../../README.md) | [Signals](../signals/README.md) | [Effect](../effect/README.md)
