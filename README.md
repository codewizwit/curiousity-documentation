# 🧭 Curiosity Documentation

_A living notebook for everything I’ve taken apart to understand better._

This repo is my personal learning lab. A collection of concise READMEs that document how different tools, frameworks, and developer-experience systems work under the hood.
Each folder represents a topic I've explored while building or debugging in real projects — from **pnpm** and **Nx** to **GitHub Actions**, **Angular**, **TypeScript**, testing with **Cypress**, **Playwright**, and **Jest**, and **JWTs**.

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

**[📦 npm](./npm/README.md)**
Guide on how to publish to npm using [human-in-the-loop](https://www.npmjs.com/package/@human-in-the-loop/cli) as an example. 

**[⚙️ GitHub Actions](./github-actions/README.md)**
Reusable workflows, dynamic E2E testing, @actions/core, secrets encryption, and writing coverage to release notes.

**[🏗️ Angular + Nx](./angular-nx/README.md)**
Monorepo architecture, affected commands, computation caching, and dependency graphs.

**[⚡ Angular Modern Reactivity](./angular/README.md)**
Understanding signals, computed, and effect — Angular's new reactivity system explained with the whiteboard metaphor.

**[⚛️ Next.js](./nextjs/README.md)**
The full-stack React framework explained with the restaurant kitchen metaphor — from rendering modes to API routes.

**[📘 TypeScript](./typescript/README.md)**
Deep dives into tricky TypeScript concepts with memorable metaphors: null vs undefined (toilet paper), async/await (hot dog stand), and more.

### Testing

**[🧪 Cypress](./testing/cypress/README.md)**
E2E testing patterns and passing secrets from GitHub Actions to Cypress.

**[🎭 Playwright](./testing/playwright/README.md)**
Model-Driven Development (MDD), AI-powered test generation with agents, and intelligent test automation.

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
├── npm/
│   └── README.md
├── github-actions/
│   └── README.md
├── angular-nx/
│   └── README.md
├── angular/
│   ├── README.md
│   ├── signals/
│   │   └── README.md
│   ├── computed/
│   │   └── README.md
│   └── effect/
│       └── README.md
├── nextjs/
│   └── README.md
├── typescript/
│   ├── README.md
│   ├── null-vs-undefined/
│   │   ├── README.md
│   │   └── null-undefined-meme.png
│   └── async-await/
│       └── README.md
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

> _Build with care. Ship with purpose._
