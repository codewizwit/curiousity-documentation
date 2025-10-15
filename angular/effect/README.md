# Effect: Side Effects Without the Ceremony

_The automatic janitor for your reactive code._

## Overview

Side effects are everywhere in Angular apps: logging, localStorage, analytics, API calls, DOM manipulation. Before signals, we handled these with `.subscribe()`, lifecycle hooks like `ngOnInit` and `ngOnChanges`, or a mix of both. It worked, but required manual cleanup and careful orchestration.

`effect()` changes the rules. It's a **function that automatically runs when the signals it reads change**, with automatic cleanup and dependency tracking. No subscriptions, no `takeUntil`, no forgetting to unsubscribe.

Think of it as a janitor that watches the whiteboard: whenever a sticky note changes, the janitor automatically does their job.

---

## The Janitor Metaphor

**The Old Way (subscribe + lifecycle hooks):**
- You hire a janitor to clean when certain notes change
- You manually tell them which notes to watch
- You have to remember to fire the janitor when the room closes (unsubscribe)
- If you forget, the janitor keeps showing up even after the room is gone (memory leak)
- It's manual management all the way down

**The New Way (effect):**
- The janitor automatically knows which notes they care about (dependency tracking)
- Whenever those notes change, they automatically do their job
- When the room closes, the janitor automatically stops showing up (automatic cleanup)
- No manual management needed

---

## The Old Way: subscribe() + Lifecycle Hooks

### Example 1: Logging State Changes

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
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
export class CounterComponent implements OnInit, OnDestroy {
  private count$ = new BehaviorSubject(0);
  private destroy$ = new Subject<void>();

  ngOnInit() {
    // Subscribe to log changes
    this.count$
      .pipe(takeUntil(this.destroy$))
      .subscribe(count => {
        console.log('Count changed:', count);
        localStorage.setItem('count', count.toString());
      });
  }

  increment() {
    this.count$.next(this.count$.value + 1);
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**Problems:**
- Manual subscription management
- Manual cleanup with `takeUntil`
- Boilerplate: `OnInit`, `OnDestroy`, `destroy$` subject
- Easy to forget unsubscribe → memory leaks

---

### Example 2: Syncing Multiple Values

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { BehaviorSubject, Subject, combineLatest } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

@Component({
  selector: 'app-settings',
  template: `...`
})
export class SettingsComponent implements OnInit, OnDestroy {
  private theme$ = new BehaviorSubject<'light' | 'dark'>('light');
  private fontSize$ = new BehaviorSubject<number>(16);
  private destroy$ = new Subject<void>();

  ngOnInit() {
    // Watch both theme and fontSize
    combineLatest([this.theme$, this.fontSize$])
      .pipe(takeUntil(this.destroy$))
      .subscribe(([theme, fontSize]) => {
        console.log('Settings changed:', { theme, fontSize });
        localStorage.setItem('settings', JSON.stringify({ theme, fontSize }));
        document.documentElement.setAttribute('data-theme', theme);
        document.documentElement.style.fontSize = `${fontSize}px`;
      });
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**Problems:**
- Need `combineLatest` to watch multiple values
- Manual cleanup required
- Verbose setup

---

## The New Way: effect()

### Example 1: Logging State Changes

```typescript
import { Component, signal, effect } from '@angular/core';

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
  count = signal(0);

  constructor() {
    // Effect automatically runs when count changes
    effect(() => {
      const currentCount = this.count();
      console.log('Count changed:', currentCount);
      localStorage.setItem('count', currentCount.toString());
    });
  }

  increment() {
    this.count.update(c => c + 1);
  }

  // No cleanup needed! 🎉
}
```

**Benefits:**
- No subscriptions, no cleanup
- Automatically tracks dependencies (reads `count()`)
- Runs when `count` changes
- Automatic cleanup when component destroys

---

### Example 2: Syncing Multiple Values

```typescript
import { Component, signal, effect } from '@angular/core';

@Component({
  selector: 'app-settings',
  template: `...`
})
export class SettingsComponent {
  theme = signal<'light' | 'dark'>('light');
  fontSize = signal(16);

  constructor() {
    // Effect automatically tracks BOTH dependencies
    effect(() => {
      const currentTheme = this.theme();
      const currentFontSize = this.fontSize();

      console.log('Settings changed:', { theme: currentTheme, fontSize: currentFontSize });

      // Side effects
      localStorage.setItem('settings', JSON.stringify({
        theme: currentTheme,
        fontSize: currentFontSize
      }));
      document.documentElement.setAttribute('data-theme', currentTheme);
      document.documentElement.style.fontSize = `${currentFontSize}px`;
    });
  }
}
```

**Benefits:**
- Automatically tracks both `theme` and `fontSize`
- No `combineLatest` needed
- Automatic cleanup
- Way less code

---

## Creating Effects

### Basic Effect

```typescript
import { signal, effect } from '@angular/core';

const count = signal(0);

// Effect runs immediately, then whenever count changes
effect(() => {
  console.log('Count is now:', count());
});

count.set(1);  // Logs: "Count is now: 1"
count.set(2);  // Logs: "Count is now: 2"
```

### Effect with Multiple Dependencies

```typescript
const firstName = signal('Alex');
const lastName = signal('Smith');

// Automatically tracks BOTH dependencies
effect(() => {
  console.log('Full name:', firstName(), lastName());
});

firstName.set('Sam');  // Logs: "Full name: Sam Smith"
lastName.set('Johnson');  // Logs: "Full name: Sam Johnson"
```

### Effect in Components

```typescript
import { Component, signal, effect } from '@angular/core';

@Component({
  selector: 'app-logger',
  template: `<p>Count: {{ count() }}</p>`
})
export class LoggerComponent {
  count = signal(0);

  constructor() {
    // Register effect in constructor
    effect(() => {
      console.log('Effect running, count:', this.count());
    });
  }
}
```

**Important:** Effects must be created in a **constructor** or **injection context**. This ensures they're properly cleaned up when the component is destroyed.

---

## How Effects Work

### Automatic Dependency Tracking

```typescript
const a = signal(1);
const b = signal(2);
const c = signal(3);

// Effect automatically tracks only 'a' and 'b'
effect(() => {
  console.log('Sum:', a() + b());  // Reads a() and b()
});

a.set(10);  // Effect runs
b.set(20);  // Effect runs
c.set(30);  // Effect does NOT run (not a dependency)
```

### Initial Execution

```typescript
const count = signal(0);

effect(() => {
  console.log('Count:', count());
});
// Immediately logs: "Count: 0"

count.set(1);
// Logs: "Count: 1"
```

**Key insight:** Effects run **immediately** when created, then run again whenever their dependencies change.

---

## Real-World Examples

### Example 1: Syncing to localStorage

```typescript
import { Component, signal, effect } from '@angular/core';

interface UserPreferences {
  theme: 'light' | 'dark';
  fontSize: number;
  sidebarOpen: boolean;
}

@Component({
  selector: 'app-preferences',
  template: `
    <div>
      <button (click)="toggleTheme()">Toggle Theme</button>
      <button (click)="increaseFontSize()">Increase Font</button>
      <button (click)="toggleSidebar()">Toggle Sidebar</button>
    </div>
  `
})
export class PreferencesComponent {
  preferences = signal<UserPreferences>({
    theme: 'light',
    fontSize: 16,
    sidebarOpen: true
  });

  constructor() {
    // Load from localStorage on init
    const saved = localStorage.getItem('preferences');
    if (saved) {
      this.preferences.set(JSON.parse(saved));
    }

    // Sync to localStorage whenever preferences change
    effect(() => {
      const prefs = this.preferences();
      localStorage.setItem('preferences', JSON.stringify(prefs));
      console.log('Preferences saved:', prefs);
    });
  }

  toggleTheme() {
    this.preferences.update(p => ({
      ...p,
      theme: p.theme === 'light' ? 'dark' : 'light'
    }));
  }

  increaseFontSize() {
    this.preferences.update(p => ({
      ...p,
      fontSize: p.fontSize + 1
    }));
  }

  toggleSidebar() {
    this.preferences.update(p => ({
      ...p,
      sidebarOpen: !p.sidebarOpen
    }));
  }
}
```

---

### Example 2: Document Title Sync

```typescript
import { Component, signal, effect } from '@angular/core';
import { Title } from '@angular/platform-browser';

@Component({
  selector: 'app-page',
  template: `
    <div>
      <input
        [value]="pageTitle()"
        (input)="pageTitle.set($any($event.target).value)"
        placeholder="Page title"
      >
      <p>Unread: {{ unreadCount() }}</p>
    </div>
  `
})
export class PageComponent {
  pageTitle = signal('My App');
  unreadCount = signal(0);

  constructor(private titleService: Title) {
    // Update document title whenever pageTitle or unreadCount changes
    effect(() => {
      const title = this.pageTitle();
      const unread = this.unreadCount();

      const fullTitle = unread > 0
        ? `(${unread}) ${title}`
        : title;

      this.titleService.setTitle(fullTitle);
    });
  }
}
```

---

### Example 3: Analytics Tracking

```typescript
import { Component, signal, effect } from '@angular/core';

@Component({
  selector: 'app-analytics',
  template: `
    <div>
      <button (click)="currentView.set('home')">Home</button>
      <button (click)="currentView.set('profile')">Profile</button>
      <button (click)="currentView.set('settings')">Settings</button>
    </div>
  `
})
export class AnalyticsComponent {
  currentView = signal('home');
  userId = signal<string | null>(null);

  constructor(private analytics: AnalyticsService) {
    // Track page views
    effect(() => {
      const view = this.currentView();
      console.log('Tracking page view:', view);
      this.analytics.trackPageView(view);
    });

    // Track user context
    effect(() => {
      const user = this.userId();
      if (user) {
        console.log('Setting user context:', user);
        this.analytics.setUserId(user);
      }
    });
  }
}
```

---

### Example 4: API Synchronization

```typescript
import { Component, signal, effect } from '@angular/core';

@Component({
  selector: 'app-auto-save',
  template: `
    <textarea
      [value]="content()"
      (input)="content.set($any($event.target).value)"
    ></textarea>

    @if (isSaving()) {
      <p>Saving...</p>
    }

    @if (lastSaved()) {
      <p>Last saved: {{ lastSaved() }}</p>
    }
  `
})
export class AutoSaveComponent {
  content = signal('');
  isSaving = signal(false);
  lastSaved = signal<Date | null>(null);

  constructor(private api: ApiService) {
    // Auto-save content changes
    effect(() => {
      const currentContent = this.content();

      // Skip if empty
      if (!currentContent.trim()) return;

      this.isSaving.set(true);

      // Debounced save (in real app, use debounceTime with toObservable)
      setTimeout(() => {
        this.api.saveContent(currentContent).then(() => {
          this.isSaving.set(false);
          this.lastSaved.set(new Date());
        });
      }, 1000);
    });
  }
}
```

---

## Effect vs. Computed

A common question: **When should I use `effect()` vs `computed()`?**

### Use `computed()` When:

- ✅ You're **deriving a value** from other signals
- ✅ The result is **used in templates or other code**
- ✅ It's a **pure calculation** (no side effects)

```typescript
// ✅ GOOD - Computed for derived values
const price = signal(100);
const quantity = signal(2);
const total = computed(() => price() * quantity());
```

### Use `effect()` When:

- ✅ You need to perform **side effects** (logging, API calls, DOM updates)
- ✅ You **don't need to return a value**
- ✅ It's an **action**, not a calculation

```typescript
// ✅ GOOD - Effect for side effects
const theme = signal('light');

effect(() => {
  document.body.className = theme();  // Side effect!
});
```

### Anti-Pattern: Using Effect for Derived State

```typescript
// ❌ BAD - Don't use effect to derive state
const price = signal(100);
const quantity = signal(2);
const total = signal(0);

effect(() => {
  total.set(price() * quantity());  // ❌ Wrong tool!
});

// ✅ GOOD - Use computed instead
const price = signal(100);
const quantity = signal(2);
const total = computed(() => price() * quantity());
```

**Rule of thumb:**
- **Computed:** For values (returns something)
- **Effect:** For actions (does something)

---

## Common Patterns

### Pattern 1: Conditional Effects

```typescript
const isAuthenticated = signal(false);
const userId = signal<string | null>(null);

effect(() => {
  if (!isAuthenticated()) {
    // Not authenticated, skip
    return;
  }

  const user = userId();
  if (user) {
    console.log('Loading user data for:', user);
    api.loadUserData(user);
  }
});
```

### Pattern 2: Cleanup with onCleanup

```typescript
import { effect } from '@angular/core';

const query = signal('');

effect((onCleanup) => {
  const currentQuery = query();

  // Setup: Start async operation
  const controller = new AbortController();
  fetch(`/api/search?q=${currentQuery}`, {
    signal: controller.signal
  });

  // Cleanup: Cancel on next run or destroy
  onCleanup(() => {
    console.log('Cleaning up previous search');
    controller.abort();
  });
});
```

**Use case:** Cancel previous operations when dependencies change.

### Pattern 3: Effect Options

```typescript
import { effect } from '@angular/core';

const count = signal(0);

effect(() => {
  console.log('Count:', count());
}, {
  allowSignalWrites: true  // Allow writing to signals inside effect (use with caution!)
});
```

---

## Common Mistakes

### Mistake 1: Infinite Loops

```typescript
// ❌ BAD - Creates infinite loop!
const count = signal(0);

effect(() => {
  console.log('Count:', count());
  count.update(c => c + 1);  // ❌ Writes to signal it reads!
});

// Effect runs → increments count → triggers effect → increments count → ...
```

**Solution:** Don't write to signals you read in the same effect (unless you have `allowSignalWrites: true` and careful logic).

### Mistake 2: Creating Effects Outside Injection Context

```typescript
@Component({...})
export class MyComponent {
  count = signal(0);

  ngOnInit() {
    // ❌ BAD - Effect created outside constructor
    effect(() => {
      console.log('Count:', this.count());
    });
  }
}
```

**Solution:** Create effects in the **constructor** or use `inject()` context:

```typescript
import { effect, inject, Injector } from '@angular/core';

@Component({...})
export class MyComponent {
  count = signal(0);
  private injector = inject(Injector);

  ngOnInit() {
    // ✅ GOOD - Provide injection context
    effect(() => {
      console.log('Count:', this.count());
    }, { injector: this.injector });
  }
}
```

### Mistake 3: Using Effect for Derived State

```typescript
// ❌ BAD - Using effect to update another signal
const firstName = signal('Alex');
const lastName = signal('Smith');
const fullName = signal('');

effect(() => {
  fullName.set(`${firstName()} ${lastName()}`);  // ❌ Use computed instead!
});

// ✅ GOOD - Use computed for derived values
const fullName = computed(() => `${firstName()} ${lastName()}`);
```

---

## Migration from subscribe()

### Before (RxJS)

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { BehaviorSubject, Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

@Component({...})
export class OldComponent implements OnInit, OnDestroy {
  private theme$ = new BehaviorSubject<'light' | 'dark'>('light');
  private destroy$ = new Subject<void>();

  ngOnInit() {
    this.theme$
      .pipe(takeUntil(this.destroy$))
      .subscribe(theme => {
        document.body.className = theme;
        localStorage.setItem('theme', theme);
      });
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### After (Signals + Effect)

```typescript
import { Component, signal, effect } from '@angular/core';

@Component({...})
export class NewComponent {
  theme = signal<'light' | 'dark'>('light');

  constructor() {
    effect(() => {
      const currentTheme = this.theme();
      document.body.className = currentTheme;
      localStorage.setItem('theme', currentTheme);
    });
  }

  // No cleanup needed!
}
```

**Much simpler!**

---

## Quick Reference

```typescript
import { signal, effect, computed } from '@angular/core';

// Create effect
const count = signal(0);

effect(() => {
  console.log('Count:', count());  // Runs immediately, then on changes
});

// Multiple dependencies (automatic)
const a = signal(1);
const b = signal(2);

effect(() => {
  console.log('Sum:', a() + b());  // Tracks both a and b
});

// Cleanup
effect((onCleanup) => {
  const timer = setTimeout(() => console.log('Done'), 1000);

  onCleanup(() => {
    clearTimeout(timer);
  });
});

// When to use what
const derived = computed(() => a() + b());  // ✅ For values
effect(() => console.log(a() + b()));      // ✅ For actions
```

---

## Resources

- [Angular Effects Guide](https://angular.io/guide/signals#effects)
- [Angular Signals Documentation](https://angular.io/guide/signals)
- [Angular Blog: Signal APIs](https://blog.angular.io/angular-v16-is-here-4d7a28ec680d)

---

_From manual subscriptions and cleanup to automatic reactive side effects._
