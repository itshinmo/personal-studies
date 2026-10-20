**Table of Contents**

- [Vitest](#vitest)
- [Writing Tests](#writing-tests)
- [Grouping Tests with `describe`](#grouping-tests-with-describe)
- [Test Files](#test-files)
- [Parameterized Tests](#parameterized-tests)

[<sub>Source</sub>](https://vitest.dev/guide/)

---

<a name="vitest" id="vitest"></a>

# Vitest

Vitest is a testing framework built to work closely with Vite, sharing its configuration and fast startup time. It is designed as a drop-in alternative to Jest, with a similar API but faster execution, especially in projects already using Vite for their build setup.

Add `vitest` to `package.json`.
Add this script to `package.json`:
```json
{
  // ...
  "scripts": {
    "test": "vitest"
  }
  // ...
}
```

<a name="vitest--writing" id="vitest--writing"></a>

# Writing Tests

A test verifies that a piece of code produces the expected result. In Vitest, you use the `test` function to define a test, and `expect` to make assertions. Each test has a name (a string describing what it checks) and a function that contains one or more assertions. If any assertion fails, the test fails.

```tsx
import { expect, test } from 'vitest'

test('Math.sqrt works for perfect squares', () => {
  expect(Math.sqrt(4)).toBe(2)
  expect(Math.sqrt(144)).toBe(12)
  expect(Math.sqrt(0)).toBe(0)
})
```

You might also see tests written with `it` instead of `test`. They behave identically. `it` is just an alias that some people prefer because it reads more naturally with a descriptive name:

```tsx
import { expect, it } from 'vitest'

it('should compute square roots', () => {
  expect(Math.sqrt(4)).toBe(2)
})
```

<a name="vitest--grouping" id="vitest--grouping"></a>

# Grouping Tests with `describe`

As your test files grow, you'll want to organize related tests together. `describe` creates a test suite, which is a named group of tests:

```tsx
import { describe, expect, test } from 'vitest'

describe('Math.sqrt', () => {
  test('returns the square root of perfect squares', () => {
    expect(Math.sqrt(4)).toBe(2)
    expect(Math.sqrt(9)).toBe(3)
  })

  test('returns NaN for negative numbers', () => {
    expect(Math.sqrt(-1)).toBeNaN()
  })

  test('returns 0 for 0', () => {
    expect(Math.sqrt(0)).toBe(0)
  })
})
```

<a name="vitest--files" id="vitest--files"></a>

# Test Files

By default, Vitest looks for any file that contains `.test.` or `.spec.` in its name, such as `utils.test.js`, `app.spec.js`, or `math.test.jsx`. It searches in all subdirectories, so it doesn't matter where you place them.

<a name="vitest--parameterized" id="vitest--parameterized"></a>

# Parameterized Tests 

When you have several test cases that only differ in their inputs and expected outputs, writing a separate `test` for each one gets repetitive. `test.for` lets you define the cases as data and run the same test logic for all of them:

```tsx
import { expect, test } from 'vitest'

test.for([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('add(%i, %i) -> %i', ([a, b, expected]) => {
  expect(a + b).toBe(expected)
})
```

In the example above, the %i placeholders are replaced with the integer values from each data row. Vitest also supports other placeholder types, such as %s for strings and %f for floating-point numbers. As a result, the test runner generates test names such as add(1, 1) -> 2, add(1, 2) -> 3, and add(2, 1) -> 3.

If your cases have more than two or three values, passing objects is more readable. Use `$property` in the name to interpolate fields:

```tsx
test.for([
  { a: 1, b: 1, expected: 2 },
  { a: 1, b: 2, expected: 3 },
  { a: 2, b: 1, expected: 3 },
])('add($a, $b) -> $expected', ({ a, b, expected }) => {
  expect(a + b).toBe(expected)
})
```

