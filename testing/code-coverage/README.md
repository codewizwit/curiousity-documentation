# Code Coverage Best Practices

_Coverage tells you what's tested. Not if it's tested well._

## What Is Code Coverage?

Code coverage measures how much of your code is executed when tests run. It's a useful metric, but it's not a measure of test quality — just a map of what got touched.

Think of it like a fitness tracker: it tells you how much you walked, but not if you walked efficiently or reached your destination.

## The Four Types of Coverage

### 1. Line Coverage
What percentage of code lines were executed?
```javascript
function divide(a, b) {
  if (b === 0) {           // ✓ Covered
    throw new Error('!');  // ✗ Not covered
  }
  return a / b;            // ✓ Covered
}

test('divides numbers', () => {
  expect(divide(10, 2)).toBe(5);
});
// Line coverage: 66% (2 of 3 lines)
```

### 2. Branch Coverage
Did tests exercise all possible paths (if/else, switch cases)?
```javascript
function checkAge(age) {
  if (age >= 18) {         // Both branches?
    return 'adult';        // ✓ Covered
  } else {
    return 'minor';        // ✗ Not covered
  }
}

test('identifies adults', () => {
  expect(checkAge(25)).toBe('adult');
});
// Branch coverage: 50% (only one branch tested)
```

### 3. Function Coverage
What percentage of functions were called?
```javascript
function add(a, b) { return a + b; }      // ✓ Called
function subtract(a, b) { return a - b; } // ✗ Never called

test('adds numbers', () => {
  expect(add(2, 3)).toBe(5);
});
// Function coverage: 50%
```

### 4. Statement Coverage
Similar to line coverage, but counts logical statements.

## The Coverage Trap

### Don't Chase 100%

**Bad reasons for 100% coverage:**
- "It looks good on a badge"
- "The boss said so"
- "Other teams have it"

**Good reasons for high coverage:**
- Core business logic is thoroughly tested
- Edge cases are handled
- Critical paths have multiple test scenarios

### What 100% Coverage Doesn't Tell You

```javascript
// 100% coverage, terrible test
function transferMoney(from, to, amount) {
  if (amount <= 0) throw new Error('Invalid amount');
  if (from.balance < amount) throw new Error('Insufficient funds');
  from.balance -= amount;
  to.balance += amount;
  return { from, to };
}

test('transfers money', () => {
  const from = { balance: 100 };
  const to = { balance: 50 };
  transferMoney(from, to, 25);
  expect(true).toBe(true); // Passes but tests nothing!
});
```

This has 100% coverage but doesn't verify:
- The correct amounts were transferred
- Error handling works
- Edge cases (negative amounts, insufficient funds)

## Best Practices

### 1. Focus on Critical Code

**High coverage priority:**
- Business logic and calculations
- Data transformations
- Validation and error handling
- Security-related code
- Complex algorithms

**Lower coverage priority:**
- Simple getters/setters
- Configuration files
- Third-party library wrappers (test integration, not the library)
- Generated code

### 2. Use Coverage to Find Gaps, Not Prove Quality

Good workflow:
1. Write tests for intended behavior
2. Run coverage report
3. Look for untested edge cases or paths you forgot
4. Add targeted tests for those gaps

Bad workflow:
1. Check coverage percentage
2. Write tests just to make the number go up

### 3. Set Realistic Thresholds

```javascript
// jest.config.js
module.exports = {
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 75,
      lines: 80,
      statements: 80,
    },
    './src/core/': {
      branches: 90,
      functions: 95,
      lines: 95,
      statements: 95,
    },
  },
};
```

- **80-85%** is a reasonable target for most codebases
- **90%+** for critical business logic
- **Less than 70%** suggests gaps worth investigating

### 4. Review Coverage Trends, Not Just Numbers

```bash
# Good: Coverage increasing as new features are tested
Week 1: 65% → Week 4: 78%

# Red flag: Coverage dropping (new untested code?)
Week 1: 80% → Week 4: 72%
```

### 5. Don't Test for Coverage

```javascript
// Bad: Testing implementation details to hit 100%
test('internal helper was called', () => {
  const spy = jest.spyOn(component, '_privateMethod');
  component.doSomething();
  expect(spy).toHaveBeenCalled();
});

// Good: Testing behavior users care about
test('displays error message when form is invalid', () => {
  render(<Form />);
  fireEvent.click(screen.getByRole('button', { name: 'Submit' }));
  expect(screen.getByText('Email is required')).toBeInTheDocument();
});
```

## Reading Coverage Reports

### Istanbul/NYC Output
```
File           | % Stmts | % Branch | % Funcs | % Lines | Uncovered Lines
---------------|---------|----------|---------|---------|------------------
All files      |   82.14 |    75.50 |   88.23 |   82.14 |
 auth.js       |   95.00 |    90.00 |  100.00 |   95.00 | 45-47
 payment.js    |   60.00 |    50.00 |   75.00 |   60.00 | 12-15,22,30-35
```

**Red flags:**
- Low branch coverage (< 70%) — missing edge cases
- Specific files with poor coverage — risky areas
- Critical files in the "Uncovered Lines" column

### HTML Reports

Most coverage tools generate interactive HTML reports:
```bash
jest --coverage
open coverage/lcov-report/index.html
```

These highlight:
- Green: Covered lines
- Red: Uncovered lines
- Yellow: Partially covered branches

Use them to visually identify gaps.

## Coverage in CI/CD

### Fail Builds on Coverage Drops

```yaml
# GitHub Actions example
- name: Run tests with coverage
  run: npm test -- --coverage

- name: Check coverage threshold
  run: npm test -- --coverage --coverageThreshold='{"global":{"branches":80}}'
```

### Coverage Badges

Show coverage in your README:
```markdown
[![Coverage Status](https://coveralls.io/repos/github/user/repo/badge.svg)](https://coveralls.io/github/user/repo)
```

### Track Coverage Over Time

Tools like Codecov or Coveralls show coverage trends and PR diffs:
- "This PR increased coverage by +2.3%"
- "⚠️  12 new lines in payment.js are not covered"

## Anti-Patterns to Avoid

**❌ Testing Everything to Hit a Number**
Quality > quantity. One good test beats five shallow tests.

**❌ Ignoring Low Coverage Areas**
If critical code has 40% coverage, that's a risk, not a statistic.

**❌ Mocking Everything for Easy 100%**
If you mock all dependencies, you're testing your mocks, not your code.

**❌ Using Coverage as a Team KPI**
Coverage as a metric to judge developers leads to gaming the system.

**❌ No Coverage at All**
Don't let perfect be the enemy of good. Start measuring, start improving.

## The Pragmatic Approach

1. **Start where you are**: Measure current coverage without judgment
2. **Set incremental goals**: Increase by 5-10% per quarter
3. **Focus on risk**: Cover critical paths first
4. **Review in PRs**: Check if new code has reasonable coverage
5. **Use it as a guide**: Not a grade, just useful feedback

## Tools

**JavaScript/TypeScript:**
- [Istanbul](https://istanbul.js.org) - Most popular, used by Jest
- [NYC](https://github.com/istanbuljs/nyc) - Istanbul's CLI
- [Codecov](https://codecov.io) - Coverage tracking service
- [Coveralls](https://coveralls.io) - Alternative tracking service

**Cross-Language:**
- [SonarQube](https://www.sonarqube.org) - Code quality + coverage
- [Codacy](https://www.codacy.com) - Automated code reviews + coverage

## Resources

- [Martin Fowler on Test Coverage](https://martinfowler.com/bliki/TestCoverage.html)
- [Google Testing Blog](https://testing.googleblog.com)
- [Istanbul Documentation](https://istanbul.js.org)
- [The Way We Test (Spotify)](https://engineering.atspotify.com/2018/01/testing-of-microservices/)

---

_Coverage is a flashlight, not a scoreboard. Use it to find what's dark, not to brag about the brightness._ 🌭

← [Back to Home](../../README.md) | [Jest](../jest/README.md) | [Stryker (Mutation Testing)](../stryker/README.md)
