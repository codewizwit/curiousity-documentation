# GitHub Actions

_Understanding CI/CD workflows and automation on GitHub._

## Overview

GitHub Actions provides a powerful platform for automating software workflows directly in your repository. From CI/CD pipelines to scheduled tasks and event-driven automation. Once you go beyond the basics, you unlock some seriously powerful patterns for building reusable, dynamic workflows.

---

## Key Concepts

### Workflows
YAML files that define automated processes triggered by events (push, PR, schedule, etc.)

### Jobs & Steps
Jobs run in parallel by default; steps execute sequentially within a job.

### Runners
Virtual machines (or self-hosted) that execute your workflow jobs.

### Actions
Reusable units of code that can be shared across workflows. Available from the GitHub Marketplace or custom-built.

---

## Advanced Patterns & Techniques

### 1. Reusable Workflows (Callable Workflows)

Instead of copying workflow YAML across repos, create reusable workflows that can be called like functions.

**Creating a Reusable Workflow**

`.github/workflows/reusable-e2e.yml`:
```yaml
name: Reusable E2E Tests

on:
  workflow_call:
    inputs:
      environment:
        description: 'Environment to test against'
        required: true
        type: string
      test-suite:
        description: 'Which test suite to run'
        required: false
        type: string
        default: 'all'
      browser:
        description: 'Browser to use'
        required: false
        type: string
        default: 'chrome'
    secrets:
      API_KEY:
        required: true
    outputs:
      test-result:
        description: 'Test run result'
        value: ${{ jobs.e2e.outputs.result }}

jobs:
  e2e:
    runs-on: ubuntu-latest
    outputs:
      result: ${{ steps.test.outputs.result }}
    steps:
      - uses: actions/checkout@v4

      - name: Run E2E Tests
        id: test
        run: |
          echo "Running ${{ inputs.test-suite }} tests on ${{ inputs.environment }}"
          npm run test:e2e -- --env=${{ inputs.environment }} --browser=${{ inputs.browser }}
          echo "result=success" >> $GITHUB_OUTPUT
        env:
          API_KEY: ${{ secrets.API_KEY }}
```

**Calling the Reusable Workflow**

`.github/workflows/main.yml`:
```yaml
name: CI Pipeline

on: [push, pull_request]

jobs:
  test-staging:
    uses: ./.github/workflows/reusable-e2e.yml
    with:
      environment: staging
      test-suite: smoke
      browser: chrome
    secrets:
      API_KEY: ${{ secrets.STAGING_API_KEY }}

  test-production:
    uses: ./.github/workflows/reusable-e2e.yml
    with:
      environment: production
      test-suite: critical
      browser: firefox
    secrets:
      API_KEY: ${{ secrets.PROD_API_KEY }}
```

**Why This Rocks:**
- DRY principle for workflows
- Single source of truth for complex testing logic
- Easy to maintain and update
- Can be called from different repos if made public

---

### 2. Dynamic Matrix Strategies

Build dynamic test matrices based on runtime conditions.

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - uses: actions/checkout@v4

      - name: Generate Dynamic Matrix
        id: set-matrix
        run: |
          # Detect which apps changed
          if git diff --name-only HEAD~1 | grep -q "^apps/web/"; then
            MATRIX='{"app": ["web"], "browser": ["chrome", "firefox"]}'
          elif git diff --name-only HEAD~1 | grep -q "^apps/mobile/"; then
            MATRIX='{"app": ["mobile"], "device": ["ios", "android"]}'
          else
            MATRIX='{"app": ["web", "mobile"]}'
          fi
          echo "matrix=$MATRIX" >> $GITHUB_OUTPUT

  test:
    needs: setup
    runs-on: ubuntu-latest
    strategy:
      matrix: ${{ fromJson(needs.setup.outputs.matrix) }}
    steps:
      - name: Test ${{ matrix.app }}
        run: echo "Testing ${{ matrix.app }}"
```

**Use Cases:**
- Only test changed services in a monorepo
- Scale test runs based on branch or PR labels
- Conditionally run different browser combinations

---

### 3. Using @actions/core for Custom Actions

When building custom JavaScript actions, `@actions/core` provides utilities for inputs, outputs, logging, and error handling.

**Example Custom Action**

`action.yml`:
```yaml
name: 'Coverage to Release Notes'
description: 'Appends test coverage to release notes'
inputs:
  coverage-file:
    description: 'Path to coverage summary'
    required: true
  release-tag:
    description: 'Release tag to update'
    required: true
outputs:
  updated:
    description: 'Whether release notes were updated'
runs:
  using: 'node20'
  main: 'index.js'
```

`index.js`:
```javascript
const core = require('@actions/core');
const github = require('@actions/github');
const fs = require('fs');

async function run() {
  try {
    // Get inputs
    const coverageFile = core.getInput('coverage-file', { required: true });
    const releaseTag = core.getInput('release-tag', { required: true });
    const token = core.getInput('github-token', { required: true });

    // Read coverage data
    const coverage = JSON.parse(fs.readFileSync(coverageFile, 'utf8'));

    // Format coverage summary
    const summary = `
## Test Coverage
- **Lines:** ${coverage.total.lines.pct}%
- **Branches:** ${coverage.total.branches.pct}%
- **Functions:** ${coverage.total.functions.pct}%
- **Statements:** ${coverage.total.statements.pct}%
    `;

    // Update release notes
    const octokit = github.getOctokit(token);
    const { owner, repo } = github.context.repo;

    const release = await octokit.rest.repos.getReleaseByTag({
      owner,
      repo,
      tag: releaseTag
    });

    await octokit.rest.repos.updateRelease({
      owner,
      repo,
      release_id: release.data.id,
      body: release.data.body + '\n\n' + summary
    });

    core.info('Successfully updated release notes with coverage');
    core.setOutput('updated', 'true');

  } catch (error) {
    core.setFailed(`Action failed: ${error.message}`);
  }
}

run();
```

**Key @actions/core Functions:**
- `core.getInput()` - Read action inputs
- `core.setOutput()` - Set action outputs
- `core.setFailed()` - Mark action as failed
- `core.info()`, `core.warning()`, `core.error()` - Logging
- `core.startGroup()` / `core.endGroup()` - Collapsible log sections
- `core.exportVariable()` - Set environment variables
- `core.setSecret()` - Mask sensitive values in logs

---

### 4. Secrets and Encryption

#### Using Encrypted Secrets

GitHub encrypts secrets using Libsodium sealed boxes. Secrets are encrypted in transit and at rest.

**Setting Secrets:**
```bash
# Repository secrets
Settings → Secrets and variables → Actions → New repository secret

# Environment secrets (for specific deployment environments)
Settings → Environments → [Environment Name] → Add secret
```

**Using Secrets in Workflows:**
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy
        run: ./deploy.sh
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

#### Encrypted Secrets in Files

For larger secrets (like service account keys), use base64 encoding:

```yaml
steps:
  - name: Create credentials file
    run: |
      echo "${{ secrets.GCP_SERVICE_ACCOUNT }}" | base64 -d > credentials.json

  - name: Use credentials
    run: gcloud auth activate-service-account --key-file=credentials.json
```

#### Secret Scanning Protection

GitHub automatically scans for leaked secrets. Best practices:
- Never log secrets (`echo $SECRET`)
- Use `core.setSecret()` to mask values
- Rotate secrets if accidentally committed
- Use environment-specific secrets

---

### 5. Writing Coverage to Release Notes

**Full Workflow Example:**

```yaml
name: Release with Coverage

on:
  release:
    types: [published]

jobs:
  test-and-update-release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run tests with coverage
        run: npm test -- --coverage --coverageReporters=json-summary

      - name: Extract coverage
        id: coverage
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json)
          LINES=$(echo $COVERAGE | jq -r '.total.lines.pct')
          BRANCHES=$(echo $COVERAGE | jq -r '.total.branches.pct')
          FUNCTIONS=$(echo $COVERAGE | jq -r '.total.functions.pct')
          STATEMENTS=$(echo $COVERAGE | jq -r '.total.statements.pct')

          echo "lines=$LINES" >> $GITHUB_OUTPUT
          echo "branches=$BRANCHES" >> $GITHUB_OUTPUT
          echo "functions=$FUNCTIONS" >> $GITHUB_OUTPUT
          echo "statements=$STATEMENTS" >> $GITHUB_OUTPUT

      - name: Update release notes
        uses: actions/github-script@v7
        with:
          script: |
            const release = await github.rest.repos.getReleaseByTag({
              owner: context.repo.owner,
              repo: context.repo.repo,
              tag: context.ref.replace('refs/tags/', '')
            });

            const coverageSection = `

            ## 📊 Test Coverage
            | Metric | Coverage |
            |--------|----------|
            | Lines | ${{ steps.coverage.outputs.lines }}% |
            | Branches | ${{ steps.coverage.outputs.branches }}% |
            | Functions | ${{ steps.coverage.outputs.functions }}% |
            | Statements | ${{ steps.coverage.outputs.statements }}% |
            `;

            await github.rest.repos.updateRelease({
              owner: context.repo.owner,
              repo: context.repo.repo,
              release_id: release.data.id,
              body: release.data.body + coverageSection
            });

            core.info('✅ Release notes updated with coverage data');
```

**Alternative: Create Coverage Badge**

```yaml
- name: Generate coverage badge
  uses: cicirello/jacoco-badge-generator@v2
  with:
    badges-directory: badges
    generate-summary: true

- name: Commit badge
  run: |
    git config user.name github-actions
    git config user.email github-actions@github.com
    git add badges/
    git commit -m "Update coverage badge [skip ci]" || exit 0
    git push
```

---

## Real-World Patterns

### Parallel E2E Tests with Sharding

```yaml
jobs:
  e2e:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4

      - name: Run E2E Shard
        run: |
          npm run test:e2e -- --shard=${{ matrix.shard }}/4
```

### Conditional Deployments

```yaml
jobs:
  deploy:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        run: ./deploy.sh
```

### Dependency Caching

```yaml
- name: Cache node modules
  uses: actions/cache@v4
  with:
    path: node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

### Matrix Strategy with Includes/Excludes

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: [18, 20]
    exclude:
      - os: macos-latest
        node: 18
    include:
      - os: ubuntu-latest
        node: 20
        experimental: true
```

---

## Common Gotchas

**1. Secrets in Pull Requests from Forks**
Secrets are NOT available to workflows triggered by PRs from forks (security measure). Use `pull_request_target` carefully.

**2. Workflow Permissions**
By default, workflows have limited permissions. Grant explicit permissions:
```yaml
permissions:
  contents: write
  pull-requests: write
```

**3. Step Outputs vs Job Outputs**
Step outputs must be explicitly exposed as job outputs to be used in other jobs.

**4. YAML Gotchas**
- Use quotes around strings with special chars (`"${{ secrets.KEY }}"`)
- `if` conditions use expressions, not bash syntax (`if: github.ref == 'refs/heads/main'`)

---

## Debugging Workflows

### Enable Debug Logging
Set repository secrets:
- `ACTIONS_STEP_DEBUG=true` - Detailed step logs
- `ACTIONS_RUNNER_DEBUG=true` - Runner diagnostic logs

### Use Workflow Dispatch for Testing
```yaml
on:
  workflow_dispatch:
    inputs:
      debug:
        description: 'Enable debug mode'
        required: false
        type: boolean
```

### Tmate for SSH Access
```yaml
- name: Setup tmate session
  if: failure()
  uses: mxschmitt/action-tmate@v3
```

---

## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Reusing Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [@actions/core NPM package](https://www.npmjs.com/package/@actions/core)
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)
- [Workflow syntax reference](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions)
- [Encrypted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

---

_Building workflows that are powerful, reusable, and maintainable._
