# Testing with Playwright

_Modern end-to-end testing for the modern web._

## Overview

Playwright is Microsoft's answer to reliable browser automation. It's fast, powerful, and built for testing modern web apps across all major browsers (Chromium, Firefox, WebKit). Think of it as the Swiss Army knife of E2E testing.

## Key Concepts

**Cross-Browser Testing**
Test in Chrome, Firefox, and Safari (WebKit) without changing your code. Run them all in parallel.

**Auto-Waiting**
Like Cypress, Playwright automatically waits for elements to be ready before interacting with them. No more flaky tests from timing issues.

**Multiple Contexts**
Test different user sessions simultaneously — like having multiple incognito windows open at once. Great for testing multi-user features.

**Network Interception**
Mock API responses, test offline scenarios, or verify what requests your app makes.

**Screenshots & Videos**
Automatic visual evidence when tests fail. Debug without reproducing the issue locally.

## Why Playwright?

- **Faster execution** than Selenium and often Cypress
- **Better mobile emulation** for responsive testing
- **Powerful selectors** including text, CSS, and accessibility tree
- **Built-in test runner** with parallel execution
- **Great TypeScript support** out of the box

## Common Patterns

- Testing authentication flows across different browser engines
- Mobile responsive testing with device emulation
- Visual regression testing with screenshots
- Testing file uploads and downloads
- Handling multiple tabs and pop-ups
- Parallel test execution for faster CI

## Playwright vs Cypress

Both are excellent. Key differences:

| Feature | Playwright | Cypress |
|---------|-----------|---------|
| **Browsers** | Chrome, Firefox, Safari | Chrome-family only |
| **Tabs/Windows** | Native support | Limited |
| **Speed** | Faster (generally) | Fast |
| **Mobile** | Better emulation | Basic viewport |
| **Learning Curve** | Steeper | Gentler |
| **Debugging** | Good | Excellent |

Choose Playwright if you need cross-browser testing or complex scenarios. Choose Cypress for developer experience and debugging.

## Getting Started Example

```javascript
import { test, expect } from '@playwright/test';

test('user can login', async ({ page }) => {
  await page.goto('https://example.com/login');
  await page.fill('input[name="email"]', 'user@example.com');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button[type="submit"]');

  await expect(page).toHaveURL('https://example.com/dashboard');
  await expect(page.locator('h1')).toContainText('Welcome');
});
```

---

## Model-Driven Development (MDD) with Playwright

Playwright supports Model-Driven Development through AI-powered test generation, code generation capabilities, and intelligent test maintenance. This approach allows you to generate tests from user interactions, recordings, or specifications, making test creation faster and more maintainable.

### Why Model-Driven Development Matters

MDD in Playwright enables:
- **Faster test creation** - Generate tests from recordings or specifications
- **AI-powered test generation** - Use AI agents to write tests based on natural language descriptions
- **Intelligent locators** - Auto-generate resilient selectors that survive UI changes
- **Test maintenance** - Update tests automatically when UI changes
- **Codegen capabilities** - Record user interactions and generate test code

This shifts testing from manual coding to declarative test specifications.

---

### Method 1: Playwright Codegen - Recording Tests

The most straightforward MDD approach is using Playwright's built-in test recorder.

**Basic Codegen:**

```bash
# Record a new test
npx playwright codegen https://example.com

# Record with specific browser
npx playwright codegen --browser=webkit https://example.com

# Record with device emulation
npx playwright codegen --device="iPhone 13 Pro" https://example.com

# Save to specific file
npx playwright codegen --output tests/my-test.spec.ts https://example.com
```

**What You Get:**

As you interact with the page, Playwright generates test code in real-time:

```javascript
import { test, expect } from '@playwright/test';

test('user login flow', async ({ page }) => {
  // Auto-generated from your actions
  await page.goto('https://example.com/login');
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', { name: 'Sign in' }).click();

  // Auto-generated assertion
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
});
```

**Key Benefits:**
- Generates resilient locators (role-based, label-based)
- Captures user intent, not just DOM structure
- Creates assertions automatically when you assert visibility

---

### Method 2: Using Playwright Agents for AI-Powered Testing

Playwright can integrate with AI agents to generate tests from natural language specifications or user stories.

**AI-Driven Test Generation Pattern:**

```javascript
/**
 * User Story: As a customer, I want to add items to cart and checkout
 *
 * AI Agent generates:
 */

import { test, expect } from '@playwright/test';

test('complete purchase flow', async ({ page }) => {
  // AI understands the user intent and generates semantic selectors
  await page.goto('https://example.com/products');

  // Search for product
  await page.getByPlaceholder('Search products').fill('laptop');
  await page.getByRole('button', { name: /search/i }).click();

  // Add to cart - AI infers common patterns
  await page.getByRole('article').first().getByRole('button', { name: /add to cart/i }).click();

  // Navigate to cart
  await page.getByRole('link', { name: /cart/i }).click();

  // Verify item in cart
  await expect(page.getByRole('heading', { name: /shopping cart/i })).toBeVisible();
  await expect(page.getByText(/laptop/i)).toBeVisible();

  // Proceed to checkout
  await page.getByRole('button', { name: /checkout/i }).click();

  // AI generates smart assertions based on expected outcomes
  await expect(page).toHaveURL(/.*checkout/);
});
```

**Intelligent Locator Strategy:**

AI agents help generate locators that are:
- **Semantic** - Based on user-visible text and roles
- **Resilient** - Survive UI refactoring
- **Accessible** - Follow ARIA and accessibility best practices

```javascript
// Instead of brittle selectors:
await page.click('.btn-primary.submit-button');

// AI generates semantic selectors:
await page.getByRole('button', { name: 'Submit' }).click();
```

---

### Method 3: Test Generation from Specifications

Use Playwright with specification-driven testing to generate tests from structured requirements.

**BDD-Style Test Generation:**

```javascript
// features/checkout.feature
// Given I am a logged-in user
// When I add a product to cart
// And I proceed to checkout
// Then I should see the order confirmation

// Generated Playwright test:
import { test, expect } from '@playwright/test';

test.describe('Checkout Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Setup: Login
    await page.goto('https://example.com/login');
    await page.getByLabel('Email').fill('test@example.com');
    await page.getByLabel('Password').fill('password');
    await page.getByRole('button', { name: 'Login' }).click();
  });

  test('user can complete purchase', async ({ page }) => {
    // When: Add product to cart
    await page.goto('https://example.com/products');
    await page.getByRole('button', { name: 'Add to Cart' }).first().click();

    // When: Proceed to checkout
    await page.getByRole('link', { name: 'Cart' }).click();
    await page.getByRole('button', { name: 'Checkout' }).click();

    // Fill shipping info
    await page.getByLabel('Address').fill('123 Main St');
    await page.getByLabel('City').fill('San Francisco');
    await page.getByLabel('Zip').fill('94102');

    // Submit order
    await page.getByRole('button', { name: 'Place Order' }).click();

    // Then: Verify confirmation
    await expect(page.getByRole('heading', { name: /order confirmed/i })).toBeVisible();
    await expect(page.getByText(/thank you/i)).toBeVisible();
  });
});
```

---

### Method 4: Page Object Model with Code Generation

Generate maintainable Page Object Models automatically.

**Auto-Generated Page Objects:**

```javascript
// pages/LoginPage.ts - Generated from UI inspection
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    // AI-generated semantic locators
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Sign in' });
    this.errorMessage = page.getByRole('alert');
  }

  async goto() {
    await this.page.goto('https://example.com/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getErrorMessage() {
    return await this.errorMessage.textContent();
  }
}

// Usage in tests
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

test('login with invalid credentials', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await loginPage.goto();
  await loginPage.login('invalid@example.com', 'wrongpassword');

  const error = await loginPage.getErrorMessage();
  expect(error).toContain('Invalid credentials');
});
```

---

### Method 5: Self-Healing Tests with AI

Playwright agents can help tests self-heal when UI changes occur.

**Self-Healing Locator Strategy:**

```javascript
import { test, expect } from '@playwright/test';

test('resilient login test', async ({ page }) => {
  await page.goto('https://example.com/login');

  // Primary locator
  let loginButton = page.getByRole('button', { name: 'Sign in' });

  // If primary fails, AI suggests alternatives
  if (!await loginButton.isVisible({ timeout: 1000 }).catch(() => false)) {
    loginButton = page.getByRole('button', { name: 'Login' }) // Alternative 1
      .or(page.getByRole('button', { name: 'Submit' }))      // Alternative 2
      .or(page.locator('button[type="submit"]'));            // Fallback
  }

  await loginButton.click();
});
```

**Auto-Updating Tests:**

```javascript
// When UI changes are detected, generate updated locators
test('self-healing search', async ({ page }) => {
  await page.goto('https://example.com');

  // AI tracks multiple ways to find the search input
  const searchInput = page
    .getByPlaceholder('Search')
    .or(page.getByLabel('Search'))
    .or(page.getByRole('searchbox'))
    .or(page.locator('input[name="search"]'));

  await searchInput.fill('laptop');
});
```

---

## Real-World MDD Patterns

### Pattern 1: Recording User Journeys for Regression Testing

```bash
# Record critical user paths
npx playwright codegen --output tests/checkout-flow.spec.ts https://example.com

# Later, when UI changes, re-record and compare
npx playwright codegen --output tests/checkout-flow-v2.spec.ts https://example.com
```

### Pattern 2: Generating Tests from API Specifications

```javascript
// Given an OpenAPI spec, generate E2E tests
import { test, expect } from '@playwright/test';

// Auto-generated from API spec
test('user CRUD operations', async ({ page, request }) => {
  // Create user via API
  const response = await request.post('/api/users', {
    data: { name: 'Test User', email: 'test@example.com' }
  });
  const user = await response.json();

  // Verify in UI
  await page.goto(`/users/${user.id}`);
  await expect(page.getByText('Test User')).toBeVisible();

  // Update via UI
  await page.getByRole('button', { name: 'Edit' }).click();
  await page.getByLabel('Name').fill('Updated Name');
  await page.getByRole('button', { name: 'Save' }).click();

  // Verify via API
  const updated = await request.get(`/api/users/${user.id}`);
  const data = await updated.json();
  expect(data.name).toBe('Updated Name');
});
```

### Pattern 3: Visual Regression with AI Comparison

```javascript
test('homepage visual regression', async ({ page }) => {
  await page.goto('https://example.com');

  // AI-powered visual comparison
  await expect(page).toHaveScreenshot('homepage.png', {
    maxDiffPixels: 100,  // Allow minor differences
    threshold: 0.2       // AI adjusts for acceptable variations
  });
});
```

---

## Best Practices for MDD with Playwright

**1. Start with Codegen, Refactor Later**
- Use codegen to quickly capture user flows
- Refactor generated code into reusable functions or Page Objects
- Extract common patterns into custom fixtures

**2. Use Semantic Locators**
```javascript
// Good - Semantic and resilient
await page.getByRole('button', { name: 'Submit' });
await page.getByLabel('Email');
await page.getByText('Welcome back');

// Avoid - Brittle and implementation-dependent
await page.click('.btn-primary');
await page.fill('#email-input-field-123');
```

**3. Layer Your Test Generation**
- **Level 1**: Record with codegen for quick coverage
- **Level 2**: AI generates tests from user stories
- **Level 3**: Specification-driven tests from requirements
- **Level 4**: Self-healing tests that adapt to changes

**4. Combine Recording with Manual Assertions**
```javascript
test('generated with manual assertions', async ({ page }) => {
  // Auto-generated navigation
  await page.goto('https://example.com/products');
  await page.getByRole('link', { name: 'Laptops' }).click();

  // Manual business logic assertions
  const products = page.getByRole('article');
  const count = await products.count();
  expect(count).toBeGreaterThan(0);

  // Verify pricing
  const prices = await products.locator('.price').allTextContents();
  prices.forEach(price => {
    const amount = parseFloat(price.replace(/[^0-9.]/g, ''));
    expect(amount).toBeGreaterThan(0);
  });
});
```

**5. Version Control Generated Tests**
- Commit generated tests to git
- Track changes over time
- Use diffs to understand UI evolution

---

## Debugging MDD Tests

**Playwright Inspector:**
```bash
npx playwright test --debug
```

**Trace Viewer:**
```bash
npx playwright test --trace on
npx playwright show-trace trace.zip
```

**Headed Mode for Visual Feedback:**
```bash
npx playwright test --headed --slowMo=1000
```

---

## Resources

- [Playwright Documentation](https://playwright.dev)
- [Playwright Codegen Guide](https://playwright.dev/docs/codegen)
- [Best Practices](https://playwright.dev/docs/best-practices)
- [Test Generator](https://playwright.dev/docs/test-generator)
- [API Reference](https://playwright.dev/docs/api/class-playwright)
- [VS Code Extension](https://marketplace.visualstudio.com/items?itemName=ms-playwright.playwright)

---

_Testing web apps through intelligent test generation and AI-powered automation._
