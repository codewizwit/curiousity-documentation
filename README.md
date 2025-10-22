# 🧭 Curiosity Documentation

_A living notebook for topics I've taken apart to understand better._

**Built by Alexandra Kelstrom** — a software engineer who puts people first. I believe the best way to understand something is to explain it simply. I approach engineering with curiosity and empathy, breaking down complexity with metaphors and documenting the journey from confusion to clarity.

This repo is my personal learning lab — a collection of deep dives into things that spark my curiosity and passion. Some topics come from building or debugging real projects. Others are just rabbit holes worth exploring. From **responsible AI** and **accessibility** to **Angular signals**, **NestJS**, **Next.js**, **GitHub Actions**, **TypeScript**, **ancient pyramid engineering**, **pnpm**, **Nx**, testing with **Cypress**, **Playwright**, and **Jest**, and **JWTs**... If I wanted to understand how it really works, it's probably documented here.

The goal isn’t to teach exhaustively; it’s to make sense of things I was curious about and keep those notes in a searchable, reusable form.

---

## 🧩 Why this exists

I learn best by documenting what I figure out.
Writing things down turns curiosity into clarity; and clarity into better code.
This repo captures that process in public, so others can learn (or correct me) along the way.

<div align="center">
  <img src="https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExdm01dTNwdGc4YTNqbmJwMmxxeGRsNWNoajhqanJ4bG53YXJsMXQ2MyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/N7aAkvk6lSpYoLaAEi/giphy.gif" alt="Why don't you explain this to me like I'm five" width="500">
  <p><em>My documentation philosophy in a nutshell.</em></p>
</div>

Every topic here is broken down with metaphors, relatable examples, and the kind of explanations that would make sense to your past self when you first encountered the concept. No gatekeeping. No assuming prior knowledge. Just clear, honest learning in public.

---

## ⭐ Featured Deep Dives

Here are some of the most detailed explorations in this repo — great starting points if you're new here:

**[🤝 Responsible AI](./responsible-ai/README.md)** — _No Human Left Behind._ A manifesto and practical framework for building AI that puts people first. Covers the Developer-First Accountability Framework, critical questions for AI adoption, and why human dignity must be non-negotiable in our systems. Includes a deep dive on [Building with LLMs](./responsible-ai/llm/README.md) with security patterns, guardrails, and human-in-the-loop implementation.

**[♿ Web Accessibility](./accessibility/README.md)** — Building for everyone, not just some. Practical patterns for semantic HTML, keyboard navigation, screen readers, ARIA, color contrast, and accessible forms. Universal design benefits everyone.

**[⚡ Angular Modern Reactivity](./angular/README.md)** — Understanding signals, computed, and effect with the whiteboard metaphor. A complete breakdown of Angular's new reactive primitives and when to use each one.

**[⚙️ GitHub Actions](./github-actions/README.md)** — Dynamic E2E testing, reusable workflows, secrets encryption with libsodium, and writing coverage reports to release notes. Real-world CI/CD patterns.

**[🏗️ Ancient Pyramids](./pyramids/README.md)** — How ancient Egyptian engineers achieved impossible precision through methodical design, geometry, and measurement. ASCII diagrams included. (Yes, this is a software engineering repo—but the principles are timeless.)

**[📘 TypeScript: null vs undefined](./typescript/null-vs-undefined/README.md)** — The toilet paper metaphor that finally makes the difference stick. Includes memes.

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
The full-stack React framework explained with the restaurant kitchen metaphor (from rendering modes to API routes).

**[🏗️ NestJS](./nestjs/README.md)**
Angular's architecture for the backend — bringing structure, scalability, and sanity to Node.js with decorators, dependency injection, and modules.

**[📘 TypeScript](./typescript/README.md)**
Deep dives into tricky TypeScript concepts with memorable metaphors: null vs undefined (toilet paper), async/await (hot dog stand), and more.

### People-First Development

**[♿ Web Accessibility](./accessibility/README.md)**
Building for everyone: semantic HTML, keyboard navigation, screen readers, ARIA, color contrast, and accessible forms. Equity in practice.

### Responsible AI

**[🤝 Responsible AI](./responsible-ai/README.md)**
No Human Left Behind. A manifesto and framework for building AI that enhances human creativity without replacing human judgment — with the Developer-First Accountability Framework and practical implementation guides.

- **[Building with LLMs](./responsible-ai/llm/README.md)** - Security, guardrails, human-in-the-loop patterns, evaluation, and cost management for production LLM systems.

### Beyond Code

**[🏗️ Ancient Pyramids](./pyramids/README.md)**
How ancient builders achieved impossible precision through math, measurement, and methodical design, with ASCII diagrams showing the geometry.

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
├── nestjs/
│   └── README.md
├── typescript/
│   ├── README.md
│   ├── null-vs-undefined/
│   │   ├── README.md
│   │   └── null-undefined-meme.png
│   └── async-await/
│       └── README.md
├── pyramids/
│   └── README.md
├── accessibility/
│   └── README.md
├── responsible-ai/
│   ├── README.md
│   └── llm/
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

## 🚀 Projects in Practice

### [Human in the Loop](https://github.com/codewizwit/human-in-the-loop)

A production toolkit for centralized AI governance- bringing order to scattered prompts, agents, and AI tools across teams.

**What it does:** CLI for discovering, installing, and managing versioned AI prompts and agents with built-in quality controls and adoption tracking.

**Published:** [@human-in-the-loop/cli](https://www.npmjs.com/package/@human-in-the-loop/cli) on npm

**Technologies from this repo:**
- TypeScript (strict mode)
- Nx monorepo architecture
- pnpm workspace management
- npm publishing pipeline
- Jest for testing
- GitHub Actions for CI/CD

This project demonstrates the concepts documented here in a real-world, open-source package, and embodies the [Responsible AI](./responsible-ai/README.md) principles through its Developer-First Accountability Framework.

---

## 🧠 Themes

- **Responsible AI & human-centered technology**
- **Accessibility & inclusive design**
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
