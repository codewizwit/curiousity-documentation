# Module Federation

_Micro-frontends without the micro-headaches._

## What is Module Federation?

Imagine you're building a mall. Each store is built by a different team, has its own inventory system, and can update independently. But they all share the same parking lot, security system, and food court.

**That's Module Federation.**

It lets you build separate applications that can share code and load each other dynamically at runtime. Your "host" app can load "remote" apps on demand, and they all share common dependencies (like React) without duplicating them.

---

## The Problem It Solves

**Traditional monolith:**
- One giant app
- Every team works in the same codebase
- Deploy everything together (even if you only changed one button)
- Merge conflicts everywhere
- Build times measured in coffee breaks

**Naive micro-frontends:**
- Each team builds separately (good!)
- Each app bundles its own React (200KB × 5 apps = 1MB of React)
- No code sharing
- Inconsistent UX across apps

**Module Federation:**
- Teams build separately (autonomy!)
- Apps share common dependencies (one copy of React for everyone)
- Host can dynamically load remotes at runtime
- Consistent UX through shared component libraries
- Deploy independently

---

## How It Works

Module Federation is a Webpack 5 plugin that turns your applications into containers that can:
1. **Expose** modules for other apps to consume
2. **Consume** modules from other apps
3. **Share** dependencies to avoid duplication

### The Host

The host is the main application that loads remotes:

```javascript
// webpack.config.js (host)
new ModuleFederationPlugin({
  name: 'host',
  remotes: {
    checkout: 'checkout@http://localhost:3001/remoteEntry.js',
    catalog: 'catalog@http://localhost:3002/remoteEntry.js',
  },
  shared: {
    react: { singleton: true },
    'react-dom': { singleton: true },
  },
})
```

### The Remote

A remote exposes modules that the host can load:

```javascript
// webpack.config.js (checkout remote)
new ModuleFederationPlugin({
  name: 'checkout',
  filename: 'remoteEntry.js',
  exposes: {
    './CheckoutPage': './src/CheckoutPage',
    './Cart': './src/components/Cart',
  },
  shared: {
    react: { singleton: true },
    'react-dom': { singleton: true },
  },
})
```

### Loading a Remote

In your host app:

```typescript
// Lazy load the remote component
const CheckoutPage = lazy(() => import('checkout/CheckoutPage'));

// Use it in a route
<Route path="/checkout" element={<CheckoutPage />} />
```

**What happens at runtime:**
1. User navigates to `/checkout`
2. Host fetches `remoteEntry.js` from the checkout app
3. Webpack loads only the `CheckoutPage` module
4. React and react-dom are shared (not duplicated)
5. Component renders

---

## Dynamic Module Federation

The examples above hardcode the remote URLs. In production, you want dynamic remotes:

### Runtime Manifest

```typescript
// manifest.json (loaded from API or CDN)
{
  "remotes": {
    "checkout": "https://checkout.example.com/remoteEntry.js",
    "catalog": "https://catalog.example.com/remoteEntry.js"
  }
}
```

### Loading Dynamic Remotes

```typescript
// Load manifest at runtime
const manifest = await fetch('/api/remotes').then(r => r.json());

// Dynamically register remotes
Object.entries(manifest.remotes).forEach(([name, url]) => {
  // Use Webpack's module federation runtime API
  __webpack_init_sharing__('default');
  const container = await loadRemoteContainer(url);
  await container.init(__webpack_share_scopes__.default);
});
```

This lets you:
- Deploy new versions without redeploying the host
- A/B test different remote versions
- Route users to different remotes based on feature flags
- Handle remote failures gracefully

---

## Shared Dependencies

The killer feature: sharing libraries to avoid duplication.

### Singleton Mode

```javascript
shared: {
  react: {
    singleton: true,  // Only one version allowed
    requiredVersion: '^18.0.0'
  },
}
```

**What happens:**
- Host loads React 18.2.0
- Remote wants React 18.3.0
- Since `singleton: true`, both use 18.2.0
- No duplication, consistent behavior

### Eager Mode

```javascript
shared: {
  '@design-system/components': {
    singleton: true,
    eager: true,  // Load immediately, don't split into async chunk
  },
}
```

Use `eager: true` for critical shared dependencies that should be in the initial bundle.

---

## Common Patterns

### Pattern 1: Shell + Features

**Shell (host):**
- Navigation
- Authentication
- Layout
- Routes

**Features (remotes):**
- Checkout flow
- Product catalog
- User profile
- Admin dashboard

Each feature team owns their remote, deploys independently.

### Pattern 2: Component Library Sharing

```javascript
// Expose design system from a remote
exposes: {
  './Button': './src/components/Button',
  './Input': './src/components/Input',
}

// Consume in other apps
import { Button } from 'design-system/Button';
```

### Pattern 3: State Sharing

Share state management across apps:

```javascript
// Expose Redux store
exposes: {
  './store': './src/store',
}

// Consume in remotes
import { store } from 'host/store';
```

---

## Common Gotchas

### 1. Version Mismatches

**Problem:** Host uses React 17, remote uses React 18.

**Solution:** Use `requiredVersion` and `singleton: true`:

```javascript
shared: {
  react: {
    singleton: true,
    requiredVersion: '^18.0.0',  // Enforce minimum version
    strictVersion: true,  // Fail if version doesn't match
  },
}
```

### 2. Type Safety

**Problem:** TypeScript doesn't know about remote modules.

**Solution:** Declare module types:

```typescript
// global.d.ts
declare module 'checkout/CheckoutPage' {
  const CheckoutPage: React.ComponentType;
  export default CheckoutPage;
}
```

### 3. Remote Failures

**Problem:** Remote app is down or slow.

**Solution:** Error boundaries and fallbacks:

```typescript
<ErrorBoundary fallback={<div>Checkout unavailable</div>}>
  <Suspense fallback={<Loading />}>
    <CheckoutPage />
  </Suspense>
</ErrorBoundary>
```

### 4. Build Order

**Problem:** Host tries to use remote types during build.

**Solution:** Use runtime types only, or build remotes first in CI.

---

## Module Federation + Nx

Nx has built-in generators for Module Federation:

```bash
# Generate a host app
nx g @nx/react:host host-app

# Generate a remote app
nx g @nx/react:remote checkout --host=host-app

# Nx auto-configures webpack with Module Federation
```

Benefits:
- Automatic webpack config
- Dependency graph visualization
- Build orchestration
- Type safety between apps

See also: [Angular + Nx](../angular-nx/README.md) and [pnpm for Nx Monorepos](../pnpm/README.md)

---

## When to Use Module Federation

**Use Module Federation when:**
- Multiple teams working on different features
- Features deploy on different schedules
- You want to share code without publishing to npm
- Build times are too long (monolith problem)
- You need independent deployment

**Don't use it when:**
- Single team, single app
- Simple application
- You're not ready for the complexity
- Your infrastructure can't support multiple deployments

---

## Production Considerations

### Performance

- **Remote bundle size matters** - Keep remotes small
- **Shared dependencies reduce duplication** - But add network overhead for fetching remoteEntry.js
- **Use code splitting** - Don't load everything upfront

### Caching

- **remoteEntry.js should not be cached** - It's the manifest, needs to be fresh
- **Actual chunks can be cached** - They're content-hashed

```nginx
# nginx config
location ~ remoteEntry\.js$ {
  add_header Cache-Control "no-cache, no-store, must-revalidate";
}

location ~ \.(js|css)$ {
  add_header Cache-Control "public, max-age=31536000, immutable";
}
```

### Versioning

- Use semantic versioning for shared dependencies
- Version your remotes (manifest can point to specific versions)
- Test compatibility between host and remote versions

---

## Resources

- [Webpack Module Federation Docs](https://webpack.js.org/concepts/module-federation/)
- [Nx Module Federation Guide](https://nx.dev/recipes/module-federation)
- [Module Federation Examples](https://github.com/module-federation/module-federation-examples)
- [Practical Module Federation Book](https://module-federation.io/)

---

_Micro-frontends without the micro-headaches._ 🌭

← [Back to Home](../README.md) | [Angular + Nx](../angular-nx/README.md) | [pnpm](../pnpm/README.md) | [Webpack](../webpack/README.md)
