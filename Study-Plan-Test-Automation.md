# Study Plan: Test Automation Fundamentals and Framework Design

This study plan is organized into three skill levels:
- Start: foundational concepts and first automation steps
- Intermediate: building practical Playwright tests and TypeScript skills
- Advanced: designing maintainable framework architecture and project hygiene

## Start: Foundation and First Automation Steps

### Focus areas
- JavaScript / TypeScript fundamentals
- Node.js runtime basics
- Test Automation Fundamentals
- Playwright runner and API
- UI testing concepts
- End-to-end test structure and lifecycle

### Goal
Build core language skills and basic automation knowledge so Playwright tests are stable and understandable.

### Study activities
- Learn JavaScript ES6 basics: arrow functions, destructuring, template literals.
- Study async/await, Promises, and module syntax (`import` / `export`).
- Learn Node.js runtime concepts and `package.json` scripts.
- Learn test types: unit, integration, end-to-end.
- Study the E2E lifecycle: setup, act, assert, teardown.
- Install Playwright and run your first test.
- Learn basic Playwright concepts: `test`, `expect`, `page`, `locator`.
- Practice navigation and page actions:
  - `page.goto()`
  - `page.click()`
  - `page.fill()`
- Try basic assertions: `toBeVisible()`, `toHaveText()`, `toHaveValue()`.
- Use selector strategies:
  - semantic: `getByRole()`
  - ARIA labels: `getByLabel()`
  - explicit contracts: `data-testid`
- Write one full user journey test with Arrange/Act/Assert.

### Recommended exercises
- Day 1: Learn JavaScript fundamentals and module syntax.
- Day 2: Run Node.js test scripts and understand `package.json`.
- Day 3: Write a simple Playwright test for a form.
- Day 4: Practice selectors and assertions with a demo page.
- Day 5: Build a complete E2E scenario with setup and teardown.

---

## Intermediate: Practical Playwright and POM Design

### Focus areas
- Strong TypeScript knowledge for typing, interfaces, and generics
- POM pattern for maintainable page abstractions
- Separation of test logic from page interactions
- Writing reusable utilities and locators
- Advanced Playwright runner and API usage

### Goal
Use TypeScript in automation, create reusable page objects, and structure tests cleanly.

### Study activities
- Study TypeScript basics in depth:
  - interfaces
  - type aliases
  - generics
  - union types
- Build typed fixtures and helper functions.
- Create Page Object Model classes:
  - `BasePage`
  - `MilkCollectionPage`
  - locator classes
- Move selectors and page actions into page objects.
- Make tests call high-level page actions instead of raw locators.
- Build reusable utilities for waits, logging, and validation.
- Learn advanced Playwright test runner features: lifecycle hooks, retries, reporters.

### Recommended exercises
- Day 1: Convert a JavaScript test to TypeScript with typed page objects.
- Day 2: Build a `BasePage` with common navigation and wait helpers.
- Day 3: Create locator classes that prefer semantic selectors.
- Day 4: Write reusable fixture generators like `generateFarmer()`.
- Day 5: Refactor tests to use POM methods and simplify assertions.

---

## Advanced: Framework Architecture and Project Maintenance

### Focus areas
- Directory structure planning
- Naming conventions for tests and locators
- Clean test isolation and cleanup strategies
- Framework-level design patterns

### Goal
Design a maintainable automation framework with strong organization, isolation, and long-term stability.

### Study activities
- Plan folder structure for tests, pages, mocks, fixtures, utils.
- Choose naming conventions:
  - feature.success.spec.ts
  - feature.error.spec.ts
  - locators with clear responsibilities
- Learn clean test isolation:
  - use `beforeEach` and `afterEach`
  - reset mocks between tests
  - avoid shared global state
- Practice cleanup strategies:
  - reset browser context
  - remove/restore test data
  - clear MSW handlers
- Study architecture patterns:
  - Page Object Pattern
  - Strategy Pattern for selectors and environment config
  - Template Method for lifecycle hooks
  - Facade Pattern for page API design
- Add reporting and debugging support:
  - screenshots on failure
  - Playwright HTML report
  - structured log output

### Recommended exercises
- Day 1: Map a complete feature to a directory and file plan.
- Day 2: Define naming rules for spec and locator files.
- Day 3: Write tests that fully isolate their state and clean up afterward.
- Day 4: Review and refactor the repo structure for maintainability.
- Day 5: Add a framework-level example using a `BasePage`, POM, and clean hooks.

---

## Implementation Plan: Day by Day

### Week 1: Start Level
- Day 1: Learn JavaScript fundamentals, modules, and Node.js basics. Install Playwright and verify `npx playwright test` works.
- Day 2: Study E2E test types and lifecycle. Write a basic Playwright test that navigates to a page and checks an element.
- Day 3: Practice Playwright selectors and actions. Build a form interaction using `page.goto()`, `page.fill()`, and `page.click()`.
- Day 4: Add assertions and state checks. Use `expect()` with `toBeVisible()`, `toHaveText()`, and `toHaveValue()`.
- Day 5: Build a complete user journey test with Arrange/Act/Assert and tag it with `@smoke`.

### Week 2: Intermediate Level
- Day 6: Learn TypeScript interfaces and type aliases. Convert a small test helper into TypeScript.
- Day 7: Study async/await, Promise handling, and import/export syntax. Use these in a Playwright helper module.
- Day 8: Build a `BasePage` class with navigation and common wait methods.
- Day 9: Create a feature page object (e.g. `MilkCollectionPage`) and locator class using semantic selectors.
- Day 10: Refactor an existing test to use the new page object and typed fixtures.

### Week 3: Intermediate to Advanced
- Day 11: Learn directory structure best practices. Organize tests, pages, mocks, fixtures, and utils.
- Day 12: Add reusable utilities for wait helpers, logging, and validation.
- Day 13: Build a data fixture generator like `generateFarmer()` and use it inside tests.
- Day 14: Add Playwright lifecycle hooks: `beforeEach`, `afterEach`, and reset mock state between tests.
- Day 15: Write a second feature test using POM, fixtures, and page interaction methods.

### Week 4: Advanced Level
- Day 16: Define naming conventions for spec files and locators. Rename sample files to align with the framework.
- Day 17: Implement clean test isolation and cleanup: browser context reset, fixture cleanup, MSW reset.
- Day 18: Add reporting and debugging support: HTML report, screenshots on failure, structured logs.
- Day 19: Review and refactor the repo structure and test code for maintainability.
- Day 20: Run the full suite, inspect results, fix flaky behavior, and document the final implementation.

## Suggested resources
- Playwright docs: https://playwright.dev/docs/intro
- TypeScript docs: https://www.typescriptlang.org/docs/
- MSW docs for mocking: https://mswjs.io/docs/
- Playwright test examples and repo patterns

## Tips for success
- Start with simple tests before moving to architecture.
- Practice by converting manual test flows into automation.
- Use POM to keep tests readable and reduce duplication.
- Keep cleanup and isolation as a first-class concern.
- Review real repo files for applied best practices.
