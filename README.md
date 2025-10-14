# 🧭 Curiosity Documentation

_A living notebook for everything I’ve taken apart to understand better._

This repo is my personal learning lab. A collection of concise READMEs that document how different tools, frameworks, and developer-experience systems work under the hood.
Each folder represents a topic I've explored while building or debugging in real projects — from **pnpm** and **Nx** to **GitHub Actions**, **Angular**, testing with **Cypress**, **Playwright**, and **Jest**, and **JWTs**.

The goal isn’t to teach exhaustively; it’s to make sense of things I was curious about and keep those notes in a searchable, reusable form.

---

## 🧩 Why this exists

I learn best by documenting what I figure out.  
Writing things down turns curiosity into clarity — and clarity into better code.  
This repo captures that process in public, so others can learn (or correct me) along the way.

---

## 📁 Structure

### Core Topics

**[📦 pnpm](./pnpm/README.md)**
Dependency management, workspace patterns, and Module Federation with Nx.

**[⚙️ GitHub Actions](./github-actions/README.md)**
Reusable workflows, dynamic E2E testing, @actions/core, secrets encryption, and writing coverage to release notes.

**[🏗️ Angular + Nx](./angular-nx/README.md)**
Monorepo architecture, affected commands, computation caching, and dependency graphs.

### Testing

**[🧪 Cypress](./testing/cypress/README.md)**
E2E testing patterns and passing secrets from GitHub Actions to Cypress.

**[🎭 Playwright](./testing/playwright/README.md)**
Cross-browser testing, mobile emulation, and modern E2E patterns.

**[🃏 Jest](./testing/jest/README.md)**
Unit testing, mocking strategies, and TypeScript integration.

**[📊 Code Coverage](./testing/code-coverage/README.md)**
Understanding coverage metrics, best practices, and pragmatic thresholds.

### Security & Auth

**[🔐 JWTs](./jwts/README.md)**
Understanding JSON Web Tokens, claims, issuers, with the passport metaphor.

---

### Quick Navigation

```
curiosity-documentation/
├── pnpm/
│   └── README.md
├── github-actions/
│   └── README.md
├── angular-nx/
│   └── README.md
├── testing/
│   ├── cypress/
│   │   └── README.md
│   ├── playwright/
│   │   └── README.md
│   ├── jest/
│   │   └── README.md
│   └── code-coverage/
│       └── README.md
└── jwts/
    └── README.md
```

---

## 🧠 Themes

- Developer Experience & tooling clarity
- Testing and CI/CD pipelines
- Dependency management
- Build performance
- Architecture patterns that make engineers faster
- Authentication and security fundamentals

---

## 🔍 How I use this

- As a personal reference when solving similar problems later  
- To share context with teammates  
- To document discoveries from debugging or projects  
- To track how my understanding evolves over time

---

## ✍️ About

Created and maintained by **Alexandra Kelstrom** — [@codewizwit](https://github.com/codewizwit)  

---

### 💡 Contributions

While this is primarily a personal archive, thoughtful issues or discussions are welcome — especially if you spot something worth clarifying or expanding.

---

> _"Build with purpose. Ship with Care"_
