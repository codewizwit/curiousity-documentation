# GitHub Actions

_Understanding CI/CD workflows and automation on GitHub._

## Overview

GitHub Actions provides a powerful platform for automating software workflows directly in your repository. From CI/CD pipelines to scheduled tasks and event-driven automation.

## Key Concepts

**Workflows**
YAML files that define automated processes triggered by events (push, PR, schedule, etc.)

**Jobs & Steps**
Jobs run in parallel by default; steps execute sequentially within a job.

**Runners**
Virtual machines (or self-hosted) that execute your workflow jobs.

**Actions**
Reusable units of code that can be shared across workflows. Available from the GitHub Marketplace or custom-built.

## Common Patterns

- Running tests on every PR
- Building and deploying on merge to main
- Matrix builds across multiple Node/OS versions
- Caching dependencies for faster builds
- Conditional job execution based on branch or file changes

## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)
- [Workflow syntax reference](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions)

---

_Exploring workflows that make development faster and more reliable._
