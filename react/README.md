# ⚛️ React

_Building user interfaces with components, hooks, and declarative thinking._

React is what happens when someone asks, "What if we could build UIs like LEGO sets instead of spaghetti code?"

Turns out, snapping reusable components together beats manually manipulating DOM nodes like it's 2010. Who knew?

---

## What's React? (The LEGO Metaphor)

Imagine you're building with LEGO blocks. Each piece has a specific shape and purpose. You don't modify the individual blocks: you snap them together in different combinations to build something bigger. When you want to change what you've built, you don't reshape the LEGO pieces; you rearrange them or swap them out.

**That's React.**

React is a JavaScript library for building user interfaces by composing reusable **components**. Each component is like a LEGO block: it has a specific job, receives inputs (props), manages its own state, and renders UI. You build complex interfaces by snapping components together.

The magic? **React is declarative.** You describe *what* the UI should look like based on your data, and React figures out *how* to make it happen.

---

## Core Concepts

### Components: The Building Blocks

Components are functions that return JSX (JavaScript XML: HTML-like syntax in JavaScript).

```jsx
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// Use it:
<Greeting name="Alexandra" />
```

**Think of components as:**
- **Reusable**: Write once, use everywhere
- **Composable**: Nest them to build complex UIs
- **Isolated**: Each component manages its own logic and state

---

### Props: Passing Data Down

Props (properties) are how you pass data from parent to child components. They're **read-only**: children can't modify props.

```jsx
function Button({ label, onClick }) {
  return <button onClick={onClick}>{label}</button>;
}

// Parent passes data down:
<Button label="Click Me" onClick={handleClick} />
```

**The rule:** Data flows **down** the component tree (parent → child). Props are immutable from the child's perspective.

---

### State: Data That Changes

State is data that changes over time and triggers re-renders when updated. Components use the `useState` hook to manage local state.

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Key insight:** When state changes, React re-renders the component with the new state value.

---

## React Hooks: The Superpowers

Hooks are functions that let you "hook into" React features like state and lifecycle from function components.

### useState: Managing Local State

```jsx
const [value, setValue] = useState(initialValue);
```

- **value**: Current state
- **setValue**: Function to update state
- **initialValue**: Starting value (only used on first render)

**The setState rule:** Always use the setter function. Never mutate state directly.

```jsx
// ✅ Correct
setCount(count + 1);

// ❌ Wrong
count = count + 1;
```

---

### useEffect: Side Effects & Lifecycle

`useEffect` runs code after rendering, perfect for side effects like fetching data, subscriptions, or DOM manipulation.

```jsx
useEffect(() => {
  // Code runs after render
  fetchData();

  // Optional cleanup function
  return () => {
    cancelRequest();
  };
}, [dependency]); // Re-run when dependency changes
```

**The dependency array:**
- `[]`: Run once on mount (like componentDidMount)
- `[value]`: Run when `value` changes
- No array: Run after every render (usually not what you want)

**Think of useEffect as:** "After React finishes updating the DOM, run this code."

---

### useContext: Sharing Data Without Prop Drilling

Context lets you share data across the component tree without manually passing props through every level.

```jsx
const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext); // Gets 'dark'
  return <button className={theme}>Click me</button>;
}
```

**When to use Context:**
- Global data (theme, auth, language)
- Data needed by many components at different nesting levels
- Avoiding "prop drilling" (passing props through many layers)

**When NOT to use Context:**
- Local component state
- Data only needed by 2-3 nearby components
- As a replacement for all state management (Context isn't optimized for frequent updates)

---

### Custom Hooks: Reusable Logic

Custom hooks let you extract component logic into reusable functions. They're just functions that use other hooks.

```jsx
// Custom hook
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return width;
}

// Use it in components
function ResponsiveComponent() {
  const width = useWindowWidth();
  return <div>Window width: {width}px</div>;
}
```

**Hook naming convention:** Always start with `use` (e.g., `useState`, `useEffect`, `useCustomHook`).

---

## Component Patterns

### Composition Over Inheritance

React favors **composition**: building complex components by combining simpler ones.

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

function ProfileCard() {
  return (
    <Card>
      <Avatar />
      <UserInfo />
      <Actions />
    </Card>
  );
}
```

**Why composition wins:** More flexible, easier to understand, avoids deep inheritance hierarchies.

---

### Controlled vs Uncontrolled Components

**Controlled:** React state is the "single source of truth" for input values.

```jsx
function ControlledInput() {
  const [value, setValue] = useState('');

  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

**Uncontrolled:** DOM is the source of truth; use refs to access values.

```jsx
function UncontrolledInput() {
  const inputRef = useRef();

  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };

  return <input ref={inputRef} />;
}
```

**Default to controlled**: React state gives you more control and makes testing easier.

---

### Lifting State Up

When multiple components need to share state, move it to their closest common ancestor.

```jsx
function Parent() {
  const [sharedState, setSharedState] = useState('');

  return (
    <>
      <ChildA value={sharedState} onChange={setSharedState} />
      <ChildB value={sharedState} />
    </>
  );
}
```

**The pattern:** State lives in the parent, flows down as props, updates flow up via callback props.

---

## State Management: When to Use What

### Local State (useState)
**Use for:** Component-specific data that doesn't need to be shared.
```jsx
const [isOpen, setIsOpen] = useState(false);
```

### Context API
**Use for:** Global data needed across many components (theme, auth, language).
```jsx
const user = useContext(UserContext);
```

### Redux / Zustand / Jotai
**Use for:** Complex global state with frequent updates, time-travel debugging, or middleware needs.

**The decision tree:**
1. Does only this component need it? → `useState`
2. Do a few nearby components need it? → Lift state up
3. Do many distant components need it? → Context
4. Is it complex global state with lots of updates? → Redux/Zustand

---

## React's Mental Model

### 1. Declarative UI

You don't manipulate the DOM directly. You describe what the UI should look like, and React handles the updates.

```jsx
// ❌ Imperative (manual DOM manipulation)
button.addEventListener('click', () => {
  counter.textContent = parseInt(counter.textContent) + 1;
});

// ✅ Declarative (describe the UI based on state)
<button onClick={() => setCount(count + 1)}>
  Count: {count}
</button>
```

---

### 2. Unidirectional Data Flow

Data flows **down** (parent → child via props), events flow **up** (child → parent via callbacks).

```
        App (state)
         ↓ props
    ┌────────────┐
    ↓            ↓
  Child1      Child2
    ↑            ↑
    └─ events ───┘
```

This makes data flow predictable and easier to debug.

---

### 3. Reconciliation & Virtual DOM

React keeps a "virtual DOM" (lightweight copy of the real DOM in memory). When state changes:

1. React creates a new virtual DOM tree
2. Compares it to the previous tree (diffing)
3. Calculates minimal changes needed
4. Updates only what changed in the real DOM

**Why this matters:** React optimizes DOM updates for you. You focus on *what* should render, React handles *how* to render it efficiently.

---

## Common Patterns & Best Practices

### Don't Mutate State

```jsx
// ❌ Wrong
state.push(newItem);
setState(state);

// ✅ Correct
setState([...state, newItem]);
```

React detects changes by reference comparison. Mutating state directly breaks change detection.

---

### Keys in Lists

Always provide unique `key` props when rendering lists.

```jsx
// ✅ Correct
{items.map(item => (
  <li key={item.id}>{item.name}</li>
))}

// ❌ Wrong (using index)
{items.map((item, index) => (
  <li key={index}>{item.name}</li>
))}
```

**Why keys matter:** React uses keys to track which items changed, were added, or removed. Using indexes as keys causes bugs when the list order changes.

---

### Conditional Rendering

```jsx
// Short-circuit evaluation
{isLoggedIn && <UserProfile />}

// Ternary operator
{isLoading ? <Spinner /> : <Content />}

// Early return
if (error) return <ErrorMessage />;
return <Content />;
```

---

### Event Handling

```jsx
// ✅ Pass function reference
<button onClick={handleClick}>Click</button>

// ✅ Inline arrow function (if you need to pass arguments)
<button onClick={() => handleClick(id)}>Click</button>

// ❌ Don't call immediately
<button onClick={handleClick()}>Click</button>
```

---

## React vs Other Frameworks

| Feature | React | Angular | Vue |
|---------|-------|---------|-----|
| **Type** | Library | Framework | Framework |
| **Learning Curve** | Moderate | Steep | Gentle |
| **Flexibility** | High | Low | Medium |
| **State Management** | Bring your own | Built-in (RxJS) | Built-in (Vuex) |
| **Templating** | JSX | HTML + directives | HTML + directives |
| **Philosophy** | Do one thing well | Full solution | Progressive framework |

**React's strength:** Flexibility. You choose your router, state manager, and build tools. React focuses on the view layer.

---

## Resources

- [Official React Docs](https://react.dev/)
- [React Beta Docs (New)](https://beta.reactjs.org/)
- [React Patterns](https://reactpatterns.com/)
- [Thinking in React](https://react.dev/learn/thinking-in-react)

---

## Quick Reference

### Component Basics
```jsx
function MyComponent({ prop1, prop2 }) {
  const [state, setState] = useState(initialValue);

  useEffect(() => {
    // Side effect
    return () => {
      // Cleanup
    };
  }, [dependency]);

  return <div>{/* JSX */}</div>;
}
```

### Common Hooks
```jsx
const [state, setState] = useState(initial);
useEffect(() => { /* effect */ }, [deps]);
const value = useContext(MyContext);
const ref = useRef(initialValue);
const memoized = useMemo(() => compute(), [deps]);
const callback = useCallback(() => { /* fn */ }, [deps]);
```

---

_Building interfaces one LEGO block at a time. Just don't step on the props._ 🌭

← [Back to Home](../README.md) | [TypeScript](../typescript/README.md) | [Next.js](../nextjs/README.md)
