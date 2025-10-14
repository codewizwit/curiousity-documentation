# Testing with Jest

_Delightful JavaScript testing._

## Overview

Jest is the de facto standard for testing JavaScript and TypeScript applications. Created by Meta (Facebook), it's a complete testing framework that works out of the box with minimal configuration. If you're writing unit or integration tests in the JS ecosystem, you're probably using Jest.

## Key Concepts

**Zero Config**
Works with most JavaScript projects right away. Sensible defaults mean you can start testing immediately.

**Fast & Parallel**
Jest runs tests in parallel across multiple workers, making large test suites finish quickly.

**Built-in Mocking**
Mock functions, modules, timers — everything you need without additional libraries.

**Snapshot Testing**
Capture component output and detect unexpected changes. Great for React components and data structures.

**Great Error Messages**
When tests fail, Jest tells you exactly what went wrong with helpful diffs.

## Common Testing Patterns

**Unit Tests**
Test individual functions or classes in isolation.
```javascript
test('adds two numbers correctly', () => {
  expect(add(2, 3)).toBe(5);
});
```

**Integration Tests**
Test how multiple units work together.
```javascript
test('user service creates and retrieves user', async () => {
  const user = await userService.create({ name: 'Alex' });
  const retrieved = await userService.findById(user.id);
  expect(retrieved.name).toBe('Alex');
});
```

**Mocking Dependencies**
Isolate what you're testing by mocking external dependencies.
```javascript
jest.mock('./api');
test('handles API errors gracefully', async () => {
  api.fetchUser.mockRejectedValue(new Error('Network error'));
  const result = await getUserData();
  expect(result.error).toBeTruthy();
});
```

**Testing Async Code**
Jest handles promises and async/await naturally.
```javascript
test('fetches user data', async () => {
  const data = await fetchUserData(123);
  expect(data.id).toBe(123);
});
```

## Best Practices

**Describe What You're Testing**
```javascript
describe('UserValidator', () => {
  describe('validateEmail', () => {
    test('accepts valid email addresses', () => {
      expect(validateEmail('user@example.com')).toBe(true);
    });

    test('rejects emails without @ symbol', () => {
      expect(validateEmail('notanemail')).toBe(false);
    });
  });
});
```

**Use Descriptive Test Names**
Good: `test('throws error when user not found')`
Bad: `test('error test')`

**Test Behavior, Not Implementation**
Focus on what the code does, not how it does it. This makes refactoring easier.

**Keep Tests Simple**
One test should verify one thing. If you're writing multiple assertions, consider splitting into multiple tests.

**Don't Mock Everything**
Only mock external dependencies (APIs, databases). Keep actual business logic unmocked.

## Useful Matchers

```javascript
// Equality
expect(value).toBe(4);                    // Strict equality (===)
expect(value).toEqual({ a: 1, b: 2 });    // Deep equality

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeDefined();

// Numbers
expect(value).toBeGreaterThan(3);
expect(value).toBeLessThanOrEqual(5);
expect(value).toBeCloseTo(0.3);           // Floating point

// Strings
expect(text).toMatch(/hello/i);
expect(text).toContain('world');

// Arrays
expect(array).toContain('item');
expect(array).toHaveLength(3);

// Objects
expect(obj).toHaveProperty('key', 'value');

// Functions
expect(fn).toThrow();
expect(fn).toHaveBeenCalled();
expect(fn).toHaveBeenCalledWith('arg');
```

## Coverage Tips

Run tests with coverage:
```bash
jest --coverage
```

Focus on meaningful coverage, not 100%:
- **High value**: Core business logic, data transformations, validation
- **Lower value**: Simple getters/setters, configuration files
- **Don't test**: Third-party libraries (trust they're tested)

## Jest with TypeScript

Jest works great with TypeScript via `ts-jest`:

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/*.test.ts'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
  ],
};
```

## Common Pitfalls

**Not Cleaning Up Between Tests**
Use `beforeEach` and `afterEach` to reset state:
```javascript
beforeEach(() => {
  jest.clearAllMocks();
});
```

**Testing Too Much at Once**
Break complex tests into smaller, focused tests.

**Ignoring Async Issues**
Always `await` or `return` promises in tests, or they'll pass even when they fail.

## Resources

- [Jest Documentation](https://jestjs.io)
- [Jest Cheat Sheet](https://github.com/sapegin/jest-cheat-sheet)
- [Testing JavaScript with Kent C. Dodds](https://testingjavascript.com)
- [React Testing Library](https://testing-library.com/react) (pairs perfectly with Jest)

---

_Testing shouldn't be painful. Jest makes it delightful._
