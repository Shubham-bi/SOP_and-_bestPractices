
STANDARD OPERATING PROCEDURE
Software Quality &
Testing Engineering
Unit · Integration · API · E2E · Performance · Security

Test-Driven · Automation-First · Shift-Left · CI-Enforced
Version 1.0  ·  April 2026
 
0 · Purpose & Scope
This SOP defines how engineers design, write, organise, and automate tests across the full software stack. It applies to every team member who writes code, reviews PRs, or builds CI pipelines.
It covers all six testing disciplines: Unit, Integration, API, End-to-End, Performance, and Security. Each discipline has defined ownership, tooling standards, naming conventions, and CI enforcement rules.

GOAL:  Tests are not a phase at the end of development. They are a continuous engineering activity that defines quality, documents behaviour, and prevents regression. Every engineer owns test quality.

Who This Applies To
•	Developers writing features (frontend, backend, full-stack)
•	QA engineers building and maintaining automated test suites
•	DevOps / platform engineers wiring CI/CD pipelines
•	Tech leads and architects reviewing test strategy and coverage
•	Engineering managers setting quality gates and release criteria

1 · Testing Philosophy & Core Principles

Principle	What It Means in Practice
Shift Left	Find bugs at design and unit test time — not in QA or production. The earlier a bug is caught, the cheaper it is to fix.
Test Behaviour, Not Code	Tests describe what the system does, not how it does it. Internal refactors must not break tests unless behaviour changes.
Automation by Default	Every repeatable check must be automated. Manual testing is reserved for exploratory sessions and one-off investigations only.
Tests Are Documentation	A test suite is the most accurate documentation of system behaviour. Tests must be readable by anyone, not just the original author.
Determinism Is Non-Negotiable	Flaky tests are bugs. A test that fails without a code change erodes team trust and must be quarantined and fixed within 24 hours.
Isolation Over Integration	Prefer testing the smallest possible unit with all external dependencies mocked. Only integrate when integration is the thing being tested.
Test the Contract, Not the Implementation	For API and integration tests, assert on the observable contract (response shape, DB state). Never assert on internal variable values.
Coverage Is a Floor, Not a Target	Meeting a coverage threshold is the minimum bar. High coverage with low-quality assertions is worthless. Both quantity and assertion quality matter.

2 · Test Types, Ownership & Tools

2.1  The Testing Pyramid
The pyramid defines the ideal ratio of test types. More tests at the bottom (fast, cheap, isolated), fewer at the top (slow, expensive, fragile). Violating this ratio increases cost and flakiness.

E2E / Journey Tests  ·  2%
Performance & Security Tests  ·  5%
API / Contract Tests  ·  8%
Integration Tests  ·  20%
Unit Tests  ·  65%

2.2  Test Type Reference
Type	Scope	Execution Speed	Primary Owner
Unit	Single function / class / hook in complete isolation	< 5ms per test	Developer (mandatory)
Integration	2+ layers together — service + DB, component + store	50–500ms per test	Developer (mandatory for STRICT)
API	Full HTTP request → response cycle with test DB	200ms–2s per test	Developer + QA
E2E	Full user journey across real UI + backend	5–30s per test	QA (developer assists)
Performance	Load, stress, spike testing against staging	Minutes per run	Developer + DevOps
Security	Vulnerability scanning, auth bypass, injection probing	Minutes per scan	Security + Developer

2.3  Tool Stack
Test Type	Recommended Tools
Unit (Frontend)	Jest / Vitest, React Testing Library, jest-axe
Unit (Backend)	Jest / Vitest, ts-jest
Integration (Frontend)	Vitest with jsdom, MSW for network mocking
Integration (Backend)	Jest + Testcontainers (Postgres, Redis), Supertest
API / Contract	Supertest, jest-openapi, Schemathesis, Pact (consumer-driven)
E2E	Playwright (preferred), Cypress (acceptable for legacy)
Performance / Load	k6 (preferred), Artillery, Locust
Security (DAST)	OWASP ZAP, Burp Suite (manual), Semgrep (SAST)
Accessibility	jest-axe, axe-core, Playwright axe plugin
Visual Regression	Playwright screenshots + pixelmatch, Chromatic (Storybook)
Dependency Audit	npm audit, Snyk, Dependabot
Mock / Stub	MSW (network), nock (HTTP), Sinon, jest.fn() / vi.fn()

3 · Unit Testing Standards
Unit tests are the foundation of the test suite. They run in milliseconds, require no infrastructure, and validate every individual behaviour in isolation.

3.1  What to Unit Test
Test This	Do NOT Test This
Business logic: calculations, validations, transformations	Framework internals (Express routing, React reconciliation)
Every code path through a conditional branch	Third-party library implementations
Every domain error / exception condition	Simple pass-through getters with no logic
Edge cases: nulls, empty arrays, max/min values, type coercions	Constants or configuration objects
Custom hooks (frontend), service methods (backend)	Auto-generated code (ORM migrations, GraphQL resolvers from schema)
Pure utility and helper functions	Private methods — test through public interface only

3.2  Structure — Arrange, Act, Assert (AAA)
Every unit test must follow the AAA pattern explicitly. The three sections must be visually separated. This makes tests readable by anyone and ensures a single behaviour per test.

describe('calculateDiscount', () => {
  it('should apply percentage discount to subtotal', () => {
    // ── Arrange ─────────────────────────────────────────────
    const subtotal = 10000; // pence
    const coupon   = { type: 'PERCENT', value: 10 };

    // ── Act ─────────────────────────────────────────────────
    const result = calculateDiscount(subtotal, coupon);

    // ── Assert ──────────────────────────────────────────────
    expect(result.discountAmount).toBe(1000);
    expect(result.finalTotal).toBe(9000);
  });
});

3.3  Test Naming Convention
Pattern	Example
should [behaviour] when [condition]	should return 0 discount when coupon is expired
[function] [condition] → [expected result]	calculateDiscount with PERCENT coupon → deducts percentage from total
[component] renders [element] when [state]	UserCard renders 'Suspended' badge when status is suspended

3.4  Coverage Rules
Layer / Mode	Minimum Coverage Requirement
Any code (LEAN)	60% statement coverage
Any code (STRICT)	80% statement coverage
Service / business logic layer (STRICT)	95% branch coverage — mandatory
Domain error paths	100% — every typed error must have a test
Pure utility functions	100% branch coverage
Auth and permission logic	100% — every permission state tested

3.5  Unit Test Anti-Patterns (Blocked)
Anti-Pattern	Correct Approach
Testing implementation details (private variable values)	Test observable output — what the function returns or throws
One giant test with 10+ assertions on unrelated things	One test per behaviour — split into individual it() blocks
Mocking your own module under test	Mock only external dependencies. If you mock your own code, you test nothing.
Using real timers (Date.now, setTimeout) in unit tests	jest.useFakeTimers() / vi.useFakeTimers() for all time-sensitive logic
Tests that pass with empty assertions (expect(true))	Every assertion must be specific and falsifiable
Skipped tests with no justification (it.skip, xit)	Remove the test or add a ticket reference explaining why it is temporarily disabled
RULE:  If a test could pass even when the code is completely broken, it is not a test — it is noise. Every assertion must be capable of failing.

4 · Integration Testing Standards
Integration tests verify that two or more layers work correctly together. They use real infrastructure (DB, cache, queue) in isolated containers — never shared environments.

4.1  Scope Definition
What Integration Tests Cover	What They Do NOT Cover
Service + Repository + real database	HTTP layer concerns (status codes, headers) — that is API testing
React component + real Redux store / Context	Business logic edge cases — that is unit testing
Database query correctness and index usage	Third-party SaaS behaviour (payment gateway, email provider)
Transaction rollback on failure	User journey flows — that is E2E testing
Event emission after persistence succeeds	Performance under load — that is performance testing
Migration correctness — apply, rollback, idempotency	Security probing — that is security testing

4.2  Database Isolation Rules
•	Each test suite must run against its own isolated database instance (Testcontainers or Docker Compose)
•	Never share a database between parallel CI workers — use unique DB names per worker
•	Seed data in beforeEach using factory builders — never assume pre-existing rows
•	Clean up using transaction rollback (fastest) or explicit table truncation after each test
•	Run migrations fresh against the test DB before each test run — never mock migrations

// ✅ Testcontainers setup — isolated real DB per test suite
let pg: StartedPostgreSqlContainer;

beforeAll(async () => {
  pg = await new PostgreSqlContainer('postgres:16').start();
  await runMigrations(pg.getConnectionUri());
});
afterAll(async () => { await pg.stop(); });
beforeEach(async () => { await db.truncateAllTables(); });

4.3  Frontend Integration Test Rules
•	Use the real store / context — never mock Redux or React Context in integration tests
•	Use MSW to intercept network requests — never mock the fetch module directly
•	Assert on rendered output — never assert on state variables or store internals
•	Simulate real user interactions (userEvent.click, userEvent.type) — not fireEvent

// ✅ Frontend integration — real store + MSW network mock
server.use(
  rest.get('/api/user/me', (req, res, ctx) =>
    res(ctx.json({ id: 'u1', name: 'Alice', role: 'admin' }))
  )
);
render(<UserDashboard />, { wrapper: storeProvider });
expect(await screen.findByText('Alice')).toBeInTheDocument();
expect(screen.getByRole('button', { name: /admin settings/i })).toBeVisible();

4.4  Integration Test Anti-Patterns
Anti-Pattern	Fix
Using a shared, persistent test DB	Use Testcontainers or a per-suite DB — isolated per run
Mocking the DB in an integration test	Integration tests exist to test DB interaction — mock = unit test
Asserting on return values only, not DB state	After a mutation, query the DB directly to verify the state was persisted
Integration tests with 5-second arbitrary delays	Wait on DB event, polling assertion, or conditional waitFor — never fixed sleep

5 · API Testing Standards
API tests make real HTTP calls to an in-process server with a seeded test database. They own the HTTP contract: status codes, response schemas, headers, and error envelopes.

5.1  Automation-First Design Requirements
Tests are only reliably automatable when the API is designed for it. These rules apply to every STRICT endpoint.

Design Rule	Why It Enables Automation
Assign operationId to every OpenAPI path	Test runners reference endpoints by operationId — stable across path renames
Machine-readable error.code in every error response	Assertions use error.code, not error.message — messages can change freely
traceId in every response (success + error)	Correlates test failures to server logs instantly during investigation
Deterministic sort order on all list endpoints	Index-based assertions are stable across seed data volume changes
Idempotency-Key on all mutation endpoints	Test setup can re-run without creating duplicate data
ISO 8601 UTC for all timestamps	Prevents locale-dependent failures in CI environments
Stable pagination contract (cursor or page/limit)	Tests do not break when seed data volume grows

5.2  Mandatory Status Code Coverage Matrix
Every STRICT API endpoint must be tested for every applicable status code listed below.

Status Code	Trigger Condition in Test	Assert On
200 OK	Valid request, existing resource	Schema match (jest-openapi) + traceId
201 Created	Valid POST creates new resource	Location header + schema + DB row exists
204 No Content	Valid DELETE	Empty body + DB row deleted
400 Bad Request	Missing required field or wrong data type	error.code = VALIDATION_ERROR + field name
401 Unauthorized	Missing token / invalid / expired token	error.code = MISSING_TOKEN / INVALID_TOKEN / TOKEN_EXPIRED
403 Forbidden	Valid token, wrong role, or non-owner	error.code = INSUFFICIENT_PERMISSIONS
404 Not Found	Resource ID does not exist	error.code = [RESOURCE]_NOT_FOUND
409 Conflict	Duplicate resource or idempotency key conflict	error.code = DUPLICATE_[RESOURCE]
422 Unprocessable	Schema valid but business rule violated	error.code matches domain error constant
429 Too Many Req.	Rate limit exceeded	Retry-After header present
500 Internal	Forced throw (test environment only)	No stack trace or internal path in response body

5.3  Self-Contained Test Runner Setup
API tests must run with a single command and require no external servers to be manually started.

// ✅ Standard self-contained API test bootstrap
import { createApp } from '../app';
import request     from 'supertest';
import { db }      from '../database';

let app: Express, server: Server;

beforeAll(async () => {
  await db.migrate.latest();          // Real migrations
  app    = await createApp();
  server = app.listen(0);             // Ephemeral port
});
afterAll(async () => {
  await server.close();
  await db.destroy();
});
beforeEach(async () => {
  await db.truncateAllTables();       // Clean slate
  await seedFixtures();              // Minimal test data
});

5.4  Contract Testing with OpenAPI
•	Commit openapi.yaml before any implementation — contract is the source of truth
•	Use jest-openapi to assert every response satisfies the spec: expect(res).toSatisfyApiSpec()
•	Use Schemathesis in CI to auto-generate fuzz inputs from the spec — free edge case coverage
•	Run Pact for consumer-driven contract tests when multiple teams depend on the same API
•	Any response that fails schema validation is a breaking change — it blocks the pipeline immediately

// ✅ OpenAPI contract assertion in every happy-path test
import { setupFilesAfterEach } from 'jest-openapi';
setupFilesAfterEach('./openapi.yaml');

it('GET /v1/users/:id matches OpenAPI contract', async () => {
  const res = await request(app)
    .get('/v1/users/user-1')
    .set('Authorization', `Bearer ${validToken}`);
  expect(res.status).toBe(200);
  expect(res).toSatisfyApiSpec();   // Schema + required fields validated
  expect(res.body.meta.traceId).toBeDefined();
});

5.5  Authentication Test Matrix (Mandatory)
Auth State	Required Assertion
No token present	401  ·  error.code: MISSING_TOKEN  ·  No data leaked
Malformed / invalid token	401  ·  error.code: INVALID_TOKEN
Expired token	401  ·  error.code: TOKEN_EXPIRED
Valid token — insufficient role	403  ·  error.code: INSUFFICIENT_PERMISSIONS
Valid token — accessing another user's resource	403 or 404 (to prevent enumeration)
Valid token — correct role and ownership	Expected success response matching schema

5.6  API Test Anti-Patterns
Anti-Pattern	Correct Approach
Using API tests to validate business logic branches	Business logic belongs in unit tests. API tests validate HTTP contract.
Asserting on error.message strings	Assert on error.code — messages are human-readable and can change without notice.
Snapshot testing response bodies	Use explicit schema assertions (jest-openapi). Snapshots break on any field addition.
No negative tests for any endpoint	Every endpoint needs at least one error path test.
Starting a real server in a separate terminal	Use in-process server (createApp().listen(0)) for self-contained, portable tests.
Sharing DB state between test files	Each test file seeds its own data in beforeEach. Shared state = order-dependent failures.

6 · End-to-End Testing Standards
E2E tests validate complete user journeys through the real application stack. They run in a staging environment, are the slowest and most expensive tests, and should be the fewest in number.

6.1  When to Write E2E Tests
RULE:  E2E tests cover user journeys — sequences of actions a real user would take. They do NOT validate business logic, API contracts, or individual component behaviour. Those belong in lower layers.

Suitable for E2E	NOT Suitable — Use Lower Layer Instead
User registration → email verification → login → dashboard	Validating that a single API returns 422 for invalid input
Product search → add to cart → checkout → confirmation email	Testing that a React component renders correctly
Password reset full flow (request → email → reset page → new login)	Validating a business rule edge case
Admin creates user → user logs in and sees their account	Testing auth middleware in isolation

6.2  Playwright Standards
•	Use data-testid attributes for E2E selectors — semantic selectors are fragile in E2E context
•	data-testid format: kebab-case, feature-scoped — checkout-submit-button, cart-item-remove-P1
•	Never use CSS classes, XPath, or text content as E2E selectors
•	All E2E tests run against a dedicated staging environment — never dev, never production
•	E2E tests must be independent — each test creates its own user/data via API setup calls
•	Record a Playwright trace on failure — attach to CI artifacts for debugging

// ✅ Playwright test — full user journey with API-driven setup
test('user can complete checkout journey', async ({ page }) => {
  // Setup: create test user and product via API (not UI)
  const { token, user } = await api.createTestUser();
  await api.seedProduct({ id: 'P1', stock: 10, price: 2999 });

  // Act: drive the UI as a real user
  await page.goto('/login');
  await page.getByLabel('Email').fill(user.email);
  await page.getByLabel('Password').fill(user.password);
  await page.getByRole('button', { name: /sign in/i }).click();
  await page.getByTestId('product-card-P1').click();
  await page.getByRole('button', { name: /add to cart/i }).click();
  await page.getByTestId('checkout-submit-button').click();

  // Assert: journey completed
  await expect(page.getByTestId('order-confirmation-banner')).toBeVisible();
});

6.3  Selector Strategy for E2E
Priority	When to Use
1. getByRole (ARIA)	Preferred for interactive elements — buttons, inputs, links
2. getByLabel	Form inputs with visible labels
3. getByTestId	Non-semantic elements — panels, cards, banners
4. NEVER: CSS class, XPath, text content	Fragile — breaks on styling or copy changes

6.4  E2E Stability Rules
•	Never use arbitrary page.waitForTimeout(2000) — use page.waitForSelector or expect().toBeVisible()
•	Disable CSS animations and transitions in E2E test runs using a global test class
•	Mock third-party services (payments, email, SMS) at the network level — never call real providers in E2E
•	Set a fixed viewport size per test to prevent responsive layout inconsistencies
•	Use page.route() to intercept and stub flaky external APIs that are not under test
•	Parallelise E2E test files — never test cases within the same file

// playwright.config.ts — stability configuration
export default defineConfig({
  use: {
    baseURL:   process.env.E2E_BASE_URL ?? 'http://localhost:3000',
    viewport:  { width: 1280, height: 720 },
    video:     'retain-on-failure',
    trace:     'retain-on-failure',
  },
  fullyParallel: true,
  retries:       process.env.CI ? 2 : 0,  // Retry only in CI
  reporter:      [['html'], ['junit', { outputFile: 'results.xml' }]],
});

6.5  E2E Anti-Patterns
Anti-Pattern	Correct Approach
Using E2E to test a business rule	Business rules belong in unit/integration tests — test the rule there
UI-driven test data setup (filling forms to create data)	Use API calls or direct DB seed for setup — E2E tests the journey, not the setup
Tests that depend on other tests' data	Every test is fully independent — creates and destroys its own data
Running E2E against production	E2E runs against staging only — production is never a test target
Ignoring flaky tests as 'sometimes fail'	Quarantine immediately. Retry logic masks a real bug — find the root cause

7 · Performance Testing Standards
Performance tests prevent regressions in response time, throughput, and resource usage. They run against a staging environment that mirrors production capacity.

7.1  Performance Test Types
Test Type	Purpose & When to Run
Smoke Test	1–5 VUs for 1 minute. Validates the system works under minimal load. Run on every staging deployment.
Load Test	Ramp to expected production peak load. Validate p95 SLAs are met. Run before every major release.
Stress Test	Ramp beyond expected peak to find the breaking point. Run quarterly or after architecture changes.
Spike Test	Sudden 10× traffic increase. Validate auto-scaling behaviour and graceful degradation. Run before campaigns.
Soak / Endurance	Sustained load over 1–4 hours. Detect memory leaks and connection pool exhaustion. Run monthly.
Volume Test	Large data sets (1M+ rows). Validate query performance does not degrade with scale. Run quarterly.

7.2  Response Time Baselines (SLAs)
Endpoint Category	p95 Target (Max)
Read — simple lookup by ID	< 100ms
Read — filtered list with pagination	< 300ms
Write — simple create / update	< 500ms
Write — multi-step transaction	< 1,000ms
Authentication endpoint	< 300ms
Search with full-text or complex filter	< 800ms
Report / aggregate query	< 3,000ms (with explicit loading state in UI)
Background job completion	< 5s SLA on completion (not enqueue latency)

7.3  k6 Test Standards
•	Tests must define: target VUs, ramp-up duration, steady-state duration, ramp-down
•	Define thresholds inline — CI fails automatically if thresholds are exceeded
•	Parameterise base URL and auth tokens via environment variables — never hardcode
•	Include think time (sleep) between requests to simulate real user behaviour
•	Collect per-endpoint metrics — not just aggregate — to identify bottleneck endpoints

// ✅ k6 load test with enforced thresholds
import http  from 'k6/http';
import { sleep, check } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 50  },  // Ramp up
    { duration: '2m',  target: 50  },  // Steady state
    { duration: '15s', target: 0   },  // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<300'],  // p95 < 300ms — CI fails if exceeded
    http_req_failed:   ['rate<0.01'],  // Error rate < 1%
  },
};

export default function () {
  const res = http.get(`${__ENV.BASE_URL}/v1/orders`, {
    headers: { Authorization: `Bearer ${__ENV.TEST_TOKEN}` },
  });
  check(res, { 'status is 200': (r) => r.status === 200 });
  sleep(1);
}

7.4  Query Performance Rules
•	Every DB query used in a production endpoint must use an index — verified with EXPLAIN ANALYZE
•	N+1 query patterns are bugs — detected with query count assertions in integration tests
•	Declare and enforce a performanceBudget per STRICT endpoint: p95, maxDbQueries, maxExternalCalls
•	All list endpoints paginated — no unbounded SELECT * queries ever
•	Foreign key indexes on all join columns — validate on schema creation

8 · Security Testing Standards
Security tests validate that the system is resistant to the most common attack patterns. They are not a substitute for a security audit — they are the minimum baseline every team owns.

8.1  OWASP Top 10 — Minimum Test Coverage
Vulnerability	How to Test It
Injection (SQL, NoSQL, Command)	Send ', OR 1=1--, $where, ; DROP TABLE -- in all string parameters. Assert sanitised response.
Broken Authentication	Test all 6 auth states (Section 5.5). Test token expiry, revocation, and replay.
Sensitive Data Exposure	Assert passwords, tokens, secrets never appear in response bodies or logs.
Broken Access Control (IDOR)	Access resource IDs belonging to other users with a valid token. Assert 403/404.
Security Misconfiguration	Assert CORS headers, security headers (CSP, HSTS, X-Frame), and no debug endpoints in staging.
Mass Assignment	Attempt to POST/PUT protected fields (role, id, balance). Assert they are ignored.
Using Components with Known CVEs	npm audit / Snyk on every CI build. Block on high/critical findings.
Insufficient Logging / Monitoring	Assert every 4xx and 5xx emits a structured log with traceId and userId.

8.2  Security Test Implementation
Test Type	Implementation Approach
SAST (Static Analysis)	Semgrep / ESLint security plugins run on every PR. Block on high severity.
DAST (Dynamic Analysis)	OWASP ZAP baseline scan runs on every staging deployment.
Dependency Audit	npm audit and Snyk on every commit. Fail CI on critical CVEs.
Secret Scanning	TruffleHog / GitLeaks scan every commit. Block on any finding.
Penetration Test (Manual)	Quarterly by security team or external firm for STRICT/critical services.
Auth Bypass Tests	Automated suite testing all auth states + token replay + role escalation.

8.3  Security Test Code Examples
// ✅ IDOR test — user cannot access another user's resource
it('returns 403 when accessing another user data', async () => {
  const targetUser = await seedUser({ id: 'victim' });
  const attackerToken = createTestToken({ userId: 'attacker' });

  const res = await request(app)
    .get(`/v1/users/victim/orders`)
    .set('Authorization', `Bearer ${attackerToken}`);

  expect(res.status).toBeOneOf([403, 404]);
  expect(res.body).not.toHaveProperty('data');
});

// ✅ Mass assignment test
it('ignores protected fields in POST body', async () => {
  const res = await request(app)
    .post('/v1/users')
    .set('Authorization', bearer)
    .send({ name: 'Eve', email: 'e@test.com', role: 'admin' }); // Try to escalate

  expect(res.status).toBe(201);
  expect(res.body.data.role).toBe('customer');  // Protected — must be ignored
});

9 · Test Data Management
9.1  Core Principles
•	Tests own their data — never assume pre-existing DB rows from a shared fixture file
•	Seed only the minimum fields relevant to the behaviour under test
•	Use factory builders with sensible defaults and per-test overrides
•	Never use real PII, email addresses, phone numbers, or payment details in test data
•	Never use production data snapshots in automated test environments
•	Dates in fixtures must be deterministic — use fixed ISO strings, not new Date()

9.2  Factory Builder Pattern (Universal)
// ✅ Factory with safe defaults — override only what the test cares about
const buildUser = (overrides: Partial<User> = {}): User => ({
  id:        `user-${crypto.randomUUID()}`,
  email:     `test+${Date.now()}@example.com`,  // example.com — safe domain
  name:      'Test User',
  role:      'customer',
  status:    'active',
  createdAt: new Date('2024-01-15T10:00:00Z'),  // Fixed — never new Date()
  ...overrides,
});

// ── Usage ───────────────────────────────────────────────────────────────
const admin      = buildUser({ role: 'admin' });
const suspended  = buildUser({ status: 'suspended' });
const targetUser = buildUser({ id: 'known-id-for-IDOR-test' });

9.3  Test Database Reset Strategies
Strategy	When to Use
Transaction rollback (fastest)	Wrap each test in a transaction; rollback in afterEach. Best for integration tests with heavy seed.
Table truncation (recommended)	TRUNCATE all tables in afterEach. Simple, reliable. Slightly slower than rollback.
Schema drop & recreate (slowest)	Drop and rebuild DB per test suite. Use only for migration tests.
Shared seed read-only (acceptable)	Global seed once for read-only test suites only — never for write tests.

NEVER:  Allow multiple developers or CI workers to share a persistent test database simultaneously. Concurrent writes produce non-deterministic test failures that are nearly impossible to debug.

10 · Mocking & Stubbing Standards
10.1  Mock Decision Matrix
What You're Testing	What to Mock
Business logic (unit)	Mock: repository, external HTTP, email service. Real: domain logic, validator, error classes.
DB query correctness (integration)	Mock: nothing. Use real DB container. This is the point.
HTTP API contract (API test)	Mock: external downstream services (via MSW/nock). Real: DB, auth middleware, business logic.
User journey (E2E)	Mock: third-party providers (Stripe, SendGrid). Real: your entire application stack.
Worker / async handler (unit)	Mock: all dependencies. Real: the handler function logic.

10.2  The Mock Boundary — Golden Rule
RULE:  Mock at the boundary of your system — external DB, HTTP, queue, clock, randomness. Never mock your own service classes, domain errors, or business logic. If you mock your own code, you are testing a simulation, not your system.

10.3  MSW — Network-Level Mocking
•	MSW is the standard for mocking outbound HTTP in both unit and integration tests
•	Define handlers in /test/msw-handlers/ — shared across test types
•	Reset handlers in afterEach — prevent handler state from bleeding across tests
•	Use msw/node for backend tests; msw/browser for frontend tests

// ✅ MSW handler setup — shared across test suite
import { setupServer } from 'msw/node';
import { rest }        from 'msw';

export const server = setupServer(
  rest.post('https://api.stripe.com/v1/charges', (req, res, ctx) =>
    res(ctx.json({ id: 'ch_test_123', status: 'succeeded' }))
  ),
);

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(()  => server.close());

10.4  Time, Randomness & IDs
•	Always inject time as a dependency or use jest.useFakeTimers() — never call new Date() or Date.now() directly in tested code
•	UUIDs and random IDs must be seed-controlled or injected — use a deterministic generator in tests
•	Test time-sensitive logic by controlling the clock: expiry checks, token TTL, scheduled job firing

// ✅ Controlling time in tests
beforeEach(() => { jest.useFakeTimers(); jest.setSystemTime(new Date('2026-01-15')); });
afterEach(()  => { jest.useRealTimers(); });

it('rejects expired coupon at boundary', () => {
  const coupon = { expiresAt: new Date('2026-01-15T00:00:00Z') };  // Exactly now
  expect(() => applyCoupon(coupon)).toThrow(CouponExpiredError);
});

11 · CI/CD Integration & Quality Gates
11.1  Fail-Fast Pipeline Order
Cheaper, faster tests run first. The pipeline stops at the first failure to save time and provide rapid developer feedback.

Stage	Gates — Pipeline Stops if Any Gate Fails
1. Static Analysis	Lint · TypeScript type check · Secret scan (TruffleHog) · Boundary violation rules
2. Unit Tests	All unit tests pass · Per-file coverage threshold met · No unjustified skipped tests
3. Contract Validation	OpenAPI schema diff check · Mode declaration present · Breaking change detection
4. Integration Tests	Migrations apply cleanly · All integration tests pass · DB constraint violations caught
5. API / Contract Tests	Full status code matrix · jest-openapi schema assertions · Auth matrix complete
6. Dependency + SAST	npm audit no critical CVEs · Semgrep no high severity findings
7. Performance Smoke	(Staging) p95 within declared budget · Query count within budget
8. DAST Scan	(Staging) OWASP ZAP baseline passing · No new critical findings
9. E2E Journey Tests	(Staging) Critical user journeys complete · Playwright reports clean

11.2  Coverage Gates per Pipeline Stage
Mode / Layer	Threshold
LEAN — overall	60% statement coverage
STRICT — overall	80% statement coverage
STRICT — service / business logic	95% branch coverage — hard gate
Domain error paths	100% — every domain error must have at least one test
Auth and permission paths	100% — every auth state tested
Idempotency paths	100% — single + concurrent duplicate key scenarios

11.3  PR-Level Gates
•	PR blocked if coverage drops more than 2% from the base branch
•	PR blocked if any new domain error class has no corresponding test
•	PR blocked if OpenAPI spec is not updated for any response shape change (STRICT)
•	PR blocked if performance budget is missing for new STRICT endpoints
•	PR blocked if any hardcoded sleep() or arbitrary timeout is introduced in tests

11.4  Test Reporting Requirements
•	All test runners output JUnit XML format — consumed by CI dashboard
•	Coverage reports published as HTML artifacts on every PR
•	E2E failures attach Playwright trace + video + screenshot automatically
•	Flaky test rate tracked per suite — alert if > 2% of runs non-deterministic
•	Weekly test health report: flaky count, coverage trend, slowest tests

FLAKY TEST POLICY:  A test that fails without a code change is a defect. Quarantine within 24 hours. Root cause and fix within one sprint. Teams with > 3% flaky rate must halt feature work and fix the suite first. No exceptions.

12 · Accessibility Testing
Accessibility testing is not optional. WCAG 2.1 AA is the minimum compliance target. Accessible design also directly enables better test automation through semantic selectors.

12.1  Automated Accessibility Rules
•	Integrate jest-axe in every component render test — run axe on every component
•	Playwright axe plugin runs on every E2E journey page visited
•	Lighthouse CI runs on staging deployments — fail on score regression > 5 points
•	Any critical axe violation blocks the PR — serious violations require a remediation ticket

// ✅ jest-axe in every component unit test
import { axe, toHaveNoViolations } from 'jest-axe';
expect.extend(toHaveNoViolations);

it('has no accessibility violations', async () => {
  const { container } = render(<CheckoutForm />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

12.2  Accessibility Selector Benefit
Semantic selectors are both accessibility-compliant and the most stable selectors for automation. They are preferred in this exact order:

Selector Priority	Example
1. getByRole (ARIA semantic)	getByRole('button', { name: /submit order/i })
2. getByLabel (form association)	getByLabelText('Email address')
3. getByPlaceholderText	getByPlaceholderText('Search products...')
4. getByTestId (fallback only)	getByTestId('order-summary-panel')
NEVER: getByClassName, querySelector('.btn')	Fragile — breaks with CSS refactors, no semantic meaning

13 · Defect Classification & Management
13.1  Severity Classification
Severity	Definition	Response Time	Example
P1 — Critical	Production system down or data corrupted	Hotfix within 2 hours	Payment processing broken; user data overwritten
P2 — High	Core feature broken or major security vulnerability	Fix in current sprint	Login fails for 20% of users; IDOR vulnerability found
P3 — Medium	Feature degraded but workaround exists	Fix in next sprint	Pagination returns wrong order; error message is misleading
P4 — Low	Minor UX issue, edge case, or cosmetic problem	Fix in backlog	Incorrect placeholder text; mobile button slightly misaligned

13.2  Bug Report Minimum Requirements
•	Title: [Component/Endpoint] — what breaks — under what condition
•	Steps to reproduce: numbered, minimal, deterministic
•	Expected behaviour: what should happen
•	Actual behaviour: what actually happens
•	Environment: browser/OS/version or API version/deployment SHA
•	Reproduction rate: always / intermittent (N/M runs) / once
•	Failing test: link to CI run or paste the failing test output

13.3  Test-First Defect Resolution
PROCESS:  Before fixing any bug: (1) Write a failing test that reproduces the defect. (2) Confirm the test fails. (3) Fix the code. (4) Confirm the test now passes. This guarantees the bug cannot silently regress.

14 · Test Environment Management
14.1  Environment Tiers
Environment	Purpose & Rules
Local Dev	Unit + integration tests only. Developer-controlled. No shared services. Uses Docker containers for DB/cache.
CI (ephemeral)	All tests up to API tests. Spun up fresh per PR. Uses Testcontainers. Destroyed after run.
Staging	E2E, performance, DAST, and security tests. Mirrors production config. Seeded with anonymised data. Never manual QA target.
Production	Zero automated tests. Passive monitoring (synthetic probes) only. No test data ever written here.

14.2  Environment Isolation Rules
•	No test ever targets production — blocked at the network/IAM level
•	Staging must use non-production API keys for all third-party services (Stripe test mode, SendGrid sandbox)
•	Environment variables for test targets are injected by CI — never hardcoded in test files
•	Test databases are ephemeral — they do not persist between CI runs
•	Staging is reset to a known seed state at the start of every E2E run

CRITICAL:  If a test accidentally runs against production and writes data, treat it as an incident. Implement network-level blocks and IAM restrictions to prevent recurrence. Audit logs must be reviewed.

15 · Test Metrics & Quality Reporting
15.1  Key Test Health Metrics
Metric	Target & Alert Threshold
Overall coverage (STRICT)	≥ 80% statement — alert if drops below 75%
Service layer branch coverage	≥ 95% — hard CI gate
Flaky test rate	< 1% of total runs — alert at 2%, halt feature work at 3%
API test suite runtime (CI)	< 60 seconds — alert if > 90 seconds
Unit test suite runtime (CI)	< 30 seconds — alert if > 45 seconds
E2E pass rate (staging)	≥ 98% — release blocked below 95%
P1 defect rate	0 in production — any P1 triggers post-mortem
Mean time to detect (MTTD)	< 1 hour for P1/P2 via automated monitoring
Mean time to resolve (MTTR)	< 4 hours P1 / < 1 sprint P2

15.2  Weekly Test Health Report
•	Flaky tests: list of tests that failed without a code change in the last 7 days
•	Coverage trend: coverage delta per feature area over the last 4 weeks
•	Slowest tests: top 10 slowest tests per type — candidates for optimisation
•	New skipped/disabled tests: any test added as skip requires justification and ticket
•	Defect escape rate: bugs found in production that had no corresponding test — root cause documented

16 · Quick Decision Checklist
Run through this checklist before opening any PR. Each item must be ticked or explicitly marked N/A with a reason.

	Checklist Item	When
☐	Unit tests written for every new function, hook, or service method	Mandatory
☐	Every conditional branch has a test — not just the happy path	Mandatory
☐	Tests follow Arrange–Act–Assert structure with clear section separation	Mandatory
☐	Test names describe behaviour, not implementation (should X when Y)	Mandatory
☐	All time-dependent logic uses fake timers — no real Date.now() in tests	Mandatory
☐	No hardcoded sleep() or setTimeout delay in any test	Mandatory
☐	Each test seeds its own data in beforeEach — no shared DB state	Mandatory
☐	No PII, real email addresses, or production data in any test fixture	Mandatory
☐	jest-axe (or equivalent) runs on every component render test	Mandatory
☐	API tests assert on error.code — never on error.message strings	Mandatory
☐	Full auth matrix covered: missing token, invalid, expired, wrong role, cross-user	Strict
☐	Full status code matrix covered for all STRICT endpoints	Strict
☐	jest-openapi contract assertion present on every happy-path API test	Strict
☐	Integration tests assert on DB state after mutations — not just return values	Strict
☐	Performance budget declared and DB query count test passing	Strict
☐	Security baseline: IDOR, mass assignment, oversized payload tests passing	Strict
☐	OWASP ZAP baseline scan passes on staging before release	Strict
☐	E2E tests use data-testid selectors — no CSS classes or XPath	E2E
☐	E2E test data created via API — not via UI form interactions	E2E
☐	k6 / Artillery load test passes p95 threshold before major release	Perf
☐	Coverage meets threshold: 60% LEAN / 80% STRICT / 95% service layer	Both

17 · Do's and Don'ts — At a Glance
17.1  Writing Tests
✅  DO	❌  DON'T
Follow AAA: Arrange, Act, Assert in every test	Write one giant test that asserts 10 unrelated things
Name tests as behaviour descriptions	Name tests test1, myTest, or checkThing
Seed data in beforeEach with factory builders	Rely on leftover data from a previous test or global fixture
Assert on what the system outputs or persists	Assert on internal state, private variables, or implementation
Use jest.useFakeTimers for time-sensitive logic	Call new Date() or Date.now() directly in tested code
Test one behaviour per test case	Use multiple act/assert cycles in a single it() block

17.2  Selectors & Assertions
✅  DO	❌  DON'T
getByRole('button', { name: /submit/i })	querySelector('.btn-primary.submit')
Assert on error.code for API error assertions	Assert on error.message — it changes without notice
expect(res).toSatisfyApiSpec() for API contract	Snapshot test response bodies
data-testid with feature prefix for E2E selectors	Use nth-child, sibling CSS selectors, or XPath in E2E
Assert on final stable DB state after mutations	Assert only on return values when a mutation test is needed

17.3  Mocking
✅  DO	❌  DON'T
Mock at system boundaries: DB, HTTP, queue, clock	Mock your own service methods or domain logic
Use MSW for network-level HTTP mocking	Mock the fetch or axios module directly
Use Testcontainers for real DB in integration tests	Mock the DB in an integration test — defeats the purpose
Inject dependencies to enable mocking without module rewire	Use jest.mock() on your own modules
Reset MSW handlers in afterEach	Share MSW handler state across tests without resetting

17.4  CI & Stability
✅  DO	❌  DON'T
Run tests in fail-fast order: lint → unit → integration → API	Run expensive E2E before unit tests pass
Quarantine flaky tests within 24 hours	Ignore flaky tests as 'sometimes they just fail'
Use retry logic in Playwright (max 2) for true E2E flakiness	Use retry to mask a genuine bug in the application
Enforce coverage thresholds as hard CI gates	Treat coverage as a soft guideline — it will decay
Track flaky test rate and act when > 2%	Accumulate flaky tests until the suite is untrustworthy

Appendix · Key Terms
Term	Definition
AAA	Arrange–Act–Assert: the mandatory structure for every test case
DAST	Dynamic Application Security Testing — probing a running application for vulnerabilities
SAST	Static Application Security Testing — analysing source code without running it
Contract Test	Verifies an API response matches a declared schema (OpenAPI or Pact)
Flaky Test	A test that produces inconsistent results across runs without any code change
Factory Builder	A function that creates test data objects with safe defaults and per-test overrides
IDOR	Insecure Direct Object Reference — accessing another user's resource via a guessable ID
MSW	Mock Service Worker — intercepts HTTP at the network level for mocking in tests
operationId	OpenAPI field providing a stable identifier for automation to reference an endpoint
Testcontainers	Library that spins up disposable Docker containers (Postgres, Redis) for integration tests
Schemathesis	Tool that auto-generates test inputs from an OpenAPI spec to find violations and edge cases
WCAG 2.1 AA	Web Content Accessibility Guidelines v2.1 Level AA — the minimum accessibility compliance target
Smoke Test	Minimal load test (1–5 VUs) that validates basic functionality under low traffic
DLQ	Dead Letter Queue — holds messages that failed processing after maximum retry attempts
Shift Left	The practice of moving testing earlier in the development lifecycle to find bugs sooner
p95	The 95th percentile response time — 95% of requests complete within this duration
MTTD	Mean Time to Detect — average time from bug introduction to discovery
MTTR	Mean Time to Resolve — average time from detection to production fix

— End of Document  ·  Testing SOP v1.0  ·  April 2026 —
