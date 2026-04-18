# Framework Development Knowledge Requirements

This document describes the complete set of knowledge areas required to design, build, and maintain the test automation framework in this repository. It is written as a reference for engineers who will own the automation architecture, implementation, and long-term stability.

## 1. Test Automation Fundamentals

### What to know
- Differences between unit, integration, and end-to-end (E2E) tests.
- Test lifecycle phases: discovery, setup, arrange, act, assert, teardown.
- Why deterministic behavior is critical for CI stability.
- How to isolate tests so they do not depend on ordering or shared state.
- How to design tests around user journeys and business outcomes.

### Why it matters
- The framework is not just a set of scripts; it is an architecture for reliable validation.
- Poorly designed test flows create flakiness and make results hard to trust.
- Correct lifecycle handling ensures cleanup, reporting, and artifact generation happen consistently.

### Key concepts and patterns
- `test.describe`, `test.beforeEach`, `test.afterEach`, `test.afterAll`: these define structure and isolation.
- Assertions: `expect(locator).toBeVisible()`, `toContainText`, `toHaveValue`.
- Retry strategy, timeout strategy, and failure artifact capture.
- State-based waiting vs hard waits.
- Test tagging and metadata for selective execution (`@smoke`, `@regression`, `@accessibility`).

### Recommended design patterns
- Template Method: use a test hook flow that always runs setup and teardown in the same shape.
- State Machine Thinking: model each UI flow as states such as loading, success, error, empty.
- Arrange/Act/Assert: keep tests readable and focused.

### Example
```typescript
import { test, expect } from '@playwright/test';

test.describe('Milk Collection flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/milk-collection');
  });

  test('should submit form successfully', async ({ page }) => {
    // Arrange
    await page.fill('input[name="liters"]', '20');
    // Act
    await page.click('button:has-text("Submit")');
    // Assert
    await expect(page.getByText('Submission successful')).toBeVisible();
  });
});
```

---

## 2. JavaScript / TypeScript

### Core skills
- Modern JavaScript syntax: arrow functions, destructuring, template literals, async/await.
- TypeScript fundamentals: interfaces, type aliases, generics, enums, union types.
- Module system: `import` / `export`, default vs named exports.
- Node.js runtime and package management with `npm` / `package.json`.
- Working with third-party libraries and typings.

### Why it matters
- The framework is implemented in TypeScript, so strong TS competency prevents runtime bugs and improves code quality.
- Node.js skills are necessary to install dependencies, run scripts, and understand the test execution environment.

### Practical examples
- Typed page objects, e.g. `class MilkCollectionPage { readonly locators: MilkCollectionLocators; ... }`.
- Data fixture types, e.g. `interface Farmer { id: string; name: string; region: string; }`.
- `async`/`await` for browser actions and network simulation.
- Using `npm run test` and package scripts.

### Example
```typescript
export interface Farmer {
  id: string;
  name: string;
  region: string;
}

export function generateFarmer(overrides: Partial<Farmer> = {}): Farmer {
  return {
    id: `farmer-${nanoid(8)}`,
    name: 'Test Farmer',
    region: 'North',
    ...overrides,
  };
}
```

```json
{
  "scripts": {
    "test": "npx playwright test",
    "test:report": "npx playwright show-report"
  }
}
```

### Patterns and rationale
- Factory Pattern: build deterministic test data generators such as `generateFarmer()`.
- Module Pattern: keep utility functions isolated in `tests/utils/`.
- Type safety as contract enforcement between tests and page objects.

---

## 3. Page Object Model (POM) & Test Design

### What to know
- The Page Object Model separates page structure from test flow.
- Use page classes for actions and locator classes for element resolution.
- Keep tests high-level and page objects low-level.
- Prefer a feature-centric page object over sprawling utility functions.

### Why it matters
- POM improves maintainability: UI selector changes only affect page objects, not every test.
- It makes tests easier to read and reduces duplication.
- It enables reuse across multiple scenarios and flows.

### Common POM structure in this repo
- `tests/pages/BasePage.ts`: common helpers for navigation, waiting, and logging.
- `tests/pages/MilkCollectionPage.ts`: feature-specific form interactions.
- `tests/pages/locators/MilkCollection.locators.ts`: selector strategy and fallback locators.

### Design patterns used
- Page Object Pattern: encapsulates UI operations into classes.
- Strategy Pattern for selector choice: semantic roles first, aria labels next, then `data-testid`.
- Template Method in base page: common navigation and wait behavior is reused.
- Facade Pattern: page objects provide a simplified interface over complex browser interactions.

### Example rationale
- Use `getByRole('button', { name: /submit/i })` because it is stable and accessible.
- Keep `fillMilkCollection()` in the page object so tests can call `page.submitMilkCollection(data)` rather than repeating form steps.
- Put log and screenshot helpers in `BasePage` so all pages share diagnostics.

### Example
```typescript
class MilkCollectionPage extends BasePage {
  readonly locators = new MilkCollectionLocators(this.page);

  async submitMilkCollection(data: MilkCollectionInput) {
    await this.locators.getLitersInput().fill(data.liters.toString());
    await this.locators.getFarmerSelect().selectOption(data.farmerId);
    await this.locators.getSubmitButton().click();
  }
}
```

```typescript
class MilkCollectionLocators {
  constructor(private page: Page) {}

  getSubmitButton() {
    return this.page.getByRole('button', { name: /submit/i });
  }

  getLitersInput() {
    return this.page.getByLabel(/liters|volume/i);
  }
}
```

---

## 4. Mocking and API Simulation

### What to know
- MSW (Mock Service Worker) is used to intercept network calls during tests.
- Test scenarios should cover success, error, empty, and loading responses.
- Handlers must be reset between tests to avoid cross-test pollution.
- Use request matching by URL and HTTP method.

### Why it matters
- Mocking removes reliance on external services and makes tests deterministic.
- It enables negative path coverage that is hard to reproduce against a real API.
- It speeds execution and makes offline testing possible.

### Implementation details
- `tests/mocks/server.ts`: single MSW server instance with default handlers.
- `tests/mocks/handlers.ts`: scenario-based handler sets for success, error, empty, loading.
- `server.use(...)`: override handlers in a test to simulate a specific scenario.
- `server.resetHandlers()`: restore default behavior before each test.

### Design patterns and rationale
- Test Double Pattern: MSW acts as a virtual backend for tests.
- Strategy Pattern: pick a handler set based on the scenario being tested.
- Composition: build common response shapes and override only the differences.

### Example use case
- For a loading-state test, override `/api/farmers` with `ctx.delay(3000)`.
- For an error-state test, use a `500` response handler for `/api/milk-collection`.
- For an empty-state test, return `{ data: [], total: 0 }`.

### Example
```typescript
import { rest } from 'msw';
import { server } from '../../mocks/server';

server.use(
  rest.get('http://localhost:5000/api/farmers', (req, res, ctx) => {
    return res(ctx.delay(3000), ctx.json({ data: [], total: 0 }));
  })
);
```

---

## 5. Web Application Testing Best Practices

### What to know
- Use semantic selectors first: `page.getByRole()`, `getByLabel()`.
- Use `data-testid` only as a fallback or contract.
- Avoid brittle selectors such as class names and DOM indexes.
- Wait for DOM state changes instead of fixed waits.
- Validate accessibility and keyboard behavior.

### Why it matters
- Semantic locators are more stable and usually tied to user-visible intent.
- Accessibility-first tests catch usability regressions.
- Hard-coded waits cause flakiness and slow tests.
- Deterministic test data avoids intermittent failures.

### Practical guidelines
- `await page.waitForLoadState('networkidle')` rather than sleeping.
- `await locator.waitFor({ state: 'visible' })` before interacting.
- `expect(locator).toHaveText(/success/i)` for stable assertions.
- No `Math.random()` or `Date.now()` inside tests that affect assertions.

### Example
```typescript
await page.goto('/milk-collection');
await page.getByRole('button', { name: /submit/i }).waitFor({ state: 'visible' });
await page.getByLabel(/liters/i).fill('20');
await page.getByRole('button', { name: /submit/i }).click();
await expect(page.getByText('Submission successful')).toBeVisible();
```

### Design patterns and quality rules
- Guard Clause Pattern: verify page readiness before actions.
- Contract-first Pattern: use `data-testid` as an explicit test contract.
- Accessibility as a first-class requirement, not an afterthought.

---

## 6. Configuration & Environment Management

### What to know
- Manage environment-specific config for dev, qa, uat, staging.
- Use `.env` variables for secrets and environment parameters.
- Configure Playwright with base URL, retries, workers, reporter settings.
- Apply toggles such as `USE_MSW` or `HEADLESS` from environment.

### Why it matters
- The same code can run in multiple environments with consistent behavior.
- Environment config helps avoid hard-coded values and makes CI reproducible.
- Central configuration prevents duplicated logic across tests.

### Implementation details
- `tests/core/config/environments.ts`: typed environment config and manager.
- `playwright.config.ts`: sets global setup/teardown, reporters, testDir, and use settings.
- `.env.example`: documents required variables for developers.

### Design patterns used
- Singleton Pattern: environment manager provides a single source of truth.
- Strategy Pattern: choose environment-specific behavior based on `process.env.ENV`.
- Configuration as Code: keep environment mappings and flags in typed files.

### Example rationale
- Use `config.useMSW()` to decide whether tests should mock API traffic.
- Use `baseURL` from config so tests can run against different deployments without rewriting.
- Use `globalSetup` / `globalTeardown` to start and stop shared infrastructure once.

### Example
```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    headless: process.env.HEADLESS !== 'false',
  },
  globalSetup: require.resolve('./tests/core/setup/global.setup.ts'),
});
```

```typescript
export function getEnvironmentConfig() {
  const env = (process.env.ENV || 'dev') as Environment;
  return envConfig[env];
}
```

---

## 7. Reporting & Debugging

### What to know
- Use Playwright reporters such as HTML and JSON.
- Capture screenshots, videos, and traces on failure.
- Log structured test metadata: status, duration, retry count.
- Persist logs and results to files or databases for later analysis.

### Why it matters
- Automated test failures must be easy to investigate.
- Failure artifacts are essential for debugging flaky or intermittent issues.
- Structured logs support dashboards and CI result summaries.

### Implementation details
- `playwright.config.ts` reporter settings.
- `tests/utils/logger.ts` for JSON logs.
- `tests/utils/hooks.ts` for after-test result persistence.
- Failure artifact folders: `reports/logs/`, `reports/screenshots/`, `reports/videos/`.

### Design patterns
- Adapter Pattern: structured logger adapts generic events into JSON output.
- Observer Pattern: test hooks observe status changes and record artifacts.
- Single Responsibility: logger only writes logs, hooks only decide when to capture.

### Example rationale
- Record retry count and test duration to identify flaky tests.
- Use Playwright trace on first retry for deeper diagnostics.
- Keep report generation separate from test execution logic.

### Example
```typescript
import { test } from '@playwright/test';

test.afterEach(async ({ page }, testInfo) => {
  if (testInfo.status !== 'passed') {
    await page.screenshot({ path: `reports/screenshots/${testInfo.title}.png` });
  }
  await page.context().tracing.stop({
    path: `reports/traces/${testInfo.title}.zip`,
  });
});
```

---

## 8. Optional but Valuable Skills

### Useful extensions to the core skill set
- Relational database fundamentals and PostgreSQL.
- SQL for analytics and test result persistence.
- CI/CD pipeline design with GitHub Actions or similar.
- Report frameworks such as Allure.
- Advanced MSW capabilities: conditional responses, response templating.

### Why it helps
- Database persistence enables metric-driven quality improvements.
- CI/CD knowledge turns a local framework into a production pipeline.
- Advanced reporting makes automation visible to stakeholders.

### Example value
- Store `test_results` and `flaky_tests` for trend analysis.
- Publish HTML reports as CI artifacts.
- Use GitHub Actions to run smoke and regression suites automatically.

---

## 9. Project Maintenance Skills

### What to know
- Organize directory structure for clear separation of concerns.
- Use consistent naming conventions for file names and test tags.
- Keep fixture data deterministic and reusable.
- Clean up test data after execution.
- Use version control judiciously and review automation changes.

### Why it matters
- Well-organized test code is easier to extend and onboard teammates.
- Predictable structure reduces search time and maintenance overhead.
- Clean data handling prevents cross-test contamination.

### Recommended structure
- `tests/specs/` for executable tests.
- `tests/pages/` for page objects.
- `tests/mocks/` for MSW handlers.
- `tests/fixtures/` for data factories.
- `tests/utils/` for shared helpers, loggers, config, and hooks.

### Example
```text
v2/
  tests/
    specs/
      ui/
        milkCollection.success.spec.ts
        milkCollection.error.spec.ts
      api/
        payment.spec.ts
    pages/
      BasePage.ts
      MilkCollectionPage.ts
      locators/
        MilkCollection.locators.ts
    mocks/
      handlers.ts
      server.ts
    fixtures/
      farmers.fixture.ts
    utils/
      logger.ts
      hooks.ts
      config.ts
```

### Design patterns and maintenance practices
- Layered Architecture: keep UI, mocks, fixtures, and utilities separate.
- Convention over configuration: use folder-based structure and naming rules.
- Clean Code principles: single responsibility, DRY, explicit intent.

---

## 10. Summary Checklist

When building this framework, you should be comfortable with:

- Playwright automation and page interactions.
- TypeScript and Node.js development.
- Page Object Model and locator strategy design.
- MSW-based mocking for success, error, empty, and loading scenarios.
- Semantic selectors, accessibility, and robust wait strategies.
- Environment configuration and CI-friendly execution.
- Reporting, artifact capture, and debugging workflows.
- Optional database persistence and CI pipeline integration.
- Building a maintainable folder structure and isolation strategy.

## Recommended learning path

1. Learn Playwright basics and write simple UI tests.
2. Add TypeScript typing and convert tests to `.ts`.
3. Implement POM for one page or form.
4. Add MSW mocking for API dependencies.
5. Build environment config and global test setup.
6. Add logging, screenshots, and trace reporting.
7. Expand the framework with fixtures, utilities, and hooks.
8. Integrate test execution into CI.

---

This document is intended to be used as the complete knowledge checklist for anyone who must develop and maintain the `00-AAF` automation framework.
