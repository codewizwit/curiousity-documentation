# Angular Cheatsheet

_Quick reference for building with modern Angular._

## Overview

This is your go-to reference for the Angular features you'll use daily. We're focusing on **modern Angular** (v14+): standalone components, the new control flow syntax, signals, and the patterns that make Angular feel less like a framework and more like a superpower.

**Not covered here**: The old stuff (NgModules, `*ngIf`, `*ngFor`, lifecycle hooks like `ngOnInit`). If you're learning Angular now, you don't need most of that anymore.

---

## Components

### Creating a Standalone Component

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-hello',
  standalone: true,
  template: `<h1>Hello, {{name}}!</h1>`,
  styles: [`h1 { color: hotpink; }`]
})
export class HelloComponent {
  name = 'World';
}
```

**What's happening:**
- `standalone: true` - No NgModule required. This component is self-contained.
- `template` - Inline HTML. For larger templates, use `templateUrl: './hello.component.html'`
- `styles` - Inline CSS. For external styles, use `styleUrls: ['./hello.component.css']`

**The metaphor:** Think of standalone components like LEGO blocks. Each one brings its own instructions (no module box needed).

---

## Control Flow (The New Way)

Angular ditched `*ngIf` and `*ngFor` for a cleaner, built-in syntax.

### Conditionals: `@if`, `@else`

```typescript
@Component({
  template: `
    @if (isLoggedIn) {
      <p>Welcome back!</p>
    } @else {
      <p>Please log in.</p>
    }
  `
})
export class AppComponent {
  isLoggedIn = false;
}
```

**Old way (don't do this):**
```typescript
<p *ngIf="isLoggedIn">Welcome back!</p>
<p *ngIf="!isLoggedIn">Please log in.</p>
```

**Why the change?** The new syntax is more readable and performs better. No more structural directives with asterisks.

### Loops: `@for`

```typescript
@Component({
  template: `
    <ul>
      @for (item of items; track item.id) {
        <li>{{ item.name }}</li>
      }
    </ul>
  `
})
export class ListComponent {
  items = [
    { id: 1, name: 'Coffee' },
    { id: 2, name: 'Code' },
    { id: 3, name: 'Repeat' }
  ];
}
```

**Important:** Always use `track` to help Angular identify items efficiently. Use a unique identifier like `item.id` or `$index` if you don't have one.

**Old way:**
```typescript
<li *ngFor="let item of items; trackBy: trackById">{{ item.name }}</li>
```

### Switch: `@switch`

```typescript
@Component({
  template: `
    @switch (status) {
      @case ('loading') { <p>Loading...</p> }
      @case ('success') { <p>Data loaded!</p> }
      @case ('error') { <p>Something went wrong.</p> }
      @default { <p>Unknown status</p> }
    }
  `
})
export class StatusComponent {
  status: 'loading' | 'success' | 'error' | 'idle' = 'loading';
}
```

**Old way:**
```typescript
<div [ngSwitch]="status">
  <p *ngSwitchCase="'loading'">Loading...</p>
  <p *ngSwitchCase="'success'">Data loaded!</p>
  <p *ngSwitchDefault>Unknown</p>
</div>
```

---

## Dependency Injection

### Using `inject()` (The Modern Way)

```typescript
import { Component, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-user',
  standalone: true,
  template: `<p>User data...</p>`
})
export class UserComponent {
  private http = inject(HttpClient);

  loadUser() {
    this.http.get('/api/user').subscribe(data => {
      console.log(data);
    });
  }
}
```

**Why `inject()`?**
- Cleaner than constructor injection
- Works in functions and initialization code
- More flexible for composition

**Old way (still works, but verbose):**
```typescript
constructor(private http: HttpClient) {}
```

---

## Inputs & Outputs

### Input Properties (Passing Data In)

```typescript
import { Component, input } from '@angular/core';

@Component({
  selector: 'app-user-card',
  standalone: true,
  template: `<div>{{ userName() }}</div>`
})
export class UserCardComponent {
  userName = input<string>('Guest'); // Signal input with default
}
```

**Usage:**
```typescript
<app-user-card [userName]="'Alexandra'" />
```

**New signal inputs:**
- `input()` - Optional input
- `input.required()` - Required input (TypeScript will complain if missing)

**Old way (still works):**
```typescript
@Input() userName: string = 'Guest';
```

### Output Events (Sending Data Out)

```typescript
import { Component, output } from '@angular/core';

@Component({
  selector: 'app-button',
  standalone: true,
  template: `<button (click)="handleClick()">Click me</button>`
})
export class ButtonComponent {
  clicked = output<string>(); // Signal output

  handleClick() {
    this.clicked.emit('Button was clicked!');
  }
}
```

**Usage:**
```typescript
<app-button (clicked)="onButtonClick($event)" />
```

**Old way:**
```typescript
@Output() clicked = new EventEmitter<string>();
```

---

## Two-Way Binding

### The Banana-in-a-Box Syntax

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-search',
  standalone: true,
  imports: [FormsModule],
  template: `
    <input [(ngModel)]="searchTerm" placeholder="Search...">
    <p>You typed: {{ searchTerm }}</p>
  `
})
export class SearchComponent {
  searchTerm = '';
}
```

**The metaphor:** `[(ngModel)]` looks like a banana in a box. Seriously. That's what Angular devs call it.

**What it does:** Binds the input value to the component property in both directions:
- `[ngModel]` - Property binding (component → template)
- `(ngModelChange)` - Event binding (template → component)
- `[(ngModel)]` - Both at once

**Important:** You need to import `FormsModule` to use `ngModel`.

---

## Services

### Creating a Service

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root' // Available app-wide
})
export class UserService {
  private users = ['Alice', 'Bob', 'Charlie'];

  getUsers() {
    return this.users;
  }

  addUser(name: string) {
    this.users.push(name);
  }
}
```

**Using the service:**
```typescript
import { Component, inject } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-users',
  standalone: true,
  template: `
    @for (user of users; track user) {
      <p>{{ user }}</p>
    }
  `
})
export class UsersComponent {
  userService = inject(UserService);
  users = this.userService.getUsers();
}
```

**The metaphor:** Services are like shared utilities that any component can grab. Think of them as the toolbox everyone on the team has access to.

---

## Routing

### Setting Up Routes

```typescript
import { Routes } from '@angular/router';
import { HomeComponent } from './home.component';
import { AboutComponent } from './about.component';

export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: '**', redirectTo: '' } // Catch-all for 404s
];
```

**Bootstrap with routing:**
```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter } from '@angular/router';
import { AppComponent } from './app.component';
import { routes } from './app.routes';

bootstrapApplication(AppComponent, {
  providers: [provideRouter(routes)]
});
```

### Navigation

```typescript
import { Component } from '@angular/core';
import { RouterLink, RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterLink, RouterOutlet],
  template: `
    <nav>
      <a routerLink="/">Home</a>
      <a routerLink="/about">About</a>
    </nav>
    <router-outlet /> <!-- Where routed components render -->
  `
})
export class AppComponent {}
```

**Programmatic navigation:**
```typescript
import { inject } from '@angular/core';
import { Router } from '@angular/router';

export class MyComponent {
  router = inject(Router);

  goToAbout() {
    this.router.navigate(['/about']);
  }
}
```

---

## HTTP Requests

### Making API Calls

```typescript
import { Component, inject, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-data',
  standalone: true,
  template: `
    @if (loading) {
      <p>Loading...</p>
    } @else {
      <pre>{{ data | json }}</pre>
    }
  `
})
export class DataComponent implements OnInit {
  http = inject(HttpClient);
  data: any;
  loading = true;

  ngOnInit() {
    this.http.get('https://api.example.com/data')
      .subscribe({
        next: (response) => {
          this.data = response;
          this.loading = false;
        },
        error: (err) => console.error('Error:', err)
      });
  }
}
```

**Don't forget to provide HttpClient:**
```typescript
import { provideHttpClient } from '@angular/common/http';

bootstrapApplication(AppComponent, {
  providers: [provideHttpClient()]
});
```

---

## Pipes (Data Transformation)

### Built-in Pipes

```typescript
@Component({
  template: `
    <!-- Uppercase -->
    <p>{{ 'hello' | uppercase }}</p> <!-- HELLO -->

    <!-- Lowercase -->
    <p>{{ 'WORLD' | lowercase }}</p> <!-- world -->

    <!-- Date formatting -->
    <p>{{ today | date:'short' }}</p> <!-- 1/25/25, 3:45 PM -->

    <!-- Currency -->
    <p>{{ price | currency:'USD' }}</p> <!-- $42.00 -->

    <!-- JSON (debugging) -->
    <pre>{{ user | json }}</pre>
  `
})
export class PipeExamplesComponent {
  today = new Date();
  price = 42;
  user = { name: 'Alex', role: 'dev' };
}
```

### Creating a Custom Pipe

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'reverse',
  standalone: true
})
export class ReversePipe implements PipeTransform {
  transform(value: string): string {
    return value.split('').reverse().join('');
  }
}
```

**Usage:**
```typescript
import { ReversePipe } from './reverse.pipe';

@Component({
  standalone: true,
  imports: [ReversePipe],
  template: `<p>{{ 'Angular' | reverse }}</p>` // ralugnA
})
```

---

## Common Patterns

### Safe Navigation Operator `?.`

```typescript
@Component({
  template: `
    <!-- Only access name if user exists -->
    <p>{{ user?.name }}</p>

    <!-- Old way would crash if user is null/undefined -->
    <p>{{ user.name }}</p> <!-- Error if user is null! -->
  `
})
export class SafeComponent {
  user?: { name: string };
}
```

### Non-Null Assertion `!`

```typescript
@Component({
  template: `
    <!-- Tell TypeScript: "Trust me, this will be defined" -->
    <p>{{ user!.name }}</p>
  `
})
export class AssertionComponent {
  user?: { name: string };

  ngOnInit() {
    // You're certain user gets set here
    this.user = { name: 'Alex' };
  }
}
```

**Warning:** Only use `!` when you're 100% sure the value won't be null. Otherwise, you'll get runtime errors.

---

## Quick Reference

| Old Angular | Modern Angular |
|------------|----------------|
| `*ngIf="condition"` | `@if (condition) { }` |
| `*ngFor="let item of items"` | `@for (item of items; track item.id) { }` |
| `[ngSwitch]` | `@switch (value) { @case ('x') { } }` |
| `@Input()` | `input()` or `input.required()` |
| `@Output()` | `output()` |
| `constructor(private http: HttpClient)` | `http = inject(HttpClient)` |
| NgModules | `standalone: true` |

---

## Resources

- [Angular Official Docs](https://angular.dev)
- [Angular Signals Guide](../signals/README.md)
- [Angular Computed Guide](../computed/README.md)
- [Angular Effect Guide](../effect/README.md)
- [Angular Observables Guide](../observables/README.md)

---

_Modern Angular: less boilerplate, more building._ 🌭

← [Back to Home](../../README.md) | [Angular Overview](../README.md) | [Signals](../signals/README.md)
