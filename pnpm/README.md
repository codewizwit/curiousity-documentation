
# pnpm for Nx Monorepo with Module Federation

## 🧭 Overview

This document outlines the proposal to migrate from `npm` to `pnpm` in our Nx monorepo environment, which uses Angular, TypeScript, and Module Federation. The main goal is to improve dependency isolation, optimize build times, reduce CI/CD cost, and prepare for scalable multi-team architecture.

---

## 🚀 Why Migrate to `pnpm`

### ✅ Benefits
- **Faster Installations**: `pnpm` installs are significantly faster due to its content-addressable store and efficient symlink structure.
- **Smaller Disk Usage**: Packages are stored once and hard-linked, saving space.
- **Strict Dependency Boundaries**: Each project must explicitly declare its dependencies, reducing accidental and bloated imports — vital for clean module federation boundaries.
- **Improved CI/CD Performance**: Smaller `node_modules`, faster install times, and better caching translate into shorter pipelines.
- **Better Monorepo Fit**: `pnpm` aligns naturally with Nx project boundaries and facilitates enforceable module boundaries, especially for shared libraries.

### ⚠️ Pain Points with npm/yarn
- All packages live at the root, even if not used by all apps.
- Federation builds are slow and risk unwanted cross-dependencies.
- CI builds and local environments often suffer from unnecessary dependency hoisting.

---

## 🧩 How pnpm Works

- `pnpm` uses a **global store** (like a content-addressable file system) and **symlinks** to create lightweight `node_modules` folders scoped per package.
- This means:
  - Less duplication
  - Faster installs
  - Enforced dependency declarations (if not in `package.json`, it won’t work)

---

## 🧠 How `pnpm` Helps with Dynamic Module Federation

### 1. Dependency Isolation per Project
`pnpm` ensures each app or library only sees what it explicitly depends on. This prevents bloating of builds and reduces the risk of shared package collisions (a common issue in MF setups).

### 2. Improved Tree-Shaking & Bundle Splitting
Isolated package visibility improves how Webpack analyzes usage, helping reduce remote bundle sizes — critical for dynamically loaded MF apps.

### 3. Faster Builds & Deployments
`pnpm` links only the necessary dependencies per project, so CI builds and local dev don't process irrelevant packages. This directly improves speed and resource usage in large MF monorepos.

### 4. Predictable Shared Dependencies
You can define shared packages (`singleton`, `eager`, `version`, etc.) without accidental hoisting. This ensures each remote and host MF config is **cleanly defined** and doesn’t rely on root-level leakage.

> Your current model (all packages at root) causes leaky, bloated MF bundles and hidden interdependencies — exactly what `pnpm` helps eliminate.

### BONUS: Combined with Nx Boundaries
- Use `nx.json` and `project.json` to define dependency boundaries.
- You gain traceable, scalable, MF-compatible isolation.

---

## 🛠 Migration Steps

### Step 1: Install pnpm
```bash
npm install -g pnpm
```

### Step 2: Enable `pnpm` in Nx
```bash
npx nx init --pm=pnpm
```

This updates:
- `nx.json` → `"packageManager": "pnpm"`
- Replaces lock file → `pnpm-lock.yaml`
- Generates workspace files with correct settings

### Step 3: Convert Each App/Lib to Declare Its Own Dependencies
- Use `project.json` and enforce boundary rules in `nx.json`
- Eliminate unused root dependencies

### Step 4: Bootstrap Projects
```bash
pnpm install
nx run-many --target=build --all
```

### Step 5: Update CI/CD Pipelines
- Update install command: `pnpm install`
- Update caching keys to include `pnpm-lock.yaml`
- Adjust Docker caching layer if necessary

---

## 🧪 Validation Plan

- Smoke test all major app builds
- Run full e2e + unit test suites
- Validate deployment speed vs. previous baseline
- Measure cold vs. warm install times

---

## ⚖️ Pros and Cons

| Pros | Cons |
|------|------|
| Fast, isolated installs | Might require developer learning |
| Dependency transparency | Initial migration effort |
| Better MF boundaries | CI/CD caching tweaks required |
| Smaller Docker images | Lockfile changes across PRs |
| Speeds up build tooling | May break if dependencies are improperly declared |

---

## 🧩 Is Nx Compatible with Dynamic Module Federation?

**Yes — 100%.** Nx added full support for **Dynamic Module Federation** through their `@nx/angular:webpack-browser` and `@nx/angular:webpack-server` executors, as well as the `@nrwl/angular/module-federation` generator.

You can:
- Dynamically load remotes at runtime
- Define MF maps in runtime configuration (e.g., JSON or env)
- Compose host/remotes using Angular's lazy routes and `loadRemoteModule`

Reference: [Nx MF Docs](https://nx.dev/packages/angular/documents/module-federation)

> If team is struggling with dynamic loading setup, consider using a dynamic remote mapping service or maintaining a runtime manifest JSON in a shared bucket.

---

**Start with one app and shared lib to validate the impact.**
