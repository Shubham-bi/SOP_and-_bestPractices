# Test Automation Framework Architecture

**SOP-Aligned | Production-Ready | Industry Best Practices**

---

## Table of Contents

1. High-Level Architecture
2. Technology Stack & Justification
3. Component Architecture
4. Data Flow & Test Execution
5. Frontend SOP Alignment
6. Directory Structure
7. Key Features & Implementation
8. Scalability & Performance
9. Integration Points
10. Security & Compliance

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     CI/CD Pipeline (GitHub Actions)             │
│  Triggered on: push, pull_request, schedule                     │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
        ┌─────────────────────────────────────────┐
        │   Test Execution Environment Setup      │
        │  • Node.js 18 runtime                   │
        │  • PostgreSQL service container         │
        │  • MSW (Mock Service Worker) server     │
        │  • Playwright browser instances         │
        └────────────┬────────────────────────────┘
                     │
        ┌────────────┴────────────┬────────────────┐
        ▼                         ▼                ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│   Unit Tests     │  │   API Tests      │  │   UI Tests       │
│ (Dev Owned)      │  │ (Playwright)     │  │ (Playwright)     │
│                  │  │                  │  │                  │
│ • Component      │  │ • Contract       │  │ • Page Object    │
│ • Hooks          │  │ • E2E Scenarios  │  │ • User Flows     │
│ • Services       │  │ • MSW Mocking    │  │ • Accessibility  │
└─────────┬────────┘  └────────┬─────────┘  └────────┬─────────┘
          │                    │                    │
          └────────────────────┼────────────────────┘
                               │
        ┌──────────────────────▼─────────────────────┐
        │  Test Results Collection & Categorization  │
        │                                            │
        │  ✅ Pass / ❌ Fail / ⚠️ Flaky / ⏱️ Timeout │
        └──────────────────────┬─────────────────────┘
                               │
        ┌──────────────────────┴─────────────────────┐
        │         Results Storage & Analytics        │
        │                                            │
        │  ┌─────────────────────────────────────┐  │
        │  │ PostgreSQL Database                 │  │
        │  │ • test_results table                │  │
        │  │ • flaky_tests tracking              │  │
        │  │ • environment metrics               │  │
        │  └─────────────────────────────────────┘  │
        │                                            │
        │  ┌─────────────────────────────────────┐  │
        │  │ JSON Logs (ELK-ready)               │  │
        │  │ • Structured logging                │  │
        │  │ • Failure diagnostics               │  │
        │  │ • Performance traces                │  │
        │  └─────────────────────────────────────┘  │
        │                                            │
        │  ┌─────────────────────────────────────┐  │
        │  │ Artifacts                           │  │
        │  │ • Screenshots (on-failure)          │  │
        │  │ • Video recordings                  │  │
        │  │ • Trace files                       │  │
        │  └─────────────────────────────────────┘  │
        └──────────────────────┬─────────────────────┘
                               │
        ┌──────────────────────▼─────────────────────┐
        │         Reporting & Visualization         │
        │                                            │
        │  ┌─────────────────────────────────────┐  │
        │  │ Playwright HTML Report              │  │
        │  │ • Test-level details                │  │
        │  │ • Failure screenshots               │  │
        │  │ • Video playback                    │  │
        │  └─────────────────────────────────────┘  │
        │                                            │
        │  ┌─────────────────────────────────────┐  │
        │  │ Grafana Dashboards                  │  │
        │  │ • Pass rate trends                  │  │
        │  │ • Execution time analysis           │  │
        │  │ • Flaky test tracking               │  │
        │  │ • Environment comparisons           │  │
        │  └─────────────────────────────────────┘  │
        │                                            │
        │  ┌─────────────────────────────────────┐  │
        │  │ Allure Reports (Optional)           │  │
        │  │ • Detailed test history             │  │
        │  │ • Test execution timeline           │  │
        │  │ • Issue tracking integration        │  │
        │  └─────────────────────────────────────┘  │
        └──────────────────────┬─────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Team Notifications  │
                    │                      │
                    │  • Slack alerts      │
                    │  • Email reports     │
                    │  • Dashboard links   │
                    └──────────────────────┘
```

---

## Technology Stack & Justification

### 1. **Playwright** (Test Framework)
**Why:**
- ✅ Supports both UI and API testing (single framework advantage)
- ✅ Built-in retry, tracing, video recording capabilities
- ✅ Better stability than Selenium/Cypress for SOP compliance
- ✅ Strong accessibility testing support
- ✅ Fast execution (headless by default)
- ✅ Excellent debugging tools

**What We Use:**
```
- @playwright/test: Test runner + assertions
- page.goto(), page.getByRole(): Semantic locators
- MSW integration: Network mocking
- Browser context isolation: Test data separation
- Trace & video artifacts: On-failure diagnostics
```

---

### 2. **MSW (Mock Service Worker)** (Network Mocking)
**Why:**
- ✅ **SOP Requirement**: "Use MSW to simulate error responses"
- ✅ Works at Service Worker level (more realistic than stubbing)
- ✅ Can mock success, error, empty, and loading scenarios independently
- ✅ No code changes needed (intercepts at network level)
- ✅ Enables parallel dev/testing (test before backend complete)

**What We Use:**
```
- rest.get/post/put/delete: HTTP handlers
- res(ctx.status(), ctx.json()): Response simulation
- server.use(): Handler override per test
- Scenario groups: success/error/empty/loading
```

---

### 3. **PostgreSQL** (Results Persistence)
**Why:**
- ✅ Structured test result storage
- ✅ SQL queries for analytics & trends
- ✅ Grafana native integration
- ✅ Supports flaky test tracking
- ✅ ACID compliance for data integrity
- ✅ Cost-effective & widely available

**What We Store:**
```sql
CREATE TABLE test_results (
  id, test_name, status, duration,
  retry_count, timestamp, environment, tags
);

CREATE TABLE flaky_tests (
  test_name, fail_count, pass_rate, last_seen
);
```

---

### 4. **Grafana** (Analytics & Dashboards)
**Why:**
- ✅ Real-time dashboard visualization
- ✅ Trends over time (pass rate, execution time)
- ✅ Alert thresholds (email/Slack on failures)
- ✅ PostgreSQL native source
- ✅ Open-source (cost-effective)
- ✅ Industry standard for observability

**What We Track:**
```
- Metrics: Pass %, Fail %, Average duration
- Trends: Daily pass rate, execution time over time
- Anomalies: Sudden spike in failures, timeouts
- Comparisons: Dev vs QA vs Staging environments
```

---

### 5. **GitHub Actions** (CI/CD)
**Why:**
- ✅ Native GitHub integration (no extra vendor)
- ✅ Supports parallel jobs & matrix strategies
- ✅ Free for public repos, reasonable pricing for private
- ✅ Excellent for branch-based testing workflows
- ✅ Built-in artifact storage (reports, logs, videos)

**What We Run:**
```yaml
- Trigger: push, pull_request, schedule
- Matrix: Multiple browsers (Chrome, Firefox)
- Services: PostgreSQL container + test DB
- Artifacts: Reports, logs, screenshots, videos
```

---

### 6. **TypeScript** (Language)
**Why:**
- ✅ **SOP Alignment**: Frontend repo is TypeScript
- ✅ Type safety prevents selector/locator bugs
- ✅ IDE autocomplete for Page Objects
- ✅ Interface definitions for test data
- ✅ Compilation-time error catching

**What We Use:**
```typescript
interface MilkCollectionInput { }
interface Farmer { }
interface EnvironmentConfig { }
```

---

### 7. **nanoid** (Deterministic IDs)
**Why:**
- ✅ **SOP Requirement**: No Math.random() or Date.now() in tests
- ✅ Generates stable, unique IDs
- ✅ URL-safe encoding
- ✅ Configurable prefix for traceability

**What We Use:**
```typescript
id: `farmer-${nanoid(8)}` // farmer-abc12345
```

---

## Component Architecture

### Layer 1: **Test Specification Layer**
```
tests/specs/
├── ui/                              # UI & integration tests
│   ├── milkCollection.success.spec.ts
│   ├── milkCollection.error.spec.ts
│   ├── milkCollection.empty.spec.ts
│   ├── milkCollection.loading.spec.ts
│   └── milkCollection.accessibility.spec.ts
└── api/                             # API contract tests
    ├── milk-collection.spec.ts
    └── payment.spec.ts
```

**Responsibility:**
- Define test scenarios (happy path, error, empty, loading)
- Use Page Objects for UI interaction
- Use API clients for backend validation
- Assert on final states (not intermediate)
- **SOP Compliance**: Every async operation has 4 state tests

**Example Test Structure:**
```typescript
test.describe('@smoke @ui Success Scenario', () => {
  test.beforeEach() // Setup: Create data
  test.afterEach()  // Teardown: Cleanup
  test()            // Assert: Verify state
});
```

---

### Layer 2: **Page Object Model (POM)**
```
tests/pages/
├── BasePage.ts                      # Base with common methods
├── MilkCollectionPage.ts            # Feature-specific POM
└── locators/
    ├── MilkCollection.locators.ts   # Selector strategy
    └── Login.locators.ts
```

**Responsibility:**
- Encapsulate UI interactions
- Manage selectors (semantic first, data-testid fallback)
- Provide high-level methods: `fillMilkCollection()`, `submitForm()`
- **SOP Compliance**: 
  - Priority 1: Semantic HTML roles
  - Priority 2: aria-label
  - Priority 3: data-testid (stable, prefixed)

**Example:**
```typescript
class MilkCollectionPage extends BasePage {
  // ✅ Semantic selector (priority 1)
  getSubmitButton() {
    return this.page.getByRole('button', { name: /submit/i });
  }

  // ✅ aria-label selector (priority 2)
  getLitersInput() {
    return this.page.getByLabel(/liters/i);
  }

  // ✅ data-testid fallback (priority 3)
  getErrorMessage() {
    return this.page.locator('[data-testid="milk-collection-error"]');
  }
}
```

---

### Layer 3: **Mocking & Data Layer**
```
tests/mocks/
├── handlers.ts                      # MSW request handlers
└── server.ts                        # MSW server setup

tests/fixtures/
├── farmers.fixture.ts               # Test data generators
├── payments.fixture.ts
└── users.fixture.ts

tests/utils/
├── dataFactory.ts                   # DB operations
├── db.ts                            # Connection pooling
└── config.ts                        # Environment config
```

**Responsibility:**
- Mock all HTTP requests (success, error, empty, loading)
- Generate deterministic test data
- Manage test database operations
- Handle environment-specific configurations

**Example - MSW Handlers:**
```typescript
// SUCCESS scenario
rest.get('/api/farmers', (req, res, ctx) => {
  return res(ctx.json({ data: [...], total: 2 }));
});

// ERROR scenario
rest.get('/api/farmers', (req, res, ctx) => {
  return res(ctx.status(500), ctx.json({ error: 'Server error' }));
});

// EMPTY scenario
rest.get('/api/farmers', (req, res, ctx) => {
  return res(ctx.json({ data: [], total: 0 }));
});

// LOADING scenario
rest.get('/api/farmers', (req, res, ctx) => {
  return res(ctx.delay(3000), ctx.json({ ... }));
});
```

**Example - Data Factories:**
```typescript
export function generateFarmer(overrides = {}) {
  return {
    id: `farmer-${nanoid(8)}`,        // Deterministic
    name: `TestFarmer-${Date.now()}`, // Time-scoped
    region: 'Region-A',
    animals: 5,
    ...overrides
  };
}
```

---

### Layer 4: **Configuration & Setup**
```
tests/core/
├── config/
│   ├── environments.ts              # Dev/QA/UAT/Staging config
│   ├── selectors.config.ts          # Selector standards
│   └── retry.config.ts              # Retry policies
└── setup/
    ├── global.setup.ts              # Pre-test initialization
    └── global.teardown.ts           # Post-test cleanup
```

**Responsibility:**
- Define environment-specific URLs, DB connections, credentials
- Set up MSW server, database, test data
- Clean up resources after all tests

**Example - Environment Config:**
```typescript
const envConfig = {
  dev: {
    baseURL: 'http://localhost:3000',
    apiURL: 'http://localhost:5000',
    dataIsolation: 'per-test',    // Clean after each test
    shouldSeed: true,
    shouldCleanup: true,
    useMSW: true                   // Mock all APIs
  },
  
  qa: {
    baseURL: 'https://qa-api.internal',
    dataIsolation: 'per-run',      // Clean after full run
    shouldSeed: true,
    useMSW: false                  // Use real APIs
  },
  
  uat: {
    dataIsolation: 'none',         // Never seed/cleanup UAT
    shouldSeed: false,
    shouldCleanup: false
  }
};
```

---

### Layer 5: **Utilities & Helpers**
```
tests/utils/
├── logger.ts                        # Structured JSON logging
├── wait.ts                          # Deterministic wait helpers
├── helpers.ts                       # Common functions
└── hooks.ts                         # Test lifecycle hooks
```

**Responsibility:**
- Provide reusable test utilities
- Structured logging (JSON for ELK integration)
- Custom wait conditions (not sleep)
- Lifecycle hooks for setup/teardown

**Example - Wait Utilities (SOP Compliant):**
```typescript
// ❌ NEVER: await new Promise(r => setTimeout(r, 2000))

// ✅ Always: Wait on state
export async function waitForElementState(locator, state, timeout) {
  switch (state) {
    case 'visible':
      await locator.waitFor({ state: 'visible', timeout });
    case 'hidden':
      await locator.waitFor({ state: 'hidden', timeout });
  }
}
```

---

### Layer 6: **Results & Reporting**
```
playwright-report/          # Playwright HTML report
test-results/
├── results.json            # JSON results
└── db_results.sql          # Database exports

reports/
├── logs/                    # Structured JSON logs
├── screenshots/            # On-failure screenshots
└── videos/                 # On-failure recordings
```

**Responsibility:**
- Capture test execution artifacts
- Store results in multiple formats (JSON, DB, logs)
- Generate HTML reports
- Enable integrations (Grafana, Allure, Slack)

---

## Data Flow & Test Execution

### Test Execution Lifecycle

```
┌─────────────────────────────────────────────────────────────┐
│ 1. TEST DISCOVERY                                           │
│ Playwright finds all *.spec.ts files in tests/specs/        │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 2. GLOBAL SETUP                                             │
│                                                               │
│ ✅ Start MSW server (network mocking)                       │
│ ✅ Connect to PostgreSQL database                           │
│ ✅ Load environment configuration                           │
│ ✅ Initialize browsers                                      │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 3. TEST GROUP (describe block)                              │
│ Group tests with shared setup/teardown                      │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 4. BEFORE EACH                                              │
│                                                               │
│ ✅ Reset MSW handlers to default state                      │
│ ✅ Override handlers for test scenario (error, empty, etc)  │
│ ✅ Create fresh browser context                             │
│ ✅ Generate test data (factories)                           │
│ ✅ Insert test data into database                           │
│ ✅ Log: [START] Test name                                   │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 5. TEST EXECUTION                                           │
│                                                               │
│ Arrange:  Setup test preconditions                          │
│   • Navigate to page: await page.goto('/milk-collection')   │
│   • Await page ready: await page.waitForLoadState()         │
│                                                               │
│ Act:      Perform user actions                              │
│   • Click: await submitButton.click()                       │
│   • Fill: await input.fill('value')                         │
│   • Type: await page.keyboard.type()                        │
│                                                               │
│ Assert:   Verify expected outcomes                          │
│   • State: expect(element).toBeVisible()                    │
│   • Text: expect(element).toContainText()                   │
│   • Value: expect(input).toHaveValue()                      │
│                                                               │
│ Wait For: Final stable state (SOP Compliant)                │
│   • ✅ await page.waitForLoadState()                        │
│   • ✅ await element.waitFor({ state: 'visible' })         │
│   • ❌ await new Promise(r => setTimeout(r, 2000))         │
└────────────────────┬────────────────────────────────────────┘
                     │
        ┌────────────▼────────────┐
        │ Test Pass / Fail / Error │
        └────────────┬────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 6. AFTER EACH                                               │
│                                                               │
│ ✅ Capture test metadata                                    │
│    • status: 'passed' | 'failed' | 'skipped'                │
│    • duration: test execution time (ms)                     │
│    • retry: attempt number                                  │
│                                                               │
│ ✅ Flaky test detection                                     │
│    • If retry > 0: Mark as FLAKY                            │
│    • Log warning: "FLAKY TEST: X failed on attempt Y"       │
│                                                               │
│ ✅ Test data cleanup                                        │
│    • DELETE FROM farmers WHERE name LIKE 'TestFarmer-%'     │
│    • DELETE FROM payments WHERE id LIKE 'payment-%'         │
│                                                               │
│ ✅ Result persistence                                       │
│    • INSERT INTO test_results (...)                         │
│    • Write JSON logs to file                                │
│                                                               │
│ ✅ Artifacts capture                                        │
│    • Screenshots: if status == 'failed'                     │
│    • Videos: if status == 'failed'                          │
│    • Traces: on first retry                                 │
│                                                               │
│ ✅ Log: [END] Test name (status, duration)                  │
│                                                               │
│ ✅ Browser cleanup                                          │
│    • Close page & context                                   │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 7. RETRY LOGIC (if enabled)                                 │
│                                                               │
│ If failed && retries remaining:                             │
│   Retry test (go back to "BEFORE EACH")                     │
│ Else:                                                        │
│   Mark as permanently failed                                │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│ 8. GLOBAL TEARDOWN                                          │
│                                                               │
│ ✅ Stop MSW server                                          │
│ ✅ Close database connections                               │
│ ✅ Generate test reports                                    │
│   • Playwright HTML report                                  │
│   • JSON results file                                       │
│   • Logs aggregation                                        │
│                                                               │
│ Final: All results in database + reports ready              │
└──────────────────────────────────────────────────────────────┘
```

---

### Data Flow Example: Milk Collection Test

```
┌────────────────────────────────────────────────────────────┐
│ TEST: should submit milk collection form successfully       │
└───────────────────────┬──────────────────────────────────────┘
                        │
        ┌───────────────▼────────────────┐
        │ BEFORE EACH Hook               │
        │                                │
        │ 1. Set MSW success handler:    │
        │    POST /api/milk-collection   │
        │    → 201 Created               │
        │                                │
        │ 2. Create test farmer:         │
        │    farmer = {                  │
        │      id: 'farmer-abc123',      │
        │      name: 'TestFarmer-2026'   │
        │    }                           │
        │                                │
        │ 3. Insert to DB:               │
        │    INSERT INTO farmers (...)   │
        │                                │
        │ 4. Create browser:             │
        │    await browser.newContext()  │
        └───────────────┬────────────────┘
                        │
        ┌───────────────▼────────────────┐
        │ TEST EXECUTION (AAA Pattern)   │
        │                                │
        │ A)RRANGEact:                   │
        │    await page.goto(             │
        │      '/milk-collection'        │
        │    )                           │
        │                                │
        │ A)CT:                          │
        │    await page                  │
        │      .getByLabel(/farmer/i)    │
        │      .fill('farmer-abc123')    │
        │                                │
        │    await page                  │
        │      .getByLabel(/liters/i)    │
        │      .fill('25')               │
        │                                │
        │    await page                  │
        │      .getByRole('button',      │
        │        { name: /submit/i }     │
        │      ).click()                 │
        │                                │
        │ → Playwright intercepts POST   │
        │ → MSW matches handler          │
        │ → Returns mocked 201 response  │
        │ → Page shows success message   │
        │                                │
        │ A)SSERT:                       │
        │    expect(                     │
        │      page.locator(             │
        │        '[data-testid=          │
        │          milk-collection-      │
        │          success]'             │
        │      )                         │
        │    ).toBeVisible()             │
        └───────────────┬────────────────┘
                        │
        ┌───────────────▼────────────────┐
        │ AFTER EACH Hook                │
        │                                │
        │ 1. STATUS = 'PASSED'           │
        │    DURATION = 2342ms           │
        │    RETRY = 0                   │
        │                                │
        │ 2. Save to DB:                 │
        │    INSERT INTO test_results(   │
        │      'should submit...',       │
        │      'PASSED',                 │
        │      2342, 0, now, 'dev'       │
        │    )                           │
        │                                │
        │ 3. Write JSON log:             │
        │    {                           │
        │      "timestamp": "2026-04-15" │
        │      "level": "INFO",          │
        │      "message": "[END] should" │
        │      "status": "PASSED",       │
        │      "duration": 2342          │
        │    }                           │
        │                                │
        │ 4. Cleanup:                    │
        │    DELETE FROM farmers         │
        │    WHERE id = 'farmer-...'     │
        │                                │
        │ 5. Close browser:              │
        │    await page.close()          │
        └────────────────────────────────┘
```

---

## Frontend SOP Alignment

### 1. **Selector Strategy Priority**

| Priority | Implementation | SOP Section |
|----------|---|---|
| **1. Semantic HTML** | `page.getByRole('button', { name: /submit/i })` | 3.1 |
| **2. ARIA Labels** | `page.getByLabel('Liters Input')` | 3.1 |
| **3. data-testid** | `page.locator('[data-testid="milk-collection-error"]')` | 3.2 |
| **4. CSS Classes** | ❌ NEVER USED | 3.1 |

**Implementation in** `tests/pages/locators/MilkCollection.locators.ts`:

```typescript
// Priority 1: Semantic - Always try first
getSubmitButton() {
  return this.page.getByRole('button', { name: /submit|save/i });
}

// Priority 2: ARIA - When semantic alone is ambiguous
getLitersInput() {
  return this.page.getByLabel(/liters|volume/i);
}

// Priority 3: data-testid - Only fallback
getErrorMessage() {
  return this.page.locator('[data-testid="milk-collection-error"]');
}

// ❌ NEVER
// document.querySelector('.btn-primary')
```

---

### 2. **Mandatory 4-State Testing** (SOP Section 6.3)

**Requirement:** 
> "No feature can be marked 'done' unless its loading, error, and empty states are both implemented and have test coverage."

**Implementation:**

```
tests/specs/ui/
├── milkCollection.success.spec.ts      ✅ Happy path + form validation
├── milkCollection.error.spec.ts        ✅ 500/400/401/404/503 errors
├── milkCollection.empty.spec.ts        ✅ No contractors, no data
├── milkCollection.loading.spec.ts      ✅ Spinner visibility + disable form
└── milkCollection.accessibility.spec.ts ✅ A11y compliance
```

**Each state is tested independently using MSW:**

```typescript
// SUCCESS scenario handler
rest.post('/api/milk-collection', (req, res, ctx) => {
  return res(ctx.status(201), ctx.json({ id: '123', status: 'recorded' }));
});

// ERROR scenario handler
rest.post('/api/milk-collection', (req, res, ctx) => {
  return res(ctx.status(500), ctx.json({ error: 'Server error' }));
});

// EMPTY scenario handler
rest.get('/api/farmers', (req, res, ctx) => {
  return res(ctx.json({ data: [], total: 0 }));
});

// LOADING scenario handler
rest.get('/api/farmers', (req, res, ctx) => {
  return res(ctx.delay(3000), ctx.json({ ... }));
});
```

---

### 3. **Deterministic UI Behavior** (SOP Section 7.1)

**Requirement:**
> "Never use Math.random(), Date.now(), or new Date() directly in rendered output"

**Implementation:**

```typescript
// ❌ NEVER in test data
export function generateFarmer() {
  return {
    id: Math.random().toString(),  // ❌ WRONG
    name: `Farmer-${Date.now()}`   // ❌ WRONG
  };
}

// ✅ ALWAYS use nanoid + deterministic prefix
export function generateFarmer() {
  return {
    id: `farmer-${nanoid(8)}`,     // ✅ Deterministic + unique
    name: `TestFarmer-${now.split('T')[0]}`  // ✅ Scoped by date
  };
}

// ✅ Disable animations in tests
await page.addInitScript(() => {
  document.documentElement.classList.add('no-animation');
});
```

---

### 4. **Async Handling** (SOP Section 7.3)

**Requirement:**
> "Never use arbitrary sleep() or setTimeout delays - always wait on DOM state"

**Implementation:**

```typescript
// ❌ NEVER
await new Promise(r => setTimeout(r, 2000));

// ✅ ALWAYS wait on state
await page.waitForLoadState('networkidle');
await element.waitFor({ state: 'visible' });
await page.waitForFunction(() => {
  return document.body.innerText.includes('Success');
});
```

---

### 5. **Accessibility Testing** (SOP Section 8.3)

**Requirement:**
> "Integrate jest-axe on every component render test"

**Implementation:**

```typescript
import { injectAxe, checkA11y } from 'axe-playwright';

test('should have no axe accessibility violations', async () => {
  await injectAxe(page);
  await checkA11y(page, null, { detailedReport: true });
});

test('should support keyboard-only navigation', async () => {
  // Tab through all interactive elements
  await page.keyboard.press('Tab');
  const focused = page.locator(':focus');
  expect(focused).toBeVisible();
});

test('should have visible focus indicators', async () => {
  const styles = await focused.evaluate(el => ({
    outline: getComputedStyle(el).outline,
    boxShadow: getComputedStyle(el).boxShadow
  }));
  expect(styles.outline || styles.boxShadow).toBeTruthy();
});
```

---

### 6. **Test Isolation** (SOP Section 7.4)

**Requirement:**
> "Each test must set up its own data - never rely on test execution order"

**Implementation:**

```typescript
test.beforeEach(async () => {
  // ✅ Create fresh data for THIS test only
  const farmer = await dataFactory.createFarmer();
  testContext.farmerId = farmer.id;
});

test.afterEach(async () => {
  // ✅ Clean up after THIS test
  await dataFactory.cleanupAll();
  
  // ✅ Reset MSW handlers
  server.resetHandlers();
  
  // ✅ Close browser
  await page.close();
});

test('test 1', async () => {
  // Uses farmerId from beforeEach
});

test('test 2', async () => {
  // Gets DIFFERENT farmerId (not test 1's data)
});
```

---

### 7. **Error/Empty/Loading States** (SOP Sections 6.1-6.3)

**Requirement:**
> "Every async operation must handle loading, success, error, and empty states"

**Implementation:**

```typescript
// Page Object provides methods for EACH state

async verifyLoadingState() {
  // ✅ Loading spinner visible
  const spinner = this.locators.getLoadingSpinner();
  await this.waitForElement(spinner);
}

async verifySuccessState() {
  // ✅ Success message visible
  const successMsg = this.locators.getSuccessMessage();
  await this.waitForElement(successMsg);
}

async verifyErrorState() {
  // ✅ Error message visible
  const errorMsg = this.locators.getErrorMessage();
  await this.waitForElement(errorMsg);
}

async verifyEmptyState() {
  // ✅ Empty state UI visible
  const emptyState = this.locators.getEmptyState();
  await this.waitForElement(emptyState);
}
```

---

## Directory Structure

### Complete Hierarchy

```
v2/
│
├── tests/
│   ├── specs/                           # Test files (executable)
│   │   ├── ui/
│   │   │   ├── milkCollection.success.spec.ts      (Happy path)
│   │   │   ├── milkCollection.error.spec.ts        (Error states)
│   │   │   ├── milkCollection.empty.spec.ts        (Empty states)
│   │   │   ├── milkCollection.loading.spec.ts      (Loading states)
│   │   │   └── milkCollection.accessibility.spec.ts (A11y)
│   │   └── api/
│   │       ├── milk-collection.spec.ts
│   │       └── payment.spec.ts
│   │
│   ├── pages/                           # Page Object Models
│   │   ├── BasePage.ts                  # Common methods
│   │   ├── MilkCollectionPage.ts        # Feature POM
│   │   ├── LoginPage.ts
│   │   └── locators/
│   │       ├── MilkCollection.locators.ts  (Selector strategy)
│   │       └── Login.locators.ts
│   │
│   ├── mocks/                           # Network mocking (MSW)
│   │   ├── handlers.ts                  # All HTTP handlers
│   │   │   ├── successHandlers         (201, 200 responses)
│   │   │   ├── errorHandlers           (500, 400, 401, 404, 503)
│   │   │   ├── emptyHandlers           (Empty data)
│   │   │   └── loadingHandlers         (Delayed responses)
│   │   └── server.ts                    # MSW server setup
│   │
│   ├── fixtures/                        # Test data generators
│   │   ├── farmers.fixture.ts           # Farmer factory
│   │   ├── payments.fixture.ts          # Payment factory
│   │   └── users.fixture.ts             # User factory
│   │
│   ├── core/                            # Core infrastructure
│   │   ├── config/
│   │   │   ├── environments.ts          # Dev/QA/UAT/Prod config
│   │   │   ├── selectors.config.ts      # Selector standards
│   │   │   └── retry.config.ts          # Retry policies
│   │   ├── setup/
│   │   │   ├── global.setup.ts          # Before all tests
│   │   │   ├── global.teardown.ts       # After all tests
│   │   │   └── schema.sql               # DB schema
│   │   └── types/
│   │       ├── test-data.types.ts
│   │       └── api.types.ts
│   │
│   ├── utils/                           # Helper functions
│   │   ├── logger.ts                    # JSON structured logging
│   │   ├── db.ts                        # DB connection pool
│   │   ├── dataFactory.ts               # DB operations (CRUD)
│   │   ├── config.ts                    # Config helpers
│   │   ├── wait.ts                      # Wait utilities (state-based)
│   │   ├── helpers.ts                   # Common utilities
│   │   └── hooks.ts                     # Lifecycle hooks
│   │
│   ├── api/                             # API client layer
│   │   ├── clients/
│   │   │   ├── farmerClient.ts
│   │   │   └── paymentClient.ts
│   │   └── schemas/
│   │       ├── farmer.schema.ts         # JSON schema validation
│   │       └── payment.schema.ts
│   │
│   └── reports/                         # Generated artifacts
│       ├── logs/                        # JSON logs (ELK-ready)
│       ├── screenshots/                 # Failure screenshots
│       ├── videos/                      # Failure videos
│       ├── traces/                      # Playwright traces
│       └── playwright-report/           # HTML report
│
├── playwright.config.ts                 # Playwright configuration
├── tsconfig.json                        # TypeScript config
├── package.json                         # Dependencies
├── .env.example                         # Example env vars
└── .github/
    └── workflows/
        └── test.yml                     # GitHub Actions
```

---

## Key Features & Implementation

### Feature 1: **Separation of Concerns**

```
Test Layer          │ Page Object          │ Locator Strategy
────────────────    │ ─────────────────    │ ─────────────────
✅ Knows WHAT       │ ↓ Knows HOW          │ ↓ Knows WHERE
  to test           │   to interact        │   to find

test('submit        │ class                │ getByRole()
  form', async      │ MilkCollectionPage   │ getByLabel()
  () => {           │ {                    │ data-testid
  const mp = new    │   fillForm() {}      │
  MilkCollection    │   submitForm() {}    │
  Page(page);       │   ...                │
  await mp.submit() │ }                    │
})                  │                      │
```

**Benefit:**
- Changes to selector don't affect test logic
- Multiple tests can use same POM
- Selector maintainability isolated to locators file

---

### Feature 2: **MSW-Based Reliability**

```
Traditional Approach         │  Our Approach (MSW)
──────────────────          │  ──────────────────
❌ Tests hit real API       │  ✅ Tests mock all APIs
❌ Depend on test env up    │  ✅ Works offline
❌ Data conflicts           │  ✅ Isolated scenarios
❌ Hard to test errors      │  ✅ Easy error injection
❌ Hard to simulate loading │  ✅ Easy delay injection
❌ Flaky due to timing      │  ✅ Deterministic
```

**Implementation:**
```typescript
// Override handler for THIS test
test('shows error', async () => {
  server.use(
    rest.post('/api/milk', (_, res, ctx) => {
      return res(ctx.status(500), ctx.json({ error: 'Failed' }));
    })
  );
  
  // Test now gets 500 error response
  await page.goto('/');
  // Assert error is displayed
});
```

---

### Feature 3: **Environment-Specific Configuration**

```
DEVELOPMENT                 │  QA                     │  UAT
───────────────             │  ──                     │  ───
baseURL: localhost:3000     │  https://qa-api         │  https://uat-api
useMSW: true                │  useMSW: false          │  useMSW: false
dataIsolation: per-test     │  dataIsolation: per-run │  dataIsolation: none
shouldSeed: true            │  shouldSeed: true       │  shouldSeed: false
shouldCleanup: true         │  shouldCleanup: true    │  shouldCleanup: false
credentials: dev-user       │  credentials: qa-user   │  credentials: uat-user

Usage:
ENV=dev npx playwright test
ENV=qa npx playwright test
ENV=uat npx playwright test
```

---

### Feature 4: **Structured Logging**

```json
{
  "timestamp": "2026-04-15T10:30:45.123Z",
  "level": "INFO",
  "message": "Submitting milk collection form",
  "data": {
    "farmerId": "farmer-abc123",
    "liters": 25,
    "environment": "dev"
  }
}

{
  "timestamp": "2026-04-15T10:30:46.456Z",
  "level": "ERROR",
  "message": "Form submission failed",
  "data": {
    "error": "Server error",
    "statusCode": 500
  }
}
```

**Integration:** JSON logs → ELK Stack (Elasticsearch, Logstash, Kibana)

---

### Feature 5: **Flaky Test Detection**

```typescript
// After each test, check retry count
test.afterEach(async ({}, testInfo) => {
  if (testInfo.retry > 0) {
    // Log as flaky
    logger.warn(`FLAKY TEST: ${testInfo.title}`);
    
    // Save to DB
    await pool.query(
      `INSERT INTO flaky_tests (test_name, fail_count) 
       VALUES ($1, 1)
       ON CONFLICT (test_name) DO UPDATE 
       SET fail_count = fail_count + 1`
    );
  }
});
```

**Benefit:** Track which tests are intermittently failing → root cause analysis

---

### Feature 6: **Results Persistence & Analytics**

```
Every test generates:
  ├─ Pass/Fail status
  ├─ Duration (ms)
  ├─ Retry count
  ├─ Environment name
  ├─ Tags (@smoke, @regression)
  └─ Timestamp

Stored in PostgreSQL → Queried by Grafana:
  ├─ Daily pass rate trend
  ├─ Average execution time
  ├─ Flaky test frequency
  ├─ Environment comparison
  └─ Performance degradation alerts
```

---

## Scalability & Performance

### Parallel Execution Strategy

```
Currently:
  fullyParallel: false
  workers: 1
  Reason: Prevent test interference until baseline is stable

Future (when tests are 100% stable):
  fullyParallel: true
  workers: 4  # Or number of CPU cores

Execution:
  ├─ Test 1, 2, 3, 4 run simultaneously
  ├─ Each gets isolated browser context
  ├─ Separate database connections
  ├─ No state bleed (beforeEach creates fresh data)
  └─ Total time: ~25% of serial (4x speedup)
```

---

### Performance Optimization

| Optimization | Implementation | Benefit |
|---|---|---|
| **Headless Mode** | `launchOptions: { headless: true }` | 10x faster |
| **No Screenshots** | `screenshot: 'only-on-failure'` | Reduces storage |
| **Video on Failure** | `video: 'retain-on-failure'` | Saves storage |
| **Trace on Retry** | `trace: 'on-first-retry'` | Debuggable flaky tests |
| **Connection Pooling** | `Pool({ max: 10 })` | Database efficiency |
| **MSW Over Real API** | Network mocking | 50% faster tests |
| **Parallel Browsers** | Multiple `workers` | N×faster (4 workers = 4x) |

---

### Scalability Path

```
Phase 1 (Now): Single environment, serial execution
├─ 5-10 tests per feature
├─ ~5 minutes total runtime
├─ Low infrastructure cost
└─ Goal: Establish baseline stability

Phase 2 (Month 1): Multi-environment, parallel execution
├─ Add dev/qa/uat/staging
├─ Enable 4 parallel workers
├─ ~1 minute total runtime
├─ Run on every commit + schedule

Phase 3 (Month 2): Add performance & load testing
├─ k6 for load testing
├─ Edge case coverage expansion
├─ Chaos testing (API failures)

Phase 4 (Month 3+): AI-driven optimization
├─ Test selection (run only impacted)
├─ Self-healing locators (if needed)
├─ Predictive flaky detection
```

---

## Integration Points

### 1. **CI/CD Integration (GitHub Actions)**

```yaml
trigger:
  - push to main/develop
  - pull requests
  - scheduled (6am daily)

workflow:
  - Checkout code
  - Setup Node.js
  - Install dependencies
  - Run tests (filters by @tags)
  - Generate reports
  - Upload artifacts
  - Notify Slack on failure
```

---

### 2. **Database Integration**

```
PostgreSQL
  ├─ test_results (every test)
  ├─ flaky_tests (per failure)
  └─ ...other metrics

↓ SQL Queries

Grafana Dashboards
  ├─ Pass rate trend
  ├─ Execution time
  ├─ Flaky test tracking
  └─ Environment comparison
```

---

### 3. **Reporting Integration**

```
Playwright HTML
  ├─ Test-level details
  ├─ Failure screenshots
  ├─ Video playback

JSON Results
  └─ Machine-readable format

Allure Reports (Optional)
  ├─ Test history
  ├─ Timeline view
  └─ Integration with JIRA

Logs (JSON)
  └─ ELK integration
```

---

### 4. **Notifications**

```
Slack:
  - Test run summary
  - Failed tests list
  - Pass rate change
  
Email:
  - Daily digest
  - Flaky test alerts
  
GitHub:
  - PR check status
  - Report link
```

---

## Security & Compliance

### 1. **Sensitive Data Masking**

```typescript
// Never log sensitive data
logger.error('Failed to authenticate', {
  username: user.email,           // ❌ Don't log
  token: 'secret-token-123',      // ❌ Don't log
  statusCode: 401                 // ✅ OK to log
});

// ✅ Mask sensitive fields
function maskSensitive(data: any) {
  return {
    ...data,
    email: data.email.replace(/@.*/, '@***'),
    token: '***MASKED***'
  };
}
```

---

### 2. **Environment Credential Management**

```bash
# ❌ Never commit .env
git status
  .env # (in .gitignore)

# ✅ Use GitHub Secrets for CI
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
  API_KEY: ${{ secrets.API_KEY }}

# ✅ Use .env.example as template
cat .env.example
  # DB_PASSWORD=your_password_here
  # API_KEY=your_api_key_here
```

---

### 3. **Test Data Isolation**

```typescript
// Each test has isolated data
test('Test 1', async () => {
  const farmer1 = await dataFactory.createFarmer();
  // Test uses farmer1.id
  // After test: DELETE farmer1
});

test('Test 2', async () => {
  const farmer2 = await dataFactory.createFarmer();
  // Test uses farmer2.id (NOT farmer1)
  // After test: DELETE farmer2
});

// Result: No cross-test data leakage
```

---

### 4. **Compliance Tracking**

```sql
-- Audit log for payment tests
CREATE TABLE test_audit_log (
  test_name VARCHAR,
  environment VARCHAR,
  data_used TEXT,           -- What test data was used
  timestamp TIMESTAMP,
  operator VARCHAR,         -- Who triggered (CI user)
  result VARCHAR            -- Pass/Fail
);

-- Track compliance violations
INSERT INTO test_audit_log VALUES (
  'payment test',
  'qa',
  'farmer-abc123, payment-xyz789',
  now(),
  'github-actions-ci',
  'PASSED'
);
```

---

## Summary: Why This Architecture

| Requirement | Solution | Why |
|---|---|---|
| **SOP Compliance** | Selector priority, 4 states, accessibility | Enforced by framework design |
| **Stability** | MSW mocking, deterministic IDs, isolation | No flaky tests |
| **Maintainability** | POM separation, centralized locators | Easy to update selectors |
| **Parallel Testing** | Isolated contexts, per-test cleanup | Can scale to N workers |
| **Debugging** | Logs, screenshots, videos, traces | Fast issue resolution |
| **Analytics** | PostgreSQL + Grafana | View trends over time |
| **Early Testing** | MSW + factories | Test before backend ready |
| **Accessibility** | jest-axe integration, semantic HTML | Compliance built-in |
| **Performance** | Headless, pooling, mocking | 5-10 tests in ≤5 min |
| **Production Ready** | Environment config, error handling, retries | Deploy to production confidence |

---

## Conclusion

This architecture is built on:
1. **Frontend SOP compliance** (strict selector strategy, 4-state testing, A11y)
2. **Industry best practices** (POM, MSW, test isolation)
3. **Enterprise patterns** (logging, observability, scalability)
4. **Team needs** (early testing, parallel execution, analytics)

It scales from **5 developers local testing → 500+ tests in CI/CD** without architectural changes.