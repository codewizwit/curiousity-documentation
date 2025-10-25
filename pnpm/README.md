# pnpm for Nx Monorepos

_Package management that doesn't duplicate the world._

## What's the Problem?

Imagine you have 10 apps in your monorepo. Each app needs React. With npm, you install React 10 times. Your `node_modules` folder is now the size of a small country, your CI takes forever, and Docker images are 2GB because why not duplicate everything?

**That's the npm/yarn flat hoisting problem.**

pnpm fixes this by saying: "What if we stored each package once and just linked to it?" Turns out, that's way better.

---

## The Library Metaphor

Think of package management like a library system:

**npm/yarn (flat hoisting):**
- Every person gets their own copy of every book they might need
- Books pile up everywhere
- You can accidentally read books you didn't check out (phantom dependencies)
- Storage costs are insane

**pnpm (content-addressable store):**
- The library has one copy of each book
- You get a library card that points to the books you actually checked out
- Can't read books you didn't explicitly request
- Storage costs are reasonable

**The key difference:** pnpm uses symlinks to create a virtual `node_modules` per project, all pointing to a single global store.

---

## Why This Matters for Nx Monorepos

In an Nx monorepo with multiple apps and libraries, you need:
1. **Strict dependency boundaries** - Each project only sees what it explicitly declares
2. **Fast installs** - Developers shouldn't wait 10 minutes for `npm install`
3. **Clean builds** - Module Federation remotes shouldn't accidentally bundle the entire monorepo
4. **Sane CI/CD** - Pipelines should be fast and Docker images should be small

pnpm gives you all of this.

---

## How pnpm Works

### The Global Store

pnpm keeps one copy of each package version in `~/.pnpm-store`. When you install a package:
1. pnpm checks if it's already in the global store
2. If yes, it creates a symlink. If no, it downloads once and stores it
3. Your project's `node_modules` is just symlinks to the store

**Result:** React installed 10 times becomes React stored once, linked 10 times.

### Strict Dependency Access

With npm, you can accidentally import packages you didn't declare. pnpm blocks this.

```typescript
// In your package.json: no "lodash" dependency

// With npm: This magically works (bad!)
import { get } from 'lodash';

// With pnpm: Error! "lodash" not found (good!)
// You have to add it to package.json first
```

This is **crucial** for Module Federation because it prevents bloated bundles where your remote accidentally includes half the monorepo.

---

## Module Federation + pnpm: The Perfect Pairing

### The Problem Without pnpm

In a typical npm monorepo with Module Federation:
- The host app has access to everything at root `node_modules`
- Remote apps accidentally share dependencies they didn't declare
- Bundles include phantom dependencies
- Webpack can't properly tree-shake because it doesn't know what's intentional

**Your remote was supposed to be 200KB. It's 2MB. Why? Because it bundled things it shouldn't have access to.**

### How pnpm Fixes This

pnpm enforces strict boundaries:
- Host only sees its declared dependencies
- Remote only sees its declared dependencies
- Shared dependencies must be explicitly configured in `webpack.config` (singleton, eager, etc.)
- No accidental bloat

**When you build the remote, Webpack knows exactly what belongs in the bundle because pnpm made everything else invisible.**

---

## Setting Up pnpm in Nx

### 1. Install pnpm Globally

```bash
npm install -g pnpm
```

### 2. Initialize Nx with pnpm

```bash
npx nx init --pm=pnpm
```

This updates:
- `nx.json` → sets `"packageManager": "pnpm"`
- Creates `pnpm-lock.yaml`
- Configures workspace to use pnpm

### 3. Install Dependencies

```bash
pnpm install
```

pnpm reads your `package.json`, downloads what's missing to the global store, and creates symlinks.

### 4. Verify Builds Work

```bash
nx run-many --target=build --all
```

If any build fails with "module not found", it means that project was relying on phantom dependencies. **This is good.** Add the missing dependency to that project's `package.json`.

---

## Workspace Configuration

### pnpm-workspace.yaml

Create `pnpm-workspace.yaml` at the root:

```yaml
packages:
  - 'apps/*'
  - 'libs/*'
```

This tells pnpm which directories contain packages.

### Nx Integration

Nx automatically detects pnpm and adjusts its dependency graph accordingly. No extra config needed.

---

## Module Federation Setup with pnpm

### Shared Dependencies

In your `webpack.config.js` (or Module Federation config):

```javascript
shared: {
  '@angular/core': { singleton: true, strictVersion: true },
  '@angular/common': { singleton: true, strictVersion: true },
  'rxjs': { singleton: true, strictVersion: true },
}
```

With pnpm, these shared packages must be:
1. Declared in both host and remote `package.json`
2. Explicitly marked as `shared` in MF config

**No more accidental sharing via hoisting.**

### Dynamic Remotes

pnpm doesn't affect dynamic Module Federation. Your runtime manifest still works:

```typescript
// runtime-manifest.json
{
  "remotes": {
    "checkout": "https://checkout.example.com/remoteEntry.js",
    "catalog": "https://catalog.example.com/remoteEntry.js"
  }
}
```

The remotes are loaded at runtime just like before, but now each remote's bundle is clean because pnpm enforced strict dependencies during build.

---

## CI/CD with pnpm

### Update Install Command

```yaml
# .github/workflows/ci.yml
- name: Install dependencies
  run: pnpm install --frozen-lockfile
```

`--frozen-lockfile` ensures CI uses exact versions from `pnpm-lock.yaml`.

### Cache Strategy

```yaml
- name: Setup pnpm
  uses: pnpm/action-setup@v2
  with:
    version: 8

- name: Get pnpm store directory
  shell: bash
  run: echo "STORE_PATH=$(pnpm store path --silent)" >> $GITHUB_ENV

- name: Setup pnpm cache
  uses: actions/cache@v3
  with:
    path: ${{ env.STORE_PATH }}
    key: ${{ runner.os }}-pnpm-store-${{ hashFiles('**/pnpm-lock.yaml') }}
```

### Docker Optimization

```dockerfile
# Before pnpm
COPY package*.json ./
RUN npm install
# Result: 500MB+ layer

# With pnpm
COPY pnpm-lock.yaml ./
COPY package.json ./
RUN pnpm install --frozen-lockfile
# Result: 150MB layer (symlinks to global store)
```

---

## Common Issues & Solutions

### "Module not found" After Migration

**Problem:** Build fails because a package was accessible via hoisting but not declared.

**Solution:** Add it to `package.json`. This is pnpm catching phantom dependencies - it's helping you.

```bash
# In the project that fails
pnpm add the-missing-package
```

### Slow First Install

**Problem:** First `pnpm install` seems slow.

**Explanation:** pnpm is populating the global store. Subsequent installs will be lightning fast because packages are already stored.

### CI Cache Misses

**Problem:** CI keeps re-downloading packages.

**Solution:** Cache the pnpm store directory (see CI/CD section above).

---

## The Benefits You'll Notice

**Faster Installs**
- First install: Similar to npm (populating store)
- Subsequent installs: 2-3x faster (symlinks only)
- CI with cache: 5-10x faster

**Smaller Disk Usage**
- npm: 500MB `node_modules` × 10 projects = 5GB
- pnpm: 500MB global store + minimal symlink overhead = ~600MB

**Cleaner Builds**
- Module Federation bundles are smaller
- No phantom dependencies sneaking into production
- Tree-shaking works better

**Better Developer Experience**
- "Why is this package available?" → Check `package.json`
- Clear dependency graph
- Enforced project boundaries

---

## Migration Checklist

- [ ] Install pnpm globally: `npm install -g pnpm`
- [ ] Initialize Nx: `npx nx init --pm=pnpm`
- [ ] Create `pnpm-workspace.yaml`
- [ ] Run `pnpm install`
- [ ] Fix any "module not found" errors (add to `package.json`)
- [ ] Test all builds: `nx run-many --target=build --all`
- [ ] Update CI/CD workflows
- [ ] Update Docker caching strategy
- [ ] Update team documentation
- [ ] Celebrate smaller `node_modules` and faster installs

---

## When to Use pnpm

**Use pnpm when:**
- You have a monorepo with multiple projects
- You're using Module Federation
- CI/CD build times matter
- Disk space matters
- You want strict dependency boundaries

**Stick with npm when:**
- You have a single app (no monorepo)
- Your team is resistant to change
- You rely on hoisting behavior (though you probably shouldn't)

---

## Resources

- [pnpm Documentation](https://pnpm.io)
- [Nx + pnpm Guide](https://nx.dev/recipes/tips-n-tricks/use-pnpm)
- [Module Federation with Nx](https://nx.dev/packages/angular/documents/module-federation)
- [pnpm Benchmarks](https://pnpm.io/benchmarks)

---

_Package management that doesn't duplicate the world. Workspace linking, hoisting, and why node_modules deserves therapy._ 🌭

← [Back to Home](../README.md) | [npm](../npm/README.md) | [Angular + Nx](../angular-nx/README.md)
