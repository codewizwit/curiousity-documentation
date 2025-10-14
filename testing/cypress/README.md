# Testing with Cypress

_End-to-end testing that doesn't make you want to flip tables._

## Overview

Cypress is a modern testing framework built for the web. It runs in the browser alongside your app, giving you fast, reliable, and debuggable tests for real user workflows.

## Key Concepts

**Real Browser Testing**
Tests run in actual Chrome/Edge/Firefox, not simulated environments.

**Time Travel Debugging**
Hover over commands to see exactly what happened at each step. Snapshots make debugging visual.

**Automatic Waiting**
No more `sleep()` or arbitrary timeouts. Cypress waits for elements and assertions automatically.

**Network Stubbing**
Mock API responses to test edge cases without needing backend changes.

**Component Testing**
Test individual components in isolation, not just full page flows.

## Common Patterns

- Testing user authentication flows
- Form validation and submission
- API integration with fixtures
- Handling dynamic content and loading states
- Visual regression testing with snapshots
- CI integration for automated test runs

---

## Passing Secrets from GitHub Actions to Cypress

When running E2E tests in CI, you often need to pass sensitive data (API keys, credentials, tokens) to Cypress without hardcoding them. Here's how to do it securely.

### Method 1: Environment Variables (Recommended)

GitHub Actions secrets can be passed as environment variables that Cypress automatically picks up.

**GitHub Actions Workflow:**

```yaml
name: E2E Tests

on: [push, pull_request]

jobs:
  cypress:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run Cypress tests
        uses: cypress-io/github-action@v6
        env:
          # Cypress automatically picks up CYPRESS_* env vars
          CYPRESS_API_KEY: ${{ secrets.API_KEY }}
          CYPRESS_BASE_URL: ${{ secrets.BASE_URL }}
          CYPRESS_TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          CYPRESS_TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
```

**Accessing in Cypress Tests:**

```javascript
// cypress/e2e/api-test.cy.js
describe('API Integration', () => {
  it('authenticates with API', () => {
    cy.request({
      method: 'POST',
      url: `${Cypress.env('BASE_URL')}/api/login`,
      body: {
        email: Cypress.env('TEST_USER_EMAIL'),
        password: Cypress.env('TEST_USER_PASSWORD')
      },
      headers: {
        'X-API-Key': Cypress.env('API_KEY')
      }
    }).then((response) => {
      expect(response.status).to.eq(200);
    });
  });
});
```

**Why this works:**
- Any environment variable prefixed with `CYPRESS_` is automatically available via `Cypress.env()`
- Secrets never touch your codebase
- Works locally and in CI

---

### Method 2: cypress.config.js with Environment Variables

For more control, configure Cypress to read environment variables explicitly.

**cypress.config.js:**

```javascript
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  e2e: {
    baseUrl: process.env.CYPRESS_BASE_URL || 'http://localhost:3000',
    env: {
      apiKey: process.env.CYPRESS_API_KEY,
      apiUrl: process.env.CYPRESS_API_URL,
      testUser: {
        email: process.env.CYPRESS_TEST_USER_EMAIL,
        password: process.env.CYPRESS_TEST_USER_PASSWORD
      }
    },
    setupNodeEvents(on, config) {
      // Override config values from environment
      config.env.apiKey = process.env.CYPRESS_API_KEY;
      return config;
    }
  }
});
```

**GitHub Actions Workflow:**

```yaml
- name: Run Cypress tests
  run: npm run cypress:run
  env:
    CYPRESS_API_KEY: ${{ secrets.API_KEY }}
    CYPRESS_API_URL: ${{ secrets.API_URL }}
    CYPRESS_TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
    CYPRESS_TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
```

**Accessing in Tests:**

```javascript
describe('Login Flow', () => {
  it('logs in with test user', () => {
    cy.visit('/login');
    cy.get('input[name="email"]').type(Cypress.env('testUser').email);
    cy.get('input[name="password"]').type(Cypress.env('testUser').password);
    cy.get('button[type="submit"]').click();
    cy.url().should('include', '/dashboard');
  });
});
```

---

### Method 3: Dynamic cypress.env.json Generation

If you prefer using `cypress.env.json` but don't want to commit secrets, generate it dynamically in CI.

**GitHub Actions Workflow:**

```yaml
- name: Create Cypress env file
  run: |
    cat > cypress.env.json << EOF
    {
      "apiKey": "${{ secrets.API_KEY }}",
      "baseUrl": "${{ secrets.BASE_URL }}",
      "testUser": {
        "email": "${{ secrets.TEST_USER_EMAIL }}",
        "password": "${{ secrets.TEST_USER_PASSWORD }}"
      }
    }
    EOF

- name: Run Cypress tests
  uses: cypress-io/github-action@v6
```

**.gitignore:**

```
cypress.env.json
```

**Accessing in Tests:**

```javascript
describe('E2E Test', () => {
  it('uses secrets from env file', () => {
    const apiKey = Cypress.env('apiKey');
    const testUser = Cypress.env('testUser');

    cy.log(`Using API key: ${apiKey.substring(0, 4)}...`);
    cy.visit('/login');
    cy.get('input[name="email"]').type(testUser.email);
    // ... rest of test
  });
});
```

---

### Method 4: Using Cypress Cloud (Cypress Dashboard)

Cypress Cloud can store environment variables securely.

**Set in Cypress Cloud:**
1. Go to Project Settings → Environment Variables
2. Add key-value pairs (e.g., `API_KEY`, `TEST_USER_PASSWORD`)

**GitHub Actions Workflow:**

```yaml
- name: Run Cypress tests with Cloud
  uses: cypress-io/github-action@v6
  with:
    record: true
  env:
    CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
    # Cloud env vars are automatically injected
```

**Benefits:**
- Centralized secret management
- Accessible across multiple projects
- Test recordings and debugging insights

---

### Best Practices

**1. Never Log Secrets**
```javascript
// ❌ BAD - Logs secret to console
cy.log(Cypress.env('API_KEY'));

// ✅ GOOD - Redact sensitive parts
const apiKey = Cypress.env('API_KEY');
cy.log(`Using API key: ${apiKey.substring(0, 4)}***`);
```

**2. Use Different Secrets per Environment**
```yaml
jobs:
  test-staging:
    steps:
      - run: npm run cypress:run
        env:
          CYPRESS_API_KEY: ${{ secrets.STAGING_API_KEY }}

  test-production:
    steps:
      - run: npm run cypress:run
        env:
          CYPRESS_API_KEY: ${{ secrets.PROD_API_KEY }}
```

**3. Fail Fast on Missing Secrets**
```javascript
// cypress.config.js
setupNodeEvents(on, config) {
  const requiredEnvVars = ['API_KEY', 'BASE_URL', 'TEST_USER_EMAIL'];

  for (const varName of requiredEnvVars) {
    if (!process.env[`CYPRESS_${varName}`]) {
      throw new Error(`Missing required environment variable: CYPRESS_${varName}`);
    }
  }

  return config;
}
```

**4. Use GitHub Environment Secrets for Production**

For production deployments, use GitHub Environments with required reviewers:

```yaml
jobs:
  test-production:
    runs-on: ubuntu-latest
    environment: production  # Requires approval
    steps:
      - run: npm run cypress:run
        env:
          CYPRESS_API_KEY: ${{ secrets.PROD_API_KEY }}
```

---

### Common Pitfalls

**❌ Committing cypress.env.json**
Always add it to `.gitignore`.

**❌ Using secrets in test titles**
```javascript
// BAD - Secret appears in test output
it(`tests with key ${Cypress.env('API_KEY')}`, () => {});

// GOOD
it('tests API authentication', () => {});
```

**❌ Not handling missing env vars gracefully**
Always check if required secrets exist before running tests.

---

### Example: Complete E2E Test with Secrets

**GitHub Actions:**
```yaml
name: E2E Tests

on: [push]

jobs:
  cypress:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci

      - name: Run Cypress
        uses: cypress-io/github-action@v6
        env:
          CYPRESS_API_KEY: ${{ secrets.API_KEY }}
          CYPRESS_BASE_URL: https://staging.example.com
          CYPRESS_TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          CYPRESS_TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
```

**cypress.config.js:**
```javascript
module.exports = defineConfig({
  e2e: {
    baseUrl: process.env.CYPRESS_BASE_URL,
    env: {
      apiKey: process.env.CYPRESS_API_KEY,
      testUser: {
        email: process.env.CYPRESS_TEST_USER_EMAIL,
        password: process.env.CYPRESS_TEST_USER_PASSWORD
      }
    }
  }
});
```

**Test file:**
```javascript
describe('Authenticated User Flow', () => {
  beforeEach(() => {
    // Login using test credentials
    cy.request({
      method: 'POST',
      url: '/api/auth/login',
      body: {
        email: Cypress.env('testUser').email,
        password: Cypress.env('testUser').password
      },
      headers: {
        'X-API-Key': Cypress.env('apiKey')
      }
    }).then((response) => {
      window.localStorage.setItem('token', response.body.token);
    });
  });

  it('accesses protected dashboard', () => {
    cy.visit('/dashboard');
    cy.contains('Welcome back').should('be.visible');
  });
});
```

---

## Resources

- [Cypress Documentation](https://docs.cypress.io)
- [Best Practices](https://docs.cypress.io/guides/references/best-practices)
- [Cypress Cloud](https://www.cypress.io/cloud) for test recording and debugging
- [Cypress Environment Variables](https://docs.cypress.io/guides/guides/environment-variables)
- [GitHub Actions Cypress](https://github.com/cypress-io/github-action)

---

_Writing tests that actually help you ship with confidence._
