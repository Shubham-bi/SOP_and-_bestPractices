
STANDARD OPERATING PROCEDURE
Software Quality &
Testing Engineering
ISTQB-Aligned · Agile SDLC/STLC · Automation-First · CI-Enforced

Unit · Integration · API · E2E · Performance · Security · Observability
Version 2.0  ·  April 2026
 
What Changed — v1.0 → v2.0
This version upgrades the SOP from a strong foundational document into a mature, ISTQB-aligned engineering system. It incorporates Agile SDLC/STLC integration, relaxes overly rigid thresholds to reflect real-world team constraints, and adds the missing meta-layers: observability testing, PR review standards, exception handling, and SOP governance.

Area	What Changed in v2.0
ISTQB + Agile Alignment	STLC phases, sprint ceremonies, shift-left, and Agile test strategy integrated throughout
Service Tier Strategy	Tier 0/1/2 system — coverage targets and E2E caps vary by service criticality
Coverage Thresholds	Relaxed from rigid 95% to 90–95% for critical domains; 80–85% for standard services
Flaky Test Policy	Quarantine within 24h; fix SLA now severity-based (not flat 24h for root cause)
E2E Ratio	2% is target, not a hard cap; 5–10% allowed for frontend-heavy products
Snapshot Testing	Nuanced: still blocked for public APIs; allowed for internal with justification
Test Data Strategy	Data Builders vs Object Mothers distinction added; scenario-level fixtures for E2E reuse
Observability Testing	Full new section: assert traceId, userId, error.code in logs; metrics emission validation
Performance Enforcement	performanceBudget now enforced with concrete integration test assertion pattern
PR Review Standards	New section: how reviewers evaluate test quality — not just presence
Test Smell Detection	Named smells with detection strategy and remediation guidance
Exception Framework	Formal process for time-bound rule exceptions with tech lead approval
Mutation Testing	Stryker integration guidance for validating assertion quality
SOP Governance	Versioning, ownership, quarterly review cadence, and change log added
Golden Path Coverage	Critical user flows that MUST always pass mapped to E2E test IDs

0 · Purpose & Scope
This SOP defines how engineering teams design, write, execute, and automate tests across the full software stack. It is aligned with ISTQB Foundation Level principles and Agile testing practices from the Agile Testing Quadrants model.
It applies to all six testing disciplines — Unit, Integration, API, E2E, Performance, and Security — plus Observability and Accessibility testing. Every section has defined ownership, tooling standards, CI enforcement gates, and Agile ceremony integration points.

GOAL:  Tests are not a phase. They are a continuous, first-class engineering activity embedded at every stage of the SDLC. Every team member owns quality — not just QA.

Who This Applies To
•	Developers writing features (frontend, backend, full-stack)
•	QA engineers building and maintaining automated test suites
•	DevOps / platform engineers maintaining CI/CD pipelines
•	Tech leads and architects reviewing test strategy and coverage decisions
•	Scrum Masters and Engineering Managers embedding quality into sprint ceremonies

1 · Testing Philosophy & ISTQB Principles
This SOP is grounded in the seven ISTQB testing principles. Each principle is mapped to a concrete practice enforced in this document.

ISTQB Principle	Practical Enforcement in This SOP
1. Testing shows presence of defects, not their absence	Coverage thresholds are floors; assertion quality is equally enforced via mutation testing and PR review
2. Exhaustive testing is impossible	Risk-based approach via Service Tier classification (Section 3) determines test depth per feature
3. Early testing saves time and money (Shift Left)	Tests planned in sprint refinement; acceptance criteria linked to test cases before coding begins
4. Defects cluster (Pareto principle)	Critical paths — payments, auth, checkout — get Tier 0 treatment with deeper coverage and more E2E journeys
5. Beware of the pesticide paradox	Test suites reviewed quarterly; mutation testing (Stryker) validates that tests actually detect changes
6. Testing is context dependent	Service Tier system (Tier 0/1/2) applies different standards — no one-size-fits-all rule set
7. Absence-of-errors fallacy	100% coverage with weak assertions fails users. Test quality is audited in PR review (Section 16)

Agile Testing Quadrants (Brian Marick)
Tests in this SOP map to Marick's four quadrants. Understanding which quadrant a test belongs to guides the right tool, timing, and owner.

Quadrant	Tests	Purpose	Owner
Q1 — Technology-facing, support team	Unit, Integration	Guide development; fast feedback	Developer
Q2 — Business-facing, support team	API contract, Functional	Validate acceptance criteria	Dev + QA
Q3 — Business-facing, critique product	Exploratory, Usability, E2E	Find defects humans notice	QA + PO
Q4 — Technology-facing, critique product	Performance, Security, Load	Non-functional validation	Dev + DevOps

2 · Agile SDLC & STLC Integration
Testing in Agile is not a phase after development — it is woven into every sprint ceremony and every development activity. This section defines exactly when each test activity occurs in the sprint lifecycle.

2.1  Agile STLC — Testing Activities per Sprint Phase
Sprint Phase	Testing Activities (STLC touchpoints)
Product Backlog Refinement	QA reviews stories for testability. Acceptance criteria defined in Given–When–Then format. Test conditions identified. Non-testable stories are rejected and returned.
Sprint Planning	QA estimates test effort. Test cases linked to story acceptance criteria. Non-functional requirements (perf, security) identified for relevant stories.
Sprint Execution (Daily)	TDD/BDD red-green-refactor cycle. Unit and integration tests written alongside code. QA reviews test coverage in daily standup. CI gates block merges on failure.
Code Review / PR	Dev reviews code logic. QA (or dev) reviews test quality using Section 16 checklist. Meaningful assertions, edge cases, and boundary values verified.
Definition of Done (DoD) Check	All DoD criteria verified before sprint closes. Coverage thresholds met. Contract tests passing. Acceptance criteria validated.
Sprint Review	Exploratory testing session. QA executes session-based testing against new features in staging. Findings logged as bugs or backlog items.
Sprint Retrospective	Test health metrics reviewed: coverage delta, flaky test count, defect escape rate. Improvements to test process added to next sprint backlog.

2.2  Acceptance Criteria Standards (Given–When–Then)
All user stories must have acceptance criteria written in GWT format before development begins. These map directly to automated test cases.

# ✅ Story: User applies a discount coupon at checkout

Given: the user has items in their cart totalling £100
  And: a valid 10% discount coupon exists with code SAVE10
When:  the user applies coupon code SAVE10
Then:  the cart total is updated to £90
  And: the applied coupon is displayed with expiry date
  And: the order summary reflects the £10 discount

# Automated test maps directly:
it('reduces cart total by 10% when valid PERCENT coupon is applied', ...)

2.3  Definition of Done — Test Requirements
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
Unit tests: core logic paths covered
At least one API test (happy path)
Error handling present — no unhandled rejections
Basic request logging with traceId
Mode declaration in PR comment	All test layers: unit, integration, API
Coverage thresholds met per service tier
OpenAPI contract updated and schema-drift validated
All 6 auth states tested
Observability assertions verified (traceId, error.code in logs)
Acceptance criteria mapped to passing test cases
Security baseline passing

2.4  Shift-Left in Practice
•	Test planning starts at story refinement — not after code is written
•	Unit tests written before or alongside implementation (TDD encouraged, not mandated)
•	BDD acceptance tests co-authored by QA + developer + Product Owner
•	Static analysis and SAST run on every commit — not just before release
•	Performance budgets declared at design time — not measured for the first time in production
SHIFT-LEFT RULE:  A story with no testable acceptance criteria is not ready for sprint. QA has veto power on story readiness in refinement.

3 · Service Tier Strategy
Coverage targets, E2E ratios, and security requirements scale with service criticality. This replaces the rigid one-size-fits-all STRICT/LEAN model with a three-tier risk-based system. LEAN and STRICT still apply within each tier but with calibrated thresholds.

3.1  Tier Classification
Tier	Services	Coverage	E2E Cap	Mode
Tier 0 ★	Payments, Auth, Billing, Compliance, User Data (PII)	90–95% branch (service layer)	5–10% journey tests	STRICT
Tier 1	Core product features: checkout, onboarding, search, notifications	80–85% branch (service layer)	2–5% journey tests	STRICT
Tier 2	Internal tools, admin dashboards, low-risk CRUD, reporting	60–80% statement	0–2% journey tests	LEAN

TIER 0 RULE:  Any service touching money, authentication, PII, or a regulatory requirement is automatically Tier 0. This classification cannot be overridden without CTO approval.

3.2  Coverage Thresholds per Tier
Layer & Tier	Minimum Coverage Requirement
Tier 0 — service / business logic	90–95% branch coverage — calibrated per critical domain
Tier 0 — domain error paths	100% — every typed error must have a corresponding test
Tier 0 — auth and permission paths	100% — every auth state tested without exception
Tier 1 — service / business logic	80–85% branch coverage
Tier 1 — overall (statement)	75% minimum
Tier 2 / LEAN — overall	60% statement coverage
Pure utility functions (any tier)	100% branch coverage
Idempotency paths (Tier 0/1)	100% — single and concurrent duplicate scenarios

NUANCE:  The 95% hard gate from v1.0 caused teams to write superficial tests to hit the number. The correct approach is: high coverage AND high assertion quality. Mutation testing (Section 14) validates both.

4 · Test Types, Tools & Pyramid

4.1  The Testing Pyramid
The pyramid defines the ideal ratio of test types. The ratios below are targets, not hard caps. Frontend-heavy products may carry 5–10% at the E2E layer. The key constraint is: never substitute a higher-layer test for a lower-layer test.

E2E / Journey Tests  ·  ~2–5%  (up to 10% for UI-heavy products)
Performance & Security Tests  ·  ~5%
API / Contract Tests  ·  ~8%
Integration Tests  ·  ~20%
Unit Tests  ·  ~65%

4.2  Test Type Quick Reference
Type	Scope	Speed	Owner
Unit	Single function / class in isolation	< 5ms	Developer (mandatory)
Integration	2+ layers — service + DB, component + store	50–500ms	Developer (STRICT)
API	Full HTTP request → response with test DB	200ms–2s	Developer + QA
E2E	Full user journey across real UI + backend	5–30s	QA + Developer
Performance	Load, stress, spike against staging	Minutes	Developer + DevOps
Security	Vuln scanning, auth bypass, injection	Minutes	Security + Developer

4.3  Tool Stack
Test Type	Recommended Tools
Unit (Frontend)	Jest / Vitest · React Testing Library · jest-axe
Unit (Backend)	Jest / Vitest · ts-jest
Integration (Frontend)	Vitest + jsdom · MSW (network mocking)
Integration (Backend)	Jest + Testcontainers (Postgres, Redis) · Supertest
API / Contract	Supertest · jest-openapi · Schemathesis · Pact (consumer-driven)
E2E	Playwright (preferred) · Cypress (acceptable for legacy projects)
Performance / Load	k6 (preferred) · Artillery · Locust
Security (DAST/SAST)	OWASP ZAP · Semgrep · Burp Suite (manual, quarterly)
Accessibility	jest-axe · axe-core · Playwright axe plugin · Lighthouse CI
Visual Regression	Playwright screenshots + pixelmatch · Chromatic (Storybook)
Mutation Testing	Stryker (JS/TS) — validates assertion quality, not just coverage
Dependency Audit	npm audit · Snyk · Dependabot (automated PRs for CVE patches)
Mock / Stub	MSW (network) · nock (HTTP) · jest.fn() / vi.fn() · Sinon

5 · Unit Testing Standards
5.1  What to Unit Test
Test This	Do NOT Test This
Business logic: calculations, validations, transformations	Framework internals (Express routing, React reconciliation)
Every code path through a conditional branch	Third-party library implementations
Every domain error / exception condition	Simple pass-through getters with no logic
Edge cases: nulls, empty arrays, max/min, type coercions	Constants or pure configuration objects
Custom hooks (frontend), service methods (backend)	Auto-generated code (ORM migrations, GraphQL resolvers from schema)
Pure utility functions — 100% branch coverage required	Private methods — test through the public interface only

5.2  Arrange–Act–Assert (AAA) Structure
Every unit test must follow AAA with explicit visual separation. One behaviour per test. No exceptions.

describe('calculateDiscount', () => {
  it('should apply percentage discount to subtotal', () => {
    // ── Arrange ──────────────────────────────────────────
    const subtotal = 10000; // pence — avoids float errors
    const coupon   = { type: 'PERCENT', value: 10 };

    // ── Act ─────────────────────────────────────────────
    const result = calculateDiscount(subtotal, coupon);

    // ── Assert ──────────────────────────────────────────
    expect(result.discountAmount).toBe(1000);
    expect(result.finalTotal).toBe(9000);
  });
});

5.3  Naming Convention
Pattern	Example
should [behaviour] when [condition]	should return 0 discount when coupon is expired
[fn] [condition] → [result]	calculateDiscount with FIXED coupon → deducts flat amount from total
[component] renders [element] when [state]	UserCard renders Suspended badge when status is suspended

5.4  Unit Test Anti-Patterns (CI-Blocked)
Anti-Pattern	Correct Approach
Testing implementation details (private var values)	Test observable output — what the fn returns or throws
One giant test asserting 10+ unrelated things	Split into individual it() blocks — one behaviour per test
Mocking your own module under test	Mock only external deps. Mock your own code = test nothing.
Real timers (Date.now, setTimeout) in unit tests	jest.useFakeTimers() for all time-sensitive logic
Assertions that always pass (expect(true))	Every assertion must be falsifiable and specific
Skipped tests without ticket justification	Add ticket ref: it.skip('JIRA-1234: …') or remove the test

6 · Integration Testing Standards
6.1  Scope
What Integration Tests Cover	What They Do NOT Cover
Service + Repository + real database	HTTP layer (status codes, headers) — that is API testing
React component + real Redux store / Context	Business logic edge cases — that is unit testing
DB query correctness and index usage	Third-party SaaS behaviour (payment gateway, email)
Transaction rollback on failure	User journey flows — that is E2E testing
Event emission after persistence succeeds	Performance under load — that is performance testing
Migration: apply, rollback, and idempotency	Security probing — that is security testing

6.2  Database Isolation
•	Each suite uses an isolated DB instance — Testcontainers or Docker Compose with unique DB name per CI worker
•	Seed data in beforeEach with factory builders — never assume pre-existing rows
•	Clean up via transaction rollback (fastest) or explicit table truncation after each test
•	Run migrations fresh before each test run — never mock or stub migrations

// ✅ Testcontainers — isolated real DB per suite
let pg: StartedPostgreSqlContainer;
beforeAll(async () => {
  pg = await new PostgreSqlContainer('postgres:16').start();
  await runMigrations(pg.getConnectionUri());
});
afterAll(async  () => { await pg.stop(); });
beforeEach(async () => { await db.truncateAllTables(); });

6.3  Frontend Integration Rules
•	Use the real store / context — never mock Redux or React Context in integration tests
•	Use MSW to intercept network requests — never mock the fetch module directly
•	Assert on rendered output only — never assert on state variables or store internals
•	Use userEvent (not fireEvent) to simulate realistic user interactions

7 · API Testing Standards
7.1  Automation-First Design Requirements
Design Rule	Why It Enables Automation
operationId on every OpenAPI path	Test runners reference by operationId — stable across path renames
Machine-readable error.code in every error	Assertions use code, not message — messages can change freely
traceId in every response (success + error)	Correlates test failures to server logs during investigation
Deterministic sort order on list endpoints	Index assertions stable when seed data volume changes
Idempotency-Key on mutation endpoints	Test setup reruns safely without duplicate data creation
ISO 8601 UTC for all timestamps	Prevents locale-dependent failures across CI environments

7.2  Status Code Coverage Matrix
Status Code	Trigger in Test	Assert On
200 OK	Valid request, resource exists	Schema match (jest-openapi) + traceId present
201 Created	Valid POST creates new resource	Location header + schema + DB row exists
204 No Content	Valid DELETE	Empty body + DB row deleted
400 Bad Request	Missing required field / wrong type	error.code = VALIDATION_ERROR + field name
401 Unauthorized	Missing / invalid / expired token	error.code = MISSING_TOKEN / INVALID_TOKEN / TOKEN_EXPIRED
403 Forbidden	Valid token, wrong role or non-owner	error.code = INSUFFICIENT_PERMISSIONS
404 Not Found	Resource does not exist	error.code = [RESOURCE]_NOT_FOUND
409 Conflict	Duplicate key / idempotency collision	error.code = DUPLICATE_[RESOURCE]
422 Unprocessable	Schema valid but business rule violated	error.code matches domain error constant
429 Too Many Req.	Rate limit exceeded	Retry-After header present
500 Internal	Forced throw (test-env only)	No stack trace / internal path in response

7.3  Contract Testing
•	Commit openapi.yaml before implementation — contract is source of truth
•	jest-openapi: expect(res).toSatisfyApiSpec() on every happy-path test
•	Schemathesis in CI: auto-generates fuzz inputs from spec — free edge case coverage
•	Pact for consumer-driven contracts when multiple teams share an API
•	Schema validation failure = breaking change = pipeline blocked immediately

7.4  Snapshot Testing — Nuanced Policy
POLICY:  Snapshot tests for API response bodies are BLOCKED for public or external APIs. They are ALLOWED for internal or test-only endpoints with explicit written justification in the test file. A snapshot test must include a comment explaining why schema assertion via jest-openapi is insufficient.

// ✅ Allowed — internal admin endpoint, large stable payload
// JUSTIFICATION: This endpoint is internal-only, the payload has 40+ fields,
// and it is not part of any public OpenAPI contract.
// Reviewer approved: @techLead — 2026-03-01
expect(res.body).toMatchSnapshot();

8 · End-to-End Testing Standards
8.1  When to Write E2E Tests
RULE:  E2E tests cover user journeys — sequences of actions a real user takes. They do NOT validate business logic, API contracts, or component behaviour. Those belong in lower-layer tests.

Suitable for E2E	Use a Lower Layer Instead
User registration → email verify → login → dashboard	Single API endpoint returns 422 for invalid input
Product search → add to cart → checkout → confirmation	React component renders the correct state
Password reset: request → email link → new password	Business rule edge case (discount expiry, stock limit)
Admin creates user → user logs in → sees account	Auth middleware token validation

8.2  Golden Path Coverage
Golden paths are the critical user journeys that MUST always pass. They are the last line of defence before a release. A broken golden path blocks deployment regardless of unit/integration test status.

Golden Path ID	Journey Description
GP-01	User registration → email verification → first login
GP-02	Product browse → add to cart → guest checkout → order confirmation
GP-03	Registered user login → checkout with saved card → order confirmation
GP-04	Admin login → create product → publish → visible on storefront
GP-05	User requests password reset → receives email → sets new password → logs in

DEPLOYMENT RULE:  If any Golden Path test fails in staging, the deployment is blocked — regardless of all other tests passing. Golden paths are the non-negotiable minimum user experience guarantee.

8.3  Playwright Standards & Selector Hierarchy
•	data-testid for E2E selectors — format: kebab-case, feature-scoped: checkout-submit-button
•	Selector priority: getByRole → getByLabel → getByTestId → NEVER CSS class or XPath
•	All E2E runs against dedicated staging — never dev, never production
•	Each test is independent — creates its own user and data via API setup calls
•	Record Playwright trace on failure — attach to CI artifacts

// ✅ Playwright — GP-03: authenticated checkout
test('GP-03: registered user completes checkout', async ({ page }) => {
  const { user } = await api.createVerifiedUser();
  await api.seedProduct({ id: 'P1', price: 2999, stock: 10 });

  await page.goto('/login');
  await page.getByLabel('Email').fill(user.email);
  await page.getByLabel('Password').fill(user.password);
  await page.getByRole('button', { name: /sign in/i }).click();
  await page.getByTestId('product-card-P1').click();
  await page.getByRole('button', { name: /add to cart/i }).click();
  await page.getByTestId('checkout-submit-button').click();
  await expect(page.getByTestId('order-confirmation-banner')).toBeVisible();
});

8.4  E2E Stability Rules
•	Never use page.waitForTimeout() — use page.waitForSelector() or expect().toBeVisible()
•	Disable CSS animations and transitions in E2E via a global body class injected in test setup
•	Mock third-party providers (Stripe, SendGrid) at network level via page.route()
•	Set fixed viewport per test — prevents responsive layout inconsistencies
•	Parallelise test files — never parallelise test cases within the same file
•	CI retry: max 2 retries in CI only — retries do not substitute for fixing root cause

9 · Performance Testing Standards
9.1  Test Types & When to Run
Type	Purpose & Trigger
Smoke Test	1–5 VUs, 1 minute. Validates system works under minimal load. Run on every staging deploy.
Load Test	Ramp to production peak. Validates p95 SLAs. Run before every major release.
Stress Test	Beyond peak — find the breaking point. Run quarterly or after architecture changes.
Spike Test	Sudden 10× traffic. Validate auto-scaling and graceful degradation. Run before campaigns.
Soak / Endurance	Sustained load 1–4 hours. Detects memory leaks and connection pool exhaustion. Monthly.
Volume Test	Large data sets (1M+ rows). Validates query performance at scale. Quarterly.

9.2  SLA Targets
Endpoint Category	p95 Target
Read — simple lookup by ID	< 100ms
Read — filtered list with pagination	< 300ms
Write — simple create / update	< 500ms
Write — multi-step transaction	< 1,000ms
Authentication endpoint	< 300ms
Search / full-text	< 800ms
Report / aggregate query	< 3,000ms (with loading state in UI)
Background job completion	< 5s SLA on completion (not enqueue latency)

9.3  Per-Endpoint Performance Budget Enforcement
Every Tier 0 and Tier 1 endpoint declares a performanceBudget. Both the k6 smoke test and the integration query-count assertion read from this declaration — single source of truth.

// features/orders/performance/order.budget.ts
export const orderBudget = {
  p95ResponseMs:    300,   // k6 threshold reads this
  maxDbQueries:     3,     // Integration test asserts this
  maxExternalCalls: 1,     // MSW call counter asserts this
};

// Integration test — enforces DB query count from budget
it('resolves within query budget', async () => {
  await seedOrders(20);
  const { queryCount } = await withQueryCounter(() =>
    orderService.listByUser(userId));
  expect(queryCount).toBeLessThanOrEqual(orderBudget.maxDbQueries);
});

// k6 — reads budget from shared config
import { orderBudget } from './budgets/order.budget.js';
export const options = {
  thresholds: {
    http_req_duration: [`p(95)<${orderBudget.p95ResponseMs}`],
    http_req_failed:   ['rate<0.01'],
  },
};

9.4  Test Runtime Budgets (CI-Enforced)
Suite	Max Runtime — CI Fails if Exceeded
Unit test suite	< 30 seconds
Integration test suite	< 3 minutes
API test suite (all endpoints)	< 60 seconds
E2E (golden paths only in CI)	< 10 minutes
ENFORCEMENT:  If a test suite exceeds its runtime budget in CI, the build fails. Slow tests are a quality signal — they indicate either infrastructure problems or poorly isolated tests.

10 · Security Testing Standards
10.1  OWASP Top 10 — Minimum Test Coverage
Vulnerability	How to Test It
Injection (SQL, NoSQL, Command)	Send ', OR 1=1--, $where, ; DROP TABLE in all string params. Assert sanitised response.
Broken Authentication	Test all 6 auth states. Test token expiry, revocation, and replay attack.
Sensitive Data Exposure	Assert passwords, tokens, secrets never appear in response bodies or logs.
IDOR / Broken Access Control	Access resource IDs belonging to other users with a valid token. Assert 403/404.
Security Misconfiguration	Assert CORS, security headers (CSP, HSTS, X-Frame), and no debug endpoints in staging.
Mass Assignment	POST/PUT protected fields (role, id, balance). Assert they are ignored.
Known CVEs in Dependencies	npm audit / Snyk on every CI build. Block on high/critical CVEs.
Insufficient Logging	Assert every 4xx and 5xx emits structured log with traceId and userId.

10.2  Automated Security Gates
•	Semgrep SAST: runs on every PR — blocks on high severity findings
•	OWASP ZAP DAST baseline: runs on every staging deployment
•	TruffleHog / GitLeaks: scans every commit — blocks on any secret found
•	npm audit / Snyk: blocks on critical CVEs — 7-day SLA for high severity
•	Quarterly manual penetration test for all Tier 0 services

11 · Observability Testing (New in v2.0)
Observability signals — logs, metrics, traces — are part of the production contract. They must be tested with the same rigour as functional behaviour. You cannot debug production incidents without reliable observability.

11.1  What to Assert in Observability Tests
Signal	Mandatory Assertion
Request log (every request)	Contains: traceId, userId (if authenticated), method, endpoint, statusCode, latencyMs
Error log (every 4xx / 5xx)	Contains: traceId, errorCode, errorMessage (not stack trace), userId
Business event log	Contains: eventType, entityId, actorId, timestamp — for domain events (order.created)
Metric: latency	p95 per endpoint emitted and within budget — alerting threshold verified
Metric: error rate	4xx/5xx rate emitted per endpoint — alert fires above threshold
Distributed trace	W3C TraceContext header propagated across service boundaries

11.2  Observability Test Patterns
// ✅ Assert traceId present in error response
it('includes traceId in every error response', async () => {
  const res = await request(app)
    .get('/v1/orders/nonexistent')
    .set('Authorization', `Bearer ${token}`);
  expect(res.status).toBe(404);
  expect(res.body.error.traceId).toMatch(/^[0-9a-f-]{36}$/);
});

// ✅ Assert structured error log contains required fields
it('emits structured error log with errorCode and userId', async () => {
  const logSpy = jest.spyOn(logger, 'error');
  await request(app).get('/v1/orders/x').set('Authorization', bearer);
  const logCall = logSpy.mock.calls[0][0];
  expect(logCall).toMatchObject({
    level:      'error',
    errorCode:  'ORDER_NOT_FOUND',
    traceId:    expect.stringMatching(/^[0-9a-f-]{36}$/),
    userId:     expect.any(String),
  });
  expect(logCall).not.toHaveProperty('stackTrace'); // Never leak to logs readable by client
});

11.3  Metrics Emission Testing
•	Use a metrics registry test client (prom-client test mode) to assert counters increment correctly
•	Assert http_requests_total increments with correct endpoint, method, and status labels
•	Assert http_request_duration_seconds records values within the expected range
•	Do not test log volume or exact format — test field presence and value correctness
PRINCIPLE:  If your observability cannot be tested, it cannot be trusted. Untested observability silently breaks and leaves you blind in the next production incident.

12 · Test Data Management
12.1  Core Principles
•	Tests own their data — never assume pre-existing DB rows
•	Seed only the minimum fields relevant to the behaviour under test
•	Use factory builders with safe defaults and per-test overrides
•	Never use real PII, email addresses, or production data
•	Dates in fixtures must be deterministic — use fixed ISO strings, not new Date()

12.2  Data Builders vs Object Mothers
Two complementary patterns serve different needs. Know which to use and when.

Pattern	When to Use
Data Builder (per-test)	Fine-grained control. Each test overrides only what it cares about. Best for unit and integration tests.  const user = buildUser({ role: 'admin' }); const expiredCoupon = buildCoupon({ expiresAt: yesterday });
Object Mother (scenario)	Pre-built named scenarios shared across E2E and API suites. Hides complexity behind a meaningful name.  const { user, order, product } = Scenarios.checkoutReady(); const { admin, suspendedUser } = Scenarios.adminModeration();

// ✅ Data Builder — minimal, overridable
const buildUser = (overrides = {}) => ({
  id:        `user-${crypto.randomUUID()}`,
  email:     `test+${Date.now()}@example.com`,
  role:      'customer',
  createdAt: new Date('2024-01-15T10:00:00Z'),  // Fixed — never new Date()
  ...overrides,
});

// ✅ Object Mother — named scenario for E2E / API reuse
export const Scenarios = {
  checkoutReady: async () => {
    const user    = await db.users.create(buildUser());
    const product = await db.products.create(buildProduct({ stock: 10 }));
    const cart    = await db.carts.create(buildCart({ userId: user.id, items: [product] }));
    return { user, product, cart };
  },
};

12.3  Database Reset Strategies
Strategy	When to Use
Transaction rollback (fastest)	Wrap each test in a transaction; rollback in afterEach. Best for integration tests with heavy seed data.
Table truncation (recommended)	TRUNCATE all tables in afterEach. Simple, reliable. Use for most integration and API tests.
Schema drop & recreate (slowest)	Drop and rebuild DB per suite. Use only for migration validation tests.
Shared read-only seed (limited)	Global seed once for read-only test suites only — never for any write or mutation tests.
NEVER:  Allow multiple developers or CI workers to share a persistent test database. Concurrent writes produce non-deterministic failures that are nearly impossible to debug.

13 · Mocking & Stubbing Standards
13.1  Mock Decision Matrix
What You Are Testing	What to Mock
Business logic (unit)	Mock: repository, HTTP, email. Real: domain logic, validator, error classes.
DB query correctness (integration)	Mock: nothing — real DB container is the point.
HTTP API contract	Mock: external downstream (MSW). Real: DB, auth middleware, business logic.
User journey (E2E)	Mock: third-party providers (Stripe, SendGrid). Real: your entire app stack.
Worker handler (unit)	Mock: all dependencies. Real: the handler function logic.

GOLDEN RULE:  Mock at the boundary of your system — external DB, HTTP, queue, clock, randomness. Never mock your own service classes, domain errors, or business logic.

13.2  MSW Network Mocking
import { setupServer } from 'msw/node';
import { rest } from 'msw';

export const server = setupServer(
  rest.post('https://api.stripe.com/v1/charges', (req, res, ctx) =>
    res(ctx.json({ id: 'ch_test_123', status: 'succeeded' }))
  ),
);

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());  // ← Critical: prevent state bleed
afterAll(()  => server.close());

13.3  Time & Randomness Control
•	Always inject time as a dependency or use jest.useFakeTimers() — never call Date.now() in tested logic
•	UUIDs must be seed-controlled or injected — never allow true randomness in test environments
•	Test expiry, TTL, and scheduled logic by controlling the fake clock

// ✅ Controlling time at boundary
beforeEach(() => { jest.useFakeTimers(); jest.setSystemTime(new Date('2026-01-15')); });
afterEach(()  => { jest.useRealTimers(); });

it('rejects coupon expired at boundary', () => {
  const coupon = { expiresAt: new Date('2026-01-15T00:00:00Z') };
  expect(() => applyCoupon(coupon)).toThrow(CouponExpiredError);
});

14 · Mutation Testing
Coverage metrics tell you which lines were executed. Mutation testing tells you whether your tests would actually detect a change. A line can be covered without being meaningfully tested. Mutation testing closes this gap.

14.1  What Mutation Testing Does
•	Stryker introduces small automatic code mutations (flipped conditions, removed returns, changed values)
•	For each mutation, the test suite runs. If tests still pass — the mutation survived — tests are weak
•	Mutation score = killed mutations ÷ total mutations × 100
•	A high mutation score means your assertions are tight enough to catch real bugs

14.2  Mutation Score Targets
Layer / Tier	Target Mutation Score
Tier 0 — service / business logic	≥ 85% mutation score — run in CI on every STRICT PR
Tier 1 — service layer	≥ 75% mutation score — run weekly or before major release
Tier 2 / LEAN	Mutation testing optional — run quarterly as a health check
Pure calculation functions (any)	≥ 90% — calculations are high-risk for weak assertions

// stryker.config.json — minimal setup
{
  "testRunner": "jest",
  "mutate":     ["src/features/**/services/*.ts"],  // Service layer only
  "thresholds": { "high": 85, "low": 75, "break": 70 },
  "reporters":  ["html", "clear-text", "dashboard"]
}

INSIGHT:  If Stryker mutations survive in large numbers, the root cause is almost always: assertions test return values but not side effects, or code paths exist that no test exercises at all. Use the Stryker HTML report to target the weakest areas.

15 · Test Smell Detection & Remediation
Test smells are patterns that indicate low test quality. They do not necessarily break tests but they signal that tests are fragile, misleading, or failing to protect the codebase effectively.

15.1  Test Smell Catalogue
Smell Name	How to Detect	Remediation
Over-Mocking	A test mocks more than 3 dependencies. Mock chains. Module-level jest.mock() on own code.	Refactor: extract smaller units. Mock only the outer boundary.
Assertion Roulette	Multiple unrelated assertions in one test. Hard to know which one failed.	One test per behaviour. Split into individual it() blocks.
Eager Test	Test asserts on internal state (private vars, store internals) not observable output.	Test what the system returns, emits, or persists — not how it does it.
Mystery Guest	Test relies on global state, shared DB rows, or fixtures set up elsewhere.	Seed data in beforeEach. Make every test self-contained.
Slow Test	Unit test takes > 100ms. Integration test takes > 2s.	Introduce fake timers, mock heavy I/O, move to correct test layer.
Brittle Selector	E2E test uses CSS class, nth-child, or text content as selector.	Replace with getByRole, getByLabel, or data-testid.
Flaky Async	Test uses arbitrary sleep() or polls with timeouts instead of waiting on DOM/state.	Use waitFor, findBy*, page.waitForSelector() in test-framework idiomatic way.
Dead Test	Skipped test with no ticket reference. Disabled test no one remembers.	Add ticket reference or delete. A permanently skipped test is misleading documentation.
False Positive	Test passes even when the code it claims to test is deleted.	Check with Stryker. Add a specific, falsifiable assertion.
Iceberg Test	Test name says 'should create order' but also validates email, inventory, and payment silently.	Each test covers one behaviour. Name it precisely.

15.2  Smell Detection in PR Review
•	Reviewers use the Test Smell Catalogue above as a reference checklist in every PR
•	Any three or more smells in a single PR is grounds for a refactoring request before merge
•	ESLint plugins can detect some smells automatically: jest/no-disabled-tests, jest/valid-expect
•	Stryker surviving mutations are often a symptom of Assertion Roulette or False Positive smells

16 · PR Review Standards for Tests
Reviewing whether tests exist is not enough. Reviewers must evaluate whether tests are meaningful, sufficient, and will actually catch regressions.

16.1  PR Test Review Checklist (for Reviewers)
	Checklist Item	When
☐	Do the tests fail if the implementation is removed or reverted?	Review
☐	Are edge cases and boundary values covered — not just the happy path?	Review
☐	Is each test asserting on the behaviour described in its name?	Review
☐	Are assertions specific and falsifiable — not expect(true) or toBeDefined()?	Review
☐	Are mocks placed at system boundaries — not mocking own business logic?	Review
☐	Is the test self-contained — does not depend on execution order or shared state?	Review
☐	Does the test cover the error path and not just the success path?	Review
☐	Are acceptance criteria from the story mapped to at least one test case?	Review
☐	Do test names describe behaviour (should X when Y) — not implementation?	Review
☐	Are any test smells from Section 15 present that need remediation?	Review

16.2  Review Anti-Patterns (for Reviewers)
•	Do not approve tests simply because coverage percentage is met — coverage is a floor, not quality
•	Do not accept 'we'll add more tests later' as a reason to merge undertested code
•	Do not skip reviewing tests because the feature logic looks correct — tests validate regression safety
•	Do not approve tests that mock every dependency in the function under test — they are not testing anything real

REVIEWER AUTHORITY:  A PR reviewer can request additional tests or a refactoring of test quality before approving — even if CI gates pass. Test quality is a merge criterion, not just an automated metric.

17 · Accessibility Testing
17.1  Mandatory Accessibility Rules
•	Integrate jest-axe in every component render test — run axe() on every rendered component
•	Playwright axe plugin asserts on every page in every E2E journey
•	Lighthouse CI runs on staging deploys — fail on score regression > 5 points or any critical violation
•	WCAG 2.1 AA is the minimum compliance target — critical violations block PRs

// ✅ jest-axe — accessibility in every component test
import { axe, toHaveNoViolations } from 'jest-axe';
expect.extend(toHaveNoViolations);

it('has no accessibility violations', async () => {
  const { container } = render(<CheckoutForm />);
  expect(await axe(container)).toHaveNoViolations();
});

17.2  Semantic Selectors — Double Benefit
Semantic selectors satisfy both accessibility compliance and automation stability requirements. They are the most resilient selectors in the test suite.

Selector Priority	Example
1. getByRole (ARIA semantic)	getByRole('button', { name: /submit order/i })
2. getByLabel (form association)	getByLabelText('Email address')
3. getByTestId (fallback only)	getByTestId('order-summary-panel')
NEVER: className, querySelector	querySelector('.btn-primary') — fragile, no semantic meaning

18 · CI/CD Integration & Quality Gates
18.1  Fail-Fast Pipeline Order
Stage	Gates — Pipeline Stops if Any Gate Fails
1. Static Analysis	Lint · TypeScript type check · Secret scan · SAST (Semgrep) · Boundary violation rules
2. Unit Tests	All unit tests pass · Coverage threshold per tier · No unjustified skipped tests · Runtime < 30s
3. Contract Validation	OpenAPI schema diff · Mode declaration present · Breaking change detection
4. Integration Tests	Migrations apply cleanly · All integration tests pass · DB constraints verified · Runtime < 3m
5. API / Contract Tests	Status code matrix · jest-openapi schema assertions · Auth matrix · Runtime < 60s
6. Security Gates	npm audit no critical CVEs · Semgrep no high severity · Secret scan clean
7. Performance Smoke	(Staging) p95 within declared budget · Query count within budget · Suite runtime < 10m
8. DAST Scan	(Staging) OWASP ZAP baseline passing · No new critical/high findings
9. E2E Golden Paths	(Staging) All Golden Path tests pass · Trace artifacts generated · Deploy blocked if any GP fails

18.2  Test Reporting
•	JUnit XML output from all runners — consumed by CI dashboard
•	Coverage HTML reports published as PR artifacts
•	E2E failures attach Playwright trace + video + screenshot automatically
•	Flaky test rate tracked per suite — alert at 2%, halt feature work at 3%
•	Weekly test health report: coverage trend, flaky count, slowest tests, defect escape rate

FLAKY TEST POLICY:  Quarantine within 24 hours. Fix SLA: P1 flakiness (blocking release) within 1 day; P2 flakiness within current sprint; P3 flakiness within 2 sprints. Root cause required — retries do not substitute for a fix. Teams above 3% flaky rate halt feature work.

19 · Defect Classification & Management
19.1  Severity Matrix
Severity	Definition	Response SLA	Example
P1 — Critical	Production down or data corrupted	Hotfix within 2h	Payment broken; user data overwritten
P2 — High	Core feature broken or major security vulnerability	Fix in current sprint	Login fails 20% of users; IDOR found
P3 — Medium	Degraded but workaround exists	Fix in next sprint	Pagination wrong order; misleading error
P4 — Low	Minor UX / cosmetic issue	Fix in backlog	Placeholder typo; button misaligned on mobile

19.2  Test-First Bug Fix Process (ISTQB-Aligned)
PROCESS:  (1) Write a failing test that reproduces the defect. (2) Confirm the test fails. (3) Fix the code. (4) Confirm the test now passes. (5) Add to regression suite permanently. This guarantees the defect cannot silently regress.

19.3  Defect Escape Analysis
•	Any P1 or P2 defect escaping to production triggers a post-mortem
•	Post-mortem must answer: which test layer should have caught this? Why did it not?
•	Result of every post-mortem is a new test case added to the relevant suite before next sprint
•	Defect escape rate tracked per squad — reviewed in Sprint Retrospective

20 · Exception Handling Framework
Teams will sometimes have legitimate reasons to temporarily deviate from this SOP. This section defines the formal process for requesting, granting, and tracking exceptions — preventing silent rule violations while accommodating real-world constraints.

20.1  When Exceptions Are Valid
•	Time-critical hotfix where the full test suite cannot be completed before deployment
•	Third-party integration limitation — e.g., external provider has no test mode
•	Coverage threshold temporarily unachievable due to legacy code without full refactor
•	Exploratory new technology where test patterns are not yet established

20.2  Exception Process
1.	Engineer identifies which SOP rule they need to deviate from
2.	Engineer documents: the rule, the justification, the risk, and the proposed mitigation
3.	Tech lead reviews and approves — all exceptions require explicit tech lead sign-off
4.	Exception recorded in the PR description and in the team's exception register
5.	Time-bound waiver set — maximum 1 sprint for coverage exceptions, maximum 2 weeks for process exceptions
6.	At waiver expiry, either the exception is remediated or a new exception request is submitted

// ✅ Exception documentation in PR (mandatory format)
/**
 * EXCEPTION REQUEST
 * Rule:         SOP Section 3.2 — Tier 1 service layer 80% branch coverage
 * Justification: Legacy PaymentProcessor class has 400+ lines — full coverage
 *                requires refactor that is scoped to Q3 initiative JIRA-5501
 * Risk:          Medium — existing integration tests cover happy path
 * Mitigation:    Added 5 new unit tests for the 3 highest-risk branches
 * Waiver Expiry: 2026-05-01 (end of next sprint)
 * Approved by:   @techLead (2026-04-10)
 */

IMPORTANT:  Exceptions are not approvals to ignore quality. They are formal acknowledgements of a known gap with a time-bound remediation plan. Expired exceptions with no remediation are escalated to engineering leadership.

21 · Test Environment Management
21.1  Environment Tiers
Environment	Purpose & Rules
Local Dev	Unit + integration tests only. Docker containers for DB/cache. No shared services. Runs offline.
CI (ephemeral)	All tests to API layer. Spun up fresh per PR. Testcontainers. Destroyed after run. Immutable.
Staging	E2E, performance, DAST. Mirrors prod config. Non-prod API keys. Reset to known seed state before E2E.
Production	Zero automated tests. Passive synthetic monitoring only. No test data ever written here.

CRITICAL:  If a test accidentally targets production and writes data, treat it as an incident. Implement network-level blocks and IAM restrictions. Audit logs must be reviewed. Root cause documented.

22 · Metrics & Quality Reporting
22.1  Key Test Health Metrics
Metric	Target & Alert Threshold
Overall coverage (Tier 1 STRICT)	≥ 80% statement — alert if drops below 75%
Service layer branch coverage (Tier 0)	≥ 90–95% — hard CI gate per declared tier
Mutation score (Tier 0 service layer)	≥ 85% — run weekly; alert below 75%
Flaky test rate	< 1% of total runs — alert at 2%, halt at 3%
API test suite runtime (CI)	< 60s — fail CI if exceeded
Unit test suite runtime (CI)	< 30s — fail CI if exceeded
E2E golden path pass rate (staging)	100% required — any failure blocks deployment
P1/P2 defect escape rate	0 per sprint target — tracked in retrospective
Mean time to detect (MTTD)	< 1h for P1/P2 via monitoring or CI
Mean time to resolve (MTTR)	< 4h P1 / < 1 sprint P2
Exception register open count	≤ 2 active exceptions per squad at any time

22.2  Sprint Retrospective — Test Health Review
•	Coverage delta: did overall coverage increase, decrease, or hold this sprint?
•	Flaky tests introduced or resolved this sprint
•	New exceptions opened vs. exceptions remediated
•	Slowest tests — are any suites approaching their runtime budget?
•	Defect escapes: did any P1/P2 reach production? Post-mortem action items?
•	Mutation score trend for Tier 0 services

23 · SOP Governance
A living SOP must be maintained, versioned, and owned. Without governance, it becomes stale and teams stop following it.

23.1  Ownership & Review Cadence
Responsibility	Owner & Frequency
SOP primary owner	Staff Engineer / Head of Quality — reviews and approves all changes
Quarterly full review	SOP owner + one tech lead per squad — full read-through, relevance check
Change proposals	Any engineer may propose changes via PR against the SOP repository
Change approval	Requires: SOP owner approval + at least one senior engineer review
Emergency updates	SOP owner can publish urgent patches within 48h — flagged as hotfix

23.2  Version History
Version	Summary of Changes
v1.0 — Feb 2026	Initial release. Unit, integration, API, E2E, performance, security, accessibility standards.
v2.0 — Apr 2026	ISTQB/Agile alignment. Service tier system. Relaxed coverage thresholds. Observability testing. PR review standards. Test smell catalogue. Exception process. Mutation testing. Data builders vs Object Mothers. Snapshot policy nuanced. Flaky test SLA by severity. SOP governance.

23.3  Change Process
7.	Open a PR against the SOP repository with the proposed change
8.	Describe: what is changing, why, and what the impact on teams is
9.	Tag SOP owner and at least one tech lead from a different squad for review
10.	Discussion period: minimum 5 working days for non-urgent changes
11.	On merge: bump version, update change log, announce in engineering channel
12.	Teams given 2 sprint cycles to align their practices to any new mandatory rule

24 · Quick Decision Checklist
Run through this before opening any PR. Every item must be ticked or explicitly marked N/A with a justification.

	Checklist Item	When
☐	Unit tests written for every new function, hook, or service method	Mandatory
☐	Every conditional branch has a test — not just the happy path	Mandatory
☐	Tests follow AAA structure with clear section separation	Mandatory
☐	Test names describe behaviour: should X when Y format	Mandatory
☐	All time-dependent logic uses fake timers — no real Date.now()	Mandatory
☐	No hardcoded sleep() or setTimeout in any test file	Mandatory
☐	Each test seeds its own data in beforeEach via factory builder	Mandatory
☐	No PII, real emails, or production data in any test fixture	Mandatory
☐	jest-axe runs on every component render test	Mandatory
☐	Service tier declared (Tier 0/1/2) and coverage threshold confirmed	Mandatory
☐	GWT acceptance criteria from the story mapped to at least one test case	Mandatory
☐	Full auth matrix: missing, invalid, expired, wrong role, cross-user, correct	Strict
☐	Full status code matrix for all STRICT/Tier 0-1 endpoints	Strict
☐	jest-openapi contract assertion on every happy-path API test	Strict
☐	Integration tests assert on DB state after mutations — not just return values	Strict
☐	Observability: traceId present in error response — verified in API test	Strict
☐	performanceBudget declared and DB query count assertion passing	Strict
☐	Security baseline: IDOR, mass assignment, oversized payload tests pass	Strict
☐	Test smells check: no Over-Mocking, Brittle Selectors, or Dead Tests	Mandatory
☐	If any SOP rule is deviated from — exception request documented in PR	Mandatory
☐	E2E data created via API setup — not via UI form filling	E2E
☐	Golden Path tests pass in staging before release tag	E2E
☐	k6 smoke test passes p95 threshold on staging deploy	Perf
☐	Coverage meets threshold for declared service tier	Mandatory

25 · Do's and Don'ts
25.1  Writing Tests
✅  DO	❌  DON'T
Follow AAA in every test with explicit visual separation	Write a single test asserting multiple unrelated behaviours
Name tests: should [behaviour] when [condition]	Name tests: test1, myTest, verifyThing, or checkFlow
Seed data in beforeEach via factory builders	Rely on data left over from a prior test or a shared fixture
Link acceptance criteria (GWT) to specific test cases	Consider tests 'good enough' if CI passes
Review test quality — not just test existence — in PR reviews	Approve tests simply because coverage percentage is met

25.2  Coverage & Mutation
✅  DO	❌  DON'T
Apply tier-appropriate coverage targets (90–95% Tier 0, 80–85% Tier 1)	Apply the same 95% hard gate to all services regardless of risk
Use Stryker mutation testing to validate assertion quality	Trust line coverage as a proxy for test effectiveness
Treat coverage as a minimum floor AND audit assertion quality	Write trivial tests to hit a coverage number
Declare exceptions formally with tech lead approval when thresholds miss	Silently skip coverage rules to meet sprint deadlines

25.3  Mocking & Data
✅  DO	❌  DON'T
Mock at system boundary: DB, HTTP, queue, clock	Mock your own service methods, domain logic, or error classes
Use MSW for network-level HTTP mocking	Mock the fetch or axios module directly at the module level
Use Object Mothers for shared E2E / API scenario fixtures	Duplicate complex setup code across many test files
Use Data Builders for per-test minimal overrides	Hardcode test user IDs, emails, or dates across test files

25.4  CI & Stability
✅  DO	❌  DON'T
Run fail-fast: lint → unit → contract → integration → API → security	Run E2E before unit tests pass
Quarantine flaky tests within 24h; fix per severity SLA	Ignore flaky tests or accept them as 'sometimes they just fail'
Enforce suite runtime budgets as hard CI gates	Let test suites grow unboundedly slow
Track flaky rate, mutation score, and coverage trend in retrospective	Review test health only when a production incident occurs

Appendix · Key Terms
Term	Definition
AAA	Arrange–Act–Assert — the mandatory structure for every test case
ISTQB	International Software Testing Qualifications Board — globally recognised testing standards body
STLC	Software Testing Life Cycle — the phases of testing activity within the SDLC
GWT / BDD	Given–When–Then — Behaviour-Driven Development format for acceptance criteria
Testing Quadrants	Brian Marick's model for categorising tests by audience (business/technology) and purpose (support/critique)
Shift Left	Moving testing earlier in the SDLC — detecting defects at design and unit test time
Service Tier	Risk-based classification (Tier 0/1/2) determining coverage targets and test depth per service
Golden Path	Critical user journeys that must always pass — GP failure blocks any deployment
Object Mother	Pre-built named test scenario combining multiple related fixtures for E2E and API reuse
Data Builder	Minimal factory function creating test data objects with safe defaults and per-test overrides
Mutation Testing	Practice of introducing small code mutations to verify tests would detect real changes
Test Smell	Pattern indicating low test quality — e.g., Over-Mocking, Brittle Selectors, Dead Tests
Mutation Score	Killed mutations ÷ total mutations × 100 — measures assertion strength
Contract Test	Verifies API response matches a declared schema (OpenAPI or Pact)
MSW	Mock Service Worker — intercepts HTTP at network level for realistic test mocking
Testcontainers	Library that spins up disposable Docker containers (Postgres, Redis) for integration tests
Schemathesis	Auto-generates test inputs from OpenAPI spec to find violations and edge cases
IDOR	Insecure Direct Object Reference — accessing another user's resource via a guessable ID
DAST	Dynamic Application Security Testing — probing a running application for vulnerabilities
SAST	Static Application Security Testing — analysing source code without executing it
p95	95th percentile response time — 95% of requests complete within this duration
MTTD	Mean Time to Detect — average time from bug introduction to discovery
MTTR	Mean Time to Resolve — average time from detection to production fix
Flaky Test	A test that produces inconsistent results without any code change
DLQ	Dead Letter Queue — holds messages that failed after maximum retry attempts

— End of Document  ·  Testing SOP v2.0  ·  April 2026 —
