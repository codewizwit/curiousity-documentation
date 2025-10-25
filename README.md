# 🌭 Curiosity Documentation
_A living notebook where curiosity meets code._

**New here?** → Start with [The Archives](./archives/README.md) to see where this all began, or jump straight to [The Greatest Hits](#-start-here-the-greatest-hits).

Welcome to my curiosity lab: part experiment, part documentation binge, part "wait, how does that actually work?" moment.

Every folder here started because I went down a rabbit hole I couldn't stop thinking about. From **handwritten bootcamp notebooks** (2020) trying to understand JavaScript's event loop, to production docs on **AWS Lambda**, **REST APIs**, and **responsible AI governance**. From **TypeScript brain-benders** to the **geometry behind ancient pyramids** (yes, really).

This is where I document until things make sense.
It's not perfect, but it's honest, and usually a little weird (in the best way).

**The throughline?** Five years of the same instinct: If I'm confused, I write it down. If I figure it out, I document it. If I build it, I explain it like I'm five.

---

## 🤓 Why This Exists

Because writing is my debugging tool.
Because documentation is how I think.
And because curiosity is the best dev skill no one puts on their résumé.

**This habit started in bootcamp**: filling notebooks with diagrams, question marks in the margins, crossed-out explanations where I got it wrong. When async JavaScript finally clicked, I documented that moment. When MongoDB schema design made sense, I wrote it down. When JWTs stopped being magic, I captured the "aha."

Five years later, I still do the same thing. Just with better tools (and legible handwriting, since I type now).

This repo is learning in public. Mistakes documented. Confusion captured. Breakthroughs explained.

If you've ever Googled something and wished the explanation was written by a human instead of a robot, welcome home.

<div align="center">
  <img src="https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExdm01dTNwdGc4YTNqbmJwMmxxeGRsNWNoajhqanJ4bG53YXJsMXQ2MyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/N7aAkvk6lSpYoLaAEi/giphy.gif" alt="Why don't you explain this to me like I'm five" width="500">
  <p><em>My documentation philosophy in a nutshell.</em></p>
</div>

---

## 🧭 Topics of Obsession

### 🎯 Start Here (The Greatest Hits)

| Topic | What's Inside |
|-------|---------------|
| **[📓 The Archives](./archives/README.md)** | My bootcamp notebooks from 2020-2021. **34 handwritten pages** from "What is a function?" to production apps. The "class thursday march 4" MongoDB breakthrough, algorithm revelations, and JavaScript event loop diagrams with question marks everywhere. Where my documentation habbit started. |
| **[🤝 Responsible AI](./responsible-ai/README.md)** | _No Human Left Behind._ Building AI that puts people first. The Developer-First Accountability Framework, human-in-the-loop patterns, and why human dignity is non-negotiable. Plus [LLM security & guardrails](./responsible-ai/llm/README.md). |
| **[♿ Accessibility](./accessibility/README.md)** | Building for everyone, not just some. Semantic HTML, keyboard nav, screen readers, ARIA, and why universal design makes everything better. |
| **[🏗️ Ancient Pyramids](./pyramids/README.md)** | The original full-stack architecture project. Geometry, measurement, and how ancient engineers achieved impossible precision. ASCII diagrams included. |
| **[📘 TypeScript: null vs undefined](./typescript/null-vs-undefined/README.md)** | The toilet paper metaphor that finally made it stick. Includes memes. |

---

### 💻 Frontend & Frameworks

| Topic | What I'm Figuring Out |
|-------|----------------------|
| **[⚡ Angular](./angular/README.md)** | Signals, computed, effect: the whiteboard metaphor for Angular's new reactivity system |
| **[⚛️ React](./react/README.md)** | Components, hooks, and the LEGO metaphor. Building UIs one block at a time |
| **[🚀 Next.js](./nextjs/README.md)** | The restaurant kitchen metaphor for SSR, SSG, and rendering modes |
| **[🏗️ Angular + Nx](./angular-nx/README.md)** | Monorepo architecture, dependency graphs, and computation caching |
| **[🔌 Module Federation](./module-federation/README.md)** | Micro-frontends without the micro-headaches. Dynamic remote loading, shared dependencies, and independent deployment |

---

### 🔧 Backend & Infrastructure

| Topic | What I'm Figuring Out |
|-------|----------------------|
| **[🏗️ NestJS](./nestjs/README.md)** | Angular's architecture for the backend. Dependency injection meets Node.js |
| **[☁️ AWS](./aws/README.md)** | Cloud computing that scales. [Lambda](./aws/lambda/README.md) (serverless functions), [IAM](./aws/iam/README.md) (who can do what), [CloudFormation](./aws/cloudformation/README.md) (infrastructure as code) |
| **[🌐 REST APIs](./rest-apis/README.md)** | Designing interfaces that make sense. Resources, HTTP methods, status codes, and why nouns > verbs |
| **[🍃 MongoDB](./mongodb/README.md)** | The filing cabinet vs storage bins metaphor. NoSQL, schemas, and when to embed vs reference |
| **[🔐 JWTs](./jwts/README.md)** | Decoding, verifying, and the passport metaphor for understanding bearer tokens |

---

### 🛠️ DevOps & Tooling

| Topic | What I'm Figuring Out |
|-------|----------------------|
| **[⚙️ GitHub Actions](./github-actions/README.md)** | Reusable workflows, coverage uploads, secrets encryption, and artifact sorcery |
| **[📦 pnpm](./pnpm/README.md)** | Workspace linking, hoisting, and why `node_modules` deserves therapy |
| **[📦 npm](./npm/README.md)** | Publishing packages without breaking prod (featuring human-in-the-loop) |

---

### 🧪 Testing & Quality

| Topic | What I'm Figuring Out |
|-------|----------------------|
| **[🧪 Cypress](./testing/cypress/README.md)** | E2E testing patterns and passing secrets from GitHub Actions |
| **[🎭 Playwright](./testing/playwright/README.md)** | Model-driven development, AI test generation, and intelligent automation |
| **[🃏 Jest](./testing/jest/README.md)** | Unit testing, mocking strategies, and actually understanding coverage |
| **[📊 Code Coverage](./testing/code-coverage/README.md)** | What the metrics mean and when 100% is a trap |
| **[🧬 Mutation Testing](./testing/stryker/README.md)** | Testing the tests. Who tests the tests? Mutants do. |

---

### 📘 Languages & Fundamentals

| Topic | What I'm Figuring Out |
|-------|----------------------|
| **[📘 TypeScript](./typescript/README.md)** | null vs undefined (toilet paper), async/await (hot dog stand), and brain-benders with types |

---

## 🔍 How I Actually Use This

- **Reference when I forget**: "Wait, how do Lambda cold starts work again?" → search this repo, find my own explanation, remember why I wrote it that way
- **Onboarding teammates**: "Here's how GitHub Actions secrets work" → link to docs instead of explaining for the 5th time
- **Debugging amnesia**: Past-me documented the solution. Present-me just needs to remember where I put it
- **Proof I'm learning**: Tracking how my understanding evolved from "what's a closure?" (2020 notebooks) to "here's how to optimize serverless architecture" (production docs)
- **Interview prep**: Turns out documenting everything you learn is the best study guide you'll ever make

Basically: if I took the time to figure it out, I document it. Future-me always thanks past-me.

---

## 🚀 Real Projects (Where This Gets Used)

These docs aren't just theoretical. They're built from real production work, informing how I build and ship software.

**[Human in the Loop](https://github.com/codewizwit/human-in-the-loop)**

A production toolkit for centralized AI governance, bringing order to scattered prompts, agents, and AI tools across teams.

**What it does:** CLI for discovering, installing, and managing versioned AI prompts and agents with built-in quality controls and adoption tracking.

**Published:** [@human-in-the-loop/cli](https://www.npmjs.com/package/@human-in-the-loop/cli) on npm

**Technologies from this repo:**
- TypeScript (strict mode)
- Nx monorepo architecture
- pnpm workspace management
- npm publishing pipeline
- Jest for testing
- GitHub Actions for CI/CD

This project lives the concepts documented here: Nx monorepos, pnpm workspaces, npm publishing, and the [Responsible AI](./responsible-ai/README.md) principles through its Developer-First Accountability Framework.

---

### Quick Navigation

```
curiosity-documentation/
├── accessibility/
│   └── README.md
├── angular/
│   ├── README.md
│   ├── computed/
│   │   └── README.md
│   ├── effect/
│   │   └── README.md
│   └── signals/
│       └── README.md
├── angular-nx/
│   └── README.md
├── archives/
│   ├── README.md
│   ├── algorithms-problem-solving/
│   │   ├── README.md
│   │   └── [5 handwritten pages]
│   ├── backend-auth-cloud/
│   │   ├── README.md
│   │   └── [2 handwritten pages]
│   ├── javascript/
│   │   ├── README.md
│   │   └── [11 handwritten pages]
│   ├── mongodb-learning-journey/
│   │   ├── README.md
│   │   └── [9 handwritten pages]
│   └── orm-mvc-architecture/
│       ├── README.md
│       └── [7 handwritten pages]
├── aws/
│   ├── README.md
│   ├── cloudformation/
│   │   └── README.md
│   ├── iam/
│   │   └── README.md
│   └── lambda/
│       └── README.md
├── github-actions/
│   └── README.md
├── jwts/
│   └── README.md
├── module-federation/
│   └── README.md
├── mongodb/
│   └── README.md
├── nestjs/
│   └── README.md
├── nextjs/
│   └── README.md
├── npm/
│   └── README.md
├── pnpm/
│   └── README.md
├── pyramids/
│   └── README.md
├── react/
│   └── README.md
├── rest-apis/
│   └── README.md
├── responsible-ai/
│   ├── README.md
│   └── llm/
│       └── README.md
├── testing/
│   ├── code-coverage/
│   │   └── README.md
│   ├── cypress/
│   │   └── README.md
│   ├── jest/
│   │   └── README.md
│   ├── playwright/
│   │   └── README.md
│   └── stryker/
│       └── README.md
└── typescript/
    ├── README.md
    ├── async-await/
    │   └── README.md
    └── null-vs-undefined/
        ├── README.md
        └── null-undefined-meme.png
```

---

## 🪩 Credits

Written by a small human with a big appetite for learning.
Powered by snacks, curiosity, and the occasional assist from modern AI tools.

Created and maintained by **Alexandra Kelstrom** ([@codewizwit](https://github.com/codewizwit)) 🌭

While this is primarily a personal archive, thoughtful issues or discussions are welcome, especially if you spot something worth clarifying or expanding.

---

> _"Document your curiosity. The rest is just syntax."_ 🌭
