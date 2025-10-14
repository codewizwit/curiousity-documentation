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

## Resources

- [Playwright Documentation](https://playwright.dev)
- [Best Practices](https://playwright.dev/docs/best-practices)
- [API Reference](https://playwright.dev/docs/api/class-playwright)
- [VS Code Extension](https://marketplace.visualstudio.com/items?itemName=ms-playwright.playwright)

---

_Testing web apps the way users actually use them._
