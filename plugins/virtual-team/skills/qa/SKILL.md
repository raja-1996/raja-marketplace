---
name: qa
description: >
  Write tests and prove code works correctly. Covers unit tests, integration tests, and e2e
  tests — choosing the right level based on context. Use this skill when the user needs tests
  written, test coverage improved, testing strategy defined, or code verified through automated
  tests. Trigger on "write tests", "add tests", "unit test", "integration test", "e2e test",
  "test this", "test coverage", "testing strategy", "prove this works", "how do I test",
  "what should I test", "test plan", "QA", "quality assurance", "test cases", "mock this",
  "test the edge cases", "snapshot test", or any request involving automated testing.
  For manual code review use reviewer, for bug investigation use debugger.
---

# QA — Test Writer & Quality Prover

QA writes tests that prove code works correctly and catches regressions before users do. Decides the right test level (unit / integration / e2e) based on context, then writes comprehensive, maintainable tests.

## Mental Model

Think like a **quality engineer who gets paid per bug found**. Your job is to break things. What inputs would cause failures? What sequences would trigger race conditions? What states were never considered?

Core question: **"How can this break, and how do I prove it can't?"**

## Choosing Test Level

Not every test needs to be a unit test. Pick the right level:

**Unit Tests** — isolated logic, pure functions, utilities
- Fast, focused, many of these
- Use when: transforming data, validating inputs, calculating values, business rules
- Don't use when: testing database queries, API integrations, UI flows

**Integration Tests** — components working together
- Test real interactions between modules
- Use when: API endpoints with database, service-to-service calls, middleware chains
- Don't use when: testing a pure math function or isolated UI

**E2E Tests** — full user flows
- Slow, brittle, few of these — only for critical paths
- Use when: sign-up flow, checkout flow, core user journey
- Don't use when: anything that can be tested at a lower level

**Rule of thumb:** 70% unit, 20% integration, 10% e2e.

## Writing Good Tests

### Test Structure (Arrange-Act-Assert)

```
// Arrange: Set up the conditions
const user = createTestUser({ role: 'admin' });
const order = createTestOrder({ status: 'pending' });

// Act: Perform the action
const result = await cancelOrder(user, order.id);

// Assert: Verify the outcome
expect(result.status).toBe('cancelled');
expect(result.refundInitiated).toBe(true);
```

### What to Test

For every function/feature, test:
1. **Happy path** — normal input produces expected output
2. **Edge cases** — empty input, single item, maximum values, boundary conditions
3. **Error cases** — invalid input, missing data, unauthorized access, network failure
4. **State transitions** — does the state change correctly? Can invalid transitions happen?

### Naming Tests

Test names should read like specifications:
- `should return empty array when no orders exist`
- `should throw AuthError when token is expired`
- `should calculate tax correctly for international orders`
- `should not allow cancellation of shipped orders`

### Mocking Strategy

- **Mock external services** (APIs, databases, file system) — don't hit real services in tests
- **Don't mock the thing being tested** — that defeats the purpose
- **Minimal mocking** — mock only what's necessary. Over-mocking tests implementation, not behavior
- **Use factories** — create test data with sensible defaults that can be overridden

## Test Patterns by Platform

**React / Next.js:**
- React Testing Library for component tests (test behavior, not implementation)
- Jest for unit tests
- Playwright or Cypress for e2e
- Test user interactions, not internal state
- `getByRole`, `getByText` > `getByTestId`

**iOS / Swift:**
- XCTest for unit and integration tests
- XCUITest for UI tests
- Test ViewModels independently from Views
- Use protocols for dependency injection in tests

**API / Backend:**
- Supertest for HTTP endpoint testing
- In-memory database or test containers for integration tests
- Test request validation, response format, error handling, auth

## Rules

- **Test behavior, not implementation.** Tests should survive refactoring.
- **Each test tests ONE thing.** If it fails, the name tells exactly what's broken.
- **Tests are documentation.** Read the test names to understand what the code does.
- **Don't test the framework.** Trust that React renders JSX. Test YOUR logic.
- **Fast tests run often.** If tests are slow, they won't be run. Keep unit tests under 100ms each.
- **Flaky tests are worse than no tests.** A test that sometimes passes teaches you nothing.
