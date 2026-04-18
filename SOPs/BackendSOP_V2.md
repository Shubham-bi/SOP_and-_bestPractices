
STANDARD OPERATING PROCEDURE
Backend Development
API Testing, Automation & Business Logic Validation
Lean · Strict · API-First · Automation-Ready · Enforceable

Version 2.0  ·  April 2026
 
What Changed — v1.0 → v2.0
This version upgrades the SOP from a reference document into an enforceable engineering system. Every section now has a CI/PR gate, a Definition of Done, or a mandatory implementation rule.

Area	What Changed in v2.0
Definition of Done	Explicit Lean/Strict DoD added — feature not shippable without it
CI Enforcement	Mode declaration mandatory in every PR; gates block on missing tests
API Automation	Full automation-first design: test ID strategy, contract IDs, runner config
Idempotency	Formalized with Idempotency-Key header, duplicate detection, and test cases
Observability	traceId, metrics, distributed tracing added as mandatory for STRICT
Security	Replay attacks, rate-limit bypass, auth-cache invalidation added
Performance	Budget-based enforcement: p95 + max DB queries + max external calls
Async / Workers	Retry strategy, exponential backoff, DLQ rules formalized
Error Handling	Central errorMap — no inline HTTP status codes in service layer
Anti-Patterns	Testing anti-patterns defined as CI/PR blockers

0 · Definition of Done (DoD) — Non-Negotiable
A feature, endpoint, or service is not complete — and must not be merged — unless it satisfies its mode-specific DoD in full.

0.1  STRICT Mode DoD
•	All layers implemented: Unit, Integration, API / Contract tests
•	95%+ branch coverage on the service/business logic layer
•	100% domain error coverage — every typed error has a test
•	OpenAPI contract updated, merged, and schema-drift validated in CI
•	All 6 auth states tested (Section 6.1)
•	Performance budget defined and verified (p95 + DB query count)
•	Security baseline tests passing (IDOR, mass assignment, replay)
•	Idempotency implemented and tested for mutation endpoints
•	Observability verified: traceId in all responses, metrics emitting
•	Automation test IDs (x-operation-id) assigned to all endpoints

0.2  LEAN Mode DoD
•	Unit tests covering core logic paths
•	At least one API test covering the happy path
•	Error handling present — no unhandled promise rejections
•	Basic request logging with traceId
•	Mode declaration comment present in PR

ENFORCEMENT:  PRs that do not meet the DoD for their declared mode will be automatically blocked by the CI pipeline. No manual override is permitted for STRICT-mode features.

1 · Mode Declaration & CI Enforcement
Every PR must declare its mode explicitly. The CI pipeline reads this declaration and applies the correct gates automatically.

1.1  Mode Declaration Syntax
/**
 * @mode STRICT
 * @coverage 95
 * @contract required
 * @idempotency required
 * @performance-budget { p95: 300, maxDbQueries: 3 }
 */

1.2  CI Enforcement Rules by Mode
Gate	LEAN
Mode declaration comment	Required
Unit test pass	Required
API test (happy path)	Required
Integration tests	Optional
Contract / OpenAPI diff	Optional
Auth matrix (all 6 states)	Optional
Performance budget check	Optional
Security baseline scan	Optional
Coverage threshold	60% overall

Gate	STRICT
Mode declaration comment	Required — PR blocked without it
Unit test pass	Required — must pass before integration runs
API test (all status codes)	Required — full status code matrix
Integration tests	Required — DB + transaction verification
Contract / OpenAPI diff	Required — schema drift blocks merge
Auth matrix (all 6 states)	Required — missing state blocks merge
Performance budget check	Required — exceeding budget blocks merge
Security baseline scan	Required — failures block deployment
Coverage threshold	95% branch coverage (service layer)

CI FAIL-FAST ORDER:  1. Lint & type check  →  2. Unit tests  →  3. Contract diff  →  4. Integration tests  →  5. API tests  →  6. Security scan. Pipeline stops at first failure. Never run expensive tests before cheap ones pass.

2 · Mode Definitions — Lean vs Strict

Mode Decision Matrix
Criterion	→ Use STRICT Mode When...
Endpoint complexity	3+ branching conditions, multi-step orchestration, or saga patterns
Business criticality	Payments, auth, billing, compliance, data mutation, financial calculation
Consumer count	Consumed by 2+ clients (mobile, web, third-party partner)
Contract stability	External partners or public API consumers depend on the response schema
Failure blast radius	Bug causes data corruption, incorrect charges, or security breach
Regulatory scope	GDPR, PCI-DSS, HIPAA, SOC2, or any compliance framework applies
SLA target	Endpoint has a defined response-time or availability SLA
Automation requirement	Endpoint is included in a scheduled API regression suite

GOLDEN RULE:  If it touches money, auth, user data, or an external consumer contract — it is STRICT. No exceptions. No manual override.

3 · API Design & Contract Standards
3.1  Design-First Principle
The contract (OpenAPI spec) is the source of truth. It is committed before implementation begins. Code that diverges from the contract fails CI.

⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
Contract: README or PR description acceptable
Schema can be inferred from implementation
Breaking changes acceptable in early iterations	OpenAPI 3.x committed and reviewed before first line of code
CI schema-drift check runs on every build
Breaking changes require versioning + migration plan + consumer sign-off

3.2  Automation-Ready Endpoint Design
Every endpoint must be designed so that automated tests can target it without coupling to implementation details. The following are mandatory for automation stability.

Design Rule	Why It Matters for Automation
Assign x-operation-id to every OpenAPI path	Test runners use operationId to reference endpoints — avoids path-string coupling
Machine-readable error codes in every error response	Test assertions use error.code, not error.message (messages can change)
traceId in every response (success + error)	Correlates test run logs to server logs during failure investigation
Deterministic response ordering on list endpoints	Tests that assert on array index position are stable across runs
Pagination contract: consistent cursor or page/limit	Avoids test brittleness when seed data volume changes
Idempotency-Key supported on all mutation endpoints	Allows test setup to re-run safely without creating duplicate data
All timestamp fields in ISO 8601 / UTC	Prevents locale-dependent test failures in CI environments
No randomised field names or dynamic keys in response	Schema validators and contract tests cannot handle unpredictable shapes

3.3  Consistent Response Envelope (Mandatory)
// ✅ Standard success envelope
{
  "data": { ... },
  "meta": { "total": 150, "page": 1, "limit": 20, "traceId": "abc-xyz" }
}

// ✅ Standard error envelope
{
  "error": {
    "code":    "INSUFFICIENT_INVENTORY",  // machine-readable, stable, SCREAMING_SNAKE_CASE
    "message": "Product P1 has 2 units; 5 requested.",  // human-readable
    "field":   "quantity",                // optional: for field-level validation errors
    "traceId": "abc-xyz-123"              // always present — ties to server logs
  }
}

3.4  API Versioning
•	URL versioning for breaking changes: /v1/orders → /v2/orders
•	Never remove or rename a field in a minor version — add alongside the old field
•	Mark deprecated fields in OpenAPI with deprecated: true and x-sunset-date
•	Support minimum N-1 versions in production until all consumers complete migration
•	Publish a public changelog entry for every breaking change

3.5  Idempotency (STRICT — Mandatory for Mutations)
Idempotency ensures that retrying a failed request does not create duplicate side effects. It is a prerequisite for reliable API automation because test setup often re-runs.

•	Require Idempotency-Key header on: payments, order creation, any state-changing POST
•	Store the key + response hash; return the cached response for duplicate keys
•	Idempotency window: minimum 24 hours
•	Return 409 Conflict if a duplicate key is received with a different request body

// ✅ Idempotency tests — mandatory for STRICT mutation endpoints
it('returns same response for duplicate Idempotency-Key', async () => {
  const key = 'idem-key-' + Date.now();
  const r1 = await request(app).post('/v1/orders').set('Idempotency-Key', key).send(payload);
  const r2 = await request(app).post('/v1/orders').set('Idempotency-Key', key).send(payload);
  expect(r1.body.data.id).toBe(r2.body.data.id);  // Same resource
});

it('only creates one record in DB for concurrent duplicate keys', async () => {
  const key = 'concurrent-idem-key';
  const [r1, r2] = await Promise.all([
    request(app).post('/v1/orders').set('Idempotency-Key', key).send(payload),
    request(app).post('/v1/orders').set('Idempotency-Key', key).send(payload),
  ]);
  const orders = await db.orders.count({ idempotencyKey: key });
  expect(orders).toBe(1);
});

4 · API Testing Standards
4.1  Test Pyramid (Enforced Ratios)
Layer	Scope, Tools & Coverage Target
Unit  (70%)	Pure functions, service logic, validators, transformers. No I/O. Jest/Vitest. Millisecond execution.
Integration  (20%)	Service + repository + real DB container. Validates composition. Testcontainers + real migrations.
API / Contract  (8%)	Full HTTP round-trip. Status codes, schema, headers, error codes. Supertest + JSON Schema. Real test DB.
E2E / Journey  (2%)	Multi-endpoint user flows. Staging environment only. Playwright or Newman on OpenAPI spec.

4.2  API Automation — Test Runner Standards
Automated API test suites must be runnable without any human interaction, from CI or locally with a single command.

•	All API tests must be runnable via: npm run test:api — no manual server startup required
•	Tests must start an in-process server instance — never depend on a separately running server
•	Each test file must be independently runnable — no shared global state between files
•	Tests must accept environment variable overrides for base URL, token, and DB connection
•	Test run must produce JUnit XML output for CI dashboard ingestion
•	Total API test suite runtime must not exceed 60 seconds in CI

// ✅ Self-contained API test setup (no external server needed)
import { createApp } from '../app';
import request from 'supertest';

let app, server;
beforeAll(async () => { app = await createApp(); server = app.listen(0); });
afterAll(async  () => { await server.close(); await db.destroy(); });
beforeEach(async () => { await db.truncateAll(); await seedFixtures(); });

4.3  Status Code Coverage Matrix (Mandatory per Endpoint)
Every STRICT-mode endpoint must have a test for each applicable status code. Track coverage using the matrix below.

Status Code	Trigger Condition to Test	Assertion Required
200 OK	Valid request, resource exists	Schema match + traceId present
201 Created	Valid POST creates new resource	Location header + schema + DB row exists
204 No Content	Valid DELETE	Empty body + DB row deleted
400 Bad Request	Missing required field / wrong type	error.code = VALIDATION_ERROR + field name
401 Unauthorized	Missing / malformed / expired token	error.code = MISSING_TOKEN / INVALID_TOKEN / TOKEN_EXPIRED
403 Forbidden	Valid token, wrong role or non-owner	error.code = INSUFFICIENT_PERMISSIONS
404 Not Found	Resource ID does not exist	error.code = [RESOURCE]_NOT_FOUND
409 Conflict	Duplicate key / stale version / constraint	error.code = DUPLICATE_[RESOURCE] or CONFLICT
422 Unprocessable	Schema valid but business rule violated	error.code matches domain error constant
429 Too Many Req.	Rate limit exceeded	Retry-After header present
500 Internal	Force throw in handler (test only)	No stack trace, no internal path in response

4.4  Contract Testing with OpenAPI
•	Use dredd, schemathesis, or jest-openapi to validate responses against the committed OpenAPI spec
•	Run contract tests on every CI build — not just before release
•	schemathesis can auto-generate edge case inputs from the OpenAPI schema — use it for fuzz coverage
•	Contract tests must cover: response schema shape, required fields, field types, and enum values
•	Any response that fails schema validation is a breaking change — it blocks the pipeline

// ✅ Jest + jest-openapi contract assertion
import { setupFilesAfterEach } from 'jest-openapi';
setupFilesAfterEach('./openapi.yaml');

it('GET /v1/orders/:id response matches OpenAPI contract', async () => {
  const res = await request(app).get('/v1/orders/123').set('Authorization', bearer);
  expect(res).toSatisfyApiSpec();  // Validates against OpenAPI schema
});

4.5  Test Naming Convention
Pattern	Example
[METHOD] [endpoint] [condition] → [outcome]	POST /v1/orders with expired token → returns 401 TOKEN_EXPIRED
should [behaviour] when [condition]	should throw InsufficientInventoryError when stock < requested qty
[layer] [method] [scenario]	orderService.create with duplicate idempotency key → returns cached response

4.6  Anti-Patterns (PR Blockers)
BLOCKED:  The following testing practices are CI/PR blockers. Any PR containing these patterns will not merge.

Anti-Pattern	Why It Is Blocked & What to Do Instead
API tests replacing unit tests for business logic	Business rules must be unit-tested at the service layer. API tests validate the HTTP contract only.
E2E tests used for business rule validation	E2E is for user journey smoke testing. Business rules belong in unit/integration tests.
Snapshot testing API response bodies	Snapshots break on any field addition. Use explicit schema assertions (jest-openapi).
No negative test cases on any endpoint	Every endpoint must test at least one error path. Happy-path-only suites are incomplete.
Hardcoded sleep() / setTimeout in tests	Use waitFor, polling assertions, or fake timers. Sleep creates flaky tests.
Tests sharing database state across test files	Each test file must seed its own data in beforeEach. Shared state causes order-dependent failures.
Asserting on error.message strings	Messages are human-readable and can change. Always assert on error.code (machine-readable constant).
Direct DB queries inside API test assertions	Acceptable for mutation verification only. Never use raw DB reads to substitute for HTTP response assertions.

5 · Business Logic Layer Design
5.1  Layer Separation — Strictly Enforced
Layer	Responsibility  (What Belongs Here)
Route / Controller	Parse request → call service → return response. Max 15 lines. Zero business logic.
Service / Use Case	All business rules, validations, decisions, orchestration. No HTTP context allowed.
Repository / DAO	Database access only. No branching logic based on domain rules.
Domain Model / Entity	Data shape + invariant enforcement (e.g. qty cannot be negative).
Validator	Input schema validation (Zod / Joi). Not business rule validation.

CI RULE:  Static analysis (eslint-plugin-boundaries or custom AST rule) must detect and block any import of HTTP context (req, res, HttpException) inside the service layer.

// ❌ Blocked — business logic in controller
router.post('/orders', async (req, res) => {
  if (req.body.items.length === 0) return res.status(400).json({ error: { code: 'EMPTY_CART' }});
  if (req.user.creditLimit < total) return res.status(402).json({ ... });
});

// ✅ Correct — controller delegates entirely to service
router.post('/orders', async (req, res) => {
  const result = await orderService.createOrder(req.body, req.user);
  res.status(201).json({ data: result });
});

5.2  Central Error Map (Mandatory for STRICT)
All domain errors are registered in a single errorMap. Controllers must look up HTTP status codes from this map — never hardcode them.

// errors/errorMap.ts — single source of truth
export const errorMap: Record<string, { status: number; retryable: boolean }> = {
  ORDER_NOT_FOUND:          { status: 404, retryable: false },
  INSUFFICIENT_INVENTORY:   { status: 422, retryable: false },
  DUPLICATE_ORDER:          { status: 409, retryable: false },
  PAYMENT_GATEWAY_TIMEOUT:  { status: 503, retryable: true  },
  TOKEN_EXPIRED:            { status: 401, retryable: false },
};

// middleware/errorHandler.ts — maps domain error → HTTP response
app.use((err, req, res, next) => {
  const mapped = errorMap[err.code] ?? { status: 500, retryable: false };
  res.status(mapped.status).json({
    error: { code: err.code, message: err.message, traceId: req.traceId }
  });
});

5.3  Boundary Value Test Matrix
Business Rule	Boundary Cases to Test	Expected Outcome
Min order value	total=0, total=0.01, total=min-0.01, total=min	Error below min; pass at min
Discount expiry	now=expiry-1s, now=expiry, now=expiry+1s	Pass before expiry; reject at and after
Max items per order	qty=0, qty=1, qty=max, qty=max+1	Error at 0 and above max; pass at max
Duplicate prevention	Same Idempotency-Key × 1, × 2, × 3 concurrent calls	First succeeds; subsequent return 409
Permission scope	owner / admin / other-user / unauthenticated	403 or 401 for non-owner non-admin
Credit limit check	balance=limit-0.01, balance=limit, balance=limit+0.01	Pass at and below limit; reject above

5.4  Side Effect & Idempotency Rules
•	Side effects (emails, webhooks, analytics events) must be published after persistence succeeds — never before
•	Side effect emitters must be injectable interfaces — never imported directly into service methods
•	Test that side effects are NOT emitted when persistence fails (rollback scenario)
•	Worker handlers that produce side effects must be idempotent — processing the same message twice produces no additional effect

6 · Authentication & Authorization Testing
6.1  Mandatory Auth Test Matrix
Every STRICT-mode protected endpoint must cover all six states. Missing any state blocks the PR.

Auth State	Expected HTTP Response
No token (unauthenticated)	401 — error.code: MISSING_TOKEN — no data leaked in response
Invalid / malformed token	401 — error.code: INVALID_TOKEN
Expired token	401 — error.code: TOKEN_EXPIRED
Valid token, insufficient role	403 — error.code: INSUFFICIENT_PERMISSIONS
Valid token, accessing another user's resource	403 (or 404 to prevent enumeration)
Valid token, correct role, correct owner	Expected success response matching schema

6.2  Advanced Security Test Cases (New in v2.0)
•	Replay attack: capture a valid request + token and replay it 60 seconds later — assert 401 or idempotent result
•	Token reuse after logout: invalidate token server-side, then replay — assert 401 TOKEN_REVOKED
•	Role change propagation: change user role in DB, then use existing valid JWT — assert permissions reflect the new role (or cached stale role is documented)
•	Rate limit bypass: send requests from different IPs / with spoofed headers — assert limits still apply
•	Auth cache invalidation: if auth decisions are cached, verify the cache is invalidated on role/permission change

// ✅ Replay attack test
it('rejects replayed token after logout', async () => {
  const { token } = await loginUser(testUser);
  await request(app).post('/v1/auth/logout').set('Authorization', `Bearer ${token}`);

  const res = await request(app).get('/v1/orders').set('Authorization', `Bearer ${token}`);
  expect(res.status).toBe(401);
  expect(res.body.error.code).toBe('TOKEN_REVOKED');
});

7 · Error Handling & Observability
7.1  Centralised Error Handler
•	Single error handler middleware — never write res.status(500).json() inline
•	All domain errors route through the central errorMap — no inline HTTP status codes in services
•	Unhandled errors default to 500 with a safe message — no stack traces in response body
•	Log full error + stack + context server-side; surface only code, message, traceId to client

7.2  Observability Standards (New in v2.0 — Mandatory for STRICT)
Every request must be fully observable in production. The following are required before a STRICT endpoint ships.

Signal	Mandatory Fields & Behaviour
Request Log (INFO)	traceId, userId, method, endpoint, statusCode, latencyMs — on every request
Error Log (ERROR)	traceId, errorCode, errorMessage, stackTrace, userId — on every 4xx/5xx
Business Event Log	eventType, entityId, actorId, timestamp — for domain events (order.created, payment.failed)
Latency Metric	p50, p95, p99 per endpoint — alert threshold on p95 exceeding performance budget
Error Rate Metric	4xx rate and 5xx rate per endpoint — alert on 5xx > 0.1% or 4xx spike
Dependency Latency	External DB + HTTP call latency tracked separately — identify bottleneck layer
Distributed Trace	Trace context propagated across service boundaries via W3C TraceContext header

7.3  Testing Observability
•	Unit test that every domain error emits a log entry with the correct errorCode field
•	Integration test that traceId is generated and present in both the response and the log entry
•	API test that every error response includes a traceId that is a valid UUID or ULID format
•	Do not test log volume or log formatting — test presence and correctness of structured fields

// ✅ Test: traceId always present in error response
it('includes traceId in error response body', async () => {
  const res = await request(app).get('/v1/orders/nonexistent').set('Authorization', bearer);
  expect(res.status).toBe(404);
  expect(res.body.error.traceId).toMatch(/^[0-9a-f-]{36}$/);  // UUID format
});

8 · Database & Repository Layer
8.1  Repository Pattern Rules
•	All database access behind a repository interface — never call ORM directly from services
•	Repository methods named after domain operations: findActiveOrdersByUser — not selectAll
•	Repositories return domain objects — not raw ORM entity instances
•	Transactions managed at the service layer — not inside repository methods
•	Complex queries encapsulated in named methods with individual integration tests

8.2  Query Performance Budget (New in v2.0)
Every STRICT endpoint must declare and enforce a query performance budget.

// Declare at service or route level
export const performanceBudget = {
  p95ResponseMs:     300,   // Max allowed p95 response time
  maxDbQueries:      3,     // Max DB round-trips per request
  maxExternalCalls:  2,     // Max outbound HTTP calls per request
};

// ✅ Integration test — enforce DB query count
it('resolves order list within query budget', async () => {
  await seedOrders(20);
  const { queryCount } = await withQueryCounter(() =>
    orderService.listByUser(userId));
  expect(queryCount).toBeLessThanOrEqual(performanceBudget.maxDbQueries);
});

8.3  Migration Testing (Enforced in CI)
•	Migrations run against a clean schema before integration tests in every CI build
•	Test that migrations are idempotent — running twice must not alter schema
•	Include rollback test: apply → rollback → verify clean schema
•	Never skip or stub migrations in test environments — tests run against production-identical schema

CRITICAL:  Schema drift between test and production databases is the primary cause of 'tests pass / prod breaks' incidents. Migration CI gate is non-negotiable.

9 · Async Workers & Event-Driven Services
9.1  Testable Worker Architecture
•	Extract worker logic into a pure handler function — no queue SDK imports in the handler
•	Queue consumer is a thin wrapper that calls the pure handler
•	All external dependencies injected — repo, emailService, etc.
•	Pure handler is unit-testable without queue infrastructure

// ✅ Pure, injectable, unit-testable worker handler
export const handleOrderConfirmation = async (payload, deps = defaultDeps) => {
  const order = await deps.orderRepo.findById(payload.orderId);
  await deps.emailService.send({ to: order.email, template: 'order-confirmed' });
  return { status: 'sent', orderId: order.id };
};

9.2  Retry Strategy (New in v2.0)
Rule	Implementation Requirement
Max retry count	Defined per job type. Default: 3. Payment jobs: 5.
Backoff strategy	Exponential with jitter: baseDelay * 2^attempt + random(0, 1000ms)
Non-retryable errors	Domain validation errors (4xx) → send to DLQ immediately, no retry
Retryable errors	Transient failures (5xx, timeouts) → retry with backoff
DLQ monitoring	Alert if DLQ depth > 0 for more than 10 minutes
Idempotent handlers	Mandatory — processing same message twice must not create duplicate effects

9.3  Worker Testing Requirements
•	Unit test the pure handler — no queue infrastructure needed
•	Test idempotency: call handler with same payload twice — assert single side effect in DB
•	Test DLQ routing: throw a non-retryable domain error — assert message lands in DLQ
•	Integration test full consumer → handler → DB chain with a real queue container
•	Assert on final DB state — not on mock call counts alone

10 · Mocking Strategy
10.1  What to Mock and When
Dependency	Strategy
Repository / DB — unit tests	jest.fn() / vi.fn() returning typed fixture objects
External HTTP APIs	nock or msw/node — intercept at HTTP level, not module level
Message queue / event bus	In-process mock broker or real container (testcontainers)
Email / SMS / push providers	Injectable interface mock in unit; capture-mode in integration
Date.now() / new Date()	jest.useFakeTimers() — mandatory for any time-sensitive logic
Math.random() / UUID generation	Seed-controlled generator — inject as dependency, never global
Feature flags	Injected flag client with test overrides — never read env vars inline

10.2  The Mock Boundary — Golden Rule
RULE:  Mock at the boundary of your system — external services, DB, queue, clock. Never mock your own service methods, domain logic, or error classes. If you mock your own code, you are testing nothing real.

// ❌ Wrong — mocking your own service defeats the test
jest.mock('./inventoryService');
inventoryService.check.mockResolvedValue(true);

// ✅ Correct — mock only the external boundary (repository)
inventoryRepo.findByProductId.mockResolvedValue({ stock: 5, productId: 'P1' });
// Your actual service logic now runs against realistic mock data

11 · Performance & Load Testing
11.1  Response Time Baselines
Endpoint Category	p95 Target
Read — simple lookup (by ID)	< 100ms
Read — filtered list with pagination	< 300ms
Write — simple create / update	< 500ms
Write — multi-step transaction	< 1,000ms
Background job (enqueue only)	< 200ms (enqueue); < 5s (job completion SLA)
Report / aggregate query	< 3,000ms (with explicit loading state in client)

11.2  Automated Performance Enforcement (New in v2.0)
•	Every STRICT endpoint must declare a performanceBudget object (Section 8.2)
•	CI runs a lightweight load test (k6 or Artillery smoke) on every staging deployment
•	If p95 exceeds the declared budget, the deployment pipeline is blocked
•	Performance regressions are treated as bugs — require the same fix-forward process as functional regressions

11.3  Query Performance Rules
•	Every production query must use an index — verified with EXPLAIN ANALYZE in the migration
•	N+1 patterns are bugs — caught with query count assertions in integration tests
•	All list endpoints paginated — no unbounded queries
•	Joins must be explicitly tested for correctness under non-trivial data volumes (1,000+ rows in test)

12 · Security Testing Baseline
12.1  Mandatory Security Tests
Attack Vector	Test Requirement
SQL / NoSQL injection	Send ' OR 1=1--, $where, and operator payloads in all string parameters
Mass assignment	Attempt to POST/PUT protected fields: role, id, createdAt, balance
IDOR	Attempt to access resource IDs belonging to other users with a valid token
Oversized payload	Send request body exceeding configured size limit — assert 413
Replay attack	Capture and replay a valid request — assert idempotent or rejected
Rate limit bypass	Send requests with spoofed X-Forwarded-For headers — assert limits still apply
Auth cache staleness	Change user role in DB, assert cached auth decision is invalidated within TTL
Sensitive data leakage	Assert passwords, tokens, secrets never appear in any response body or log

12.2  Automated Security Scanning (STRICT — CI Gate)
•	OWASP ZAP or Semgrep runs on every merge to main
•	TruffleHog / GitLeaks scans every commit for accidentally committed secrets
•	npm audit / Snyk checks dependency vulnerabilities — high/critical blocks deployment
•	Scan results published to security dashboard with 7-day remediation SLA for critical findings

13 · CI/CD Pipeline & Enforcement
13.1  Fail-Fast Pipeline Order
Cheaper, faster checks run first. The pipeline stops at the first failure — never waste time running integration tests if linting fails.

Stage	Gates — Pipeline Stops if Any Fail
1. Static Analysis	Lint · Type check · Boundary violation detection (no HTTP in service layer) · Secret scan
2. Unit Tests	All unit tests pass · Coverage threshold met per file · No skipped tests without justification
3. Contract Diff	OpenAPI schema diff check · Breaking change detection · Mode declaration present
4. Integration Tests	Migration applies cleanly · All integration tests pass · DB constraint violations caught
5. API Tests	Full status code matrix passes · Contract assertions pass (jest-openapi) · Auth matrix complete
6. Security Scan	OWASP ZAP baseline · Dependency audit · No new critical/high CVEs
7. Performance Smoke	(Staging only) p95 within declared budget · Query count within budget
8. E2E Journey Tests	(Staging only) Critical user journeys complete successfully

13.2  Coverage Thresholds (Enforced)
Layer / Mode	Minimum Coverage
LEAN — overall	60% statement coverage
STRICT — overall	80% statement coverage
STRICT — service / business logic	95% branch coverage — mandatory
STRICT — domain error paths	100% — every domain error must have a corresponding test
Auth & permission paths	100% — every auth state must be tested
Idempotency paths	100% — both single and concurrent duplicate key scenarios

13.3  PR Gate Additions (v2.0)
•	PR blocked if mode declaration comment is absent
•	PR blocked if coverage drops more than 2% from base branch
•	PR blocked if OpenAPI spec is not updated for any response shape change (STRICT)
•	PR blocked if any domain error class has no corresponding test
•	PR blocked if performance budget is not declared for new STRICT endpoints

FLAKY TEST POLICY:  Any test that fails without a code change is a defect. Quarantine within 24h. Root cause and fix within one sprint. Teams with > 3% flaky test rate must halt feature work and fix the test suite before resuming.

14 · Test Data Management
14.1  Principles
•	Tests create their own data in beforeEach — never assume pre-existing rows
•	Seed only the minimum fields relevant to the test scenario
•	Use factory functions with sensible defaults and per-test overrides
•	Never use real PII, email addresses, or phone numbers in test fixtures
•	Never use production data in automated tests

14.2  Factory Builder Pattern
// ✅ Minimal factory with safe defaults and deterministic values
const buildUser = (overrides = {}) => ({
  id:        'user-' + Math.random().toString(36).slice(2, 10),
  email:     `test-${Date.now()}@example.com`,  // example.com — safe for tests
  role:      'customer',
  createdAt: new Date('2024-01-15T10:00:00Z'),  // Deterministic — not new Date()
  ...overrides,
});

// Usage: override only what the test cares about
const adminUser  = buildUser({ role: 'admin' });
const frozenUser = buildUser({ role: 'customer', status: 'suspended' });

14.3  Database Reset Strategy
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
Global truncate + reseed before test suite run
Acceptable for read-only suites
Single shared schema reset between runs	Per-test truncate + factory seed in beforeEach
Required for all write/mutation tests
Transaction rollback preferred: fastest reset strategy
Alternatively: explicit table truncation list in afterEach

15 · End-to-End Example — STRICT Endpoint: POST /v1/orders
This section walks through the complete lifecycle of a STRICT-mode endpoint from contract definition to passing CI, demonstrating every section of this SOP in action.

Step 1 — OpenAPI Contract (Before Code)
# openapi.yaml — committed first
paths:
  /v1/orders:
    post:
      operationId: createOrder   # Automation test ID
      summary: Create a new order
      security: [{ bearerAuth: [] }]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              required: [items, idempotencyKey]
              properties:
                items:          { type: array, minItems: 1 }
                idempotencyKey: { type: string }
      responses:
        '201': { description: Order created, content: { ... } }
        '400': { description: Validation error }
        '409': { description: Duplicate idempotency key }
        '422': { description: Business rule violation }

Step 2 — Domain Error Definition
// errors/order.errors.ts
export class InsufficientInventoryError extends DomainError {
  constructor(productId: string, requested: number, available: number) {
    super('INSUFFICIENT_INVENTORY',
      `Product ${productId} has ${available} units; ${requested} requested.`);
    this.context = { productId, requested, available };
  }
}

Step 3 — Service Layer (All Business Rules)
// services/orderService.ts
async createOrder(dto: CreateOrderDto, user: User): Promise<Order> {
  if (dto.items.length === 0) throw new EmptyCartError();

  for (const item of dto.items) {
    const inv = await this.inventoryRepo.findByProductId(item.productId);
    if (inv.stock < item.qty) throw new InsufficientInventoryError(item.productId, item.qty, inv.stock);
  }

  const order = await this.orderRepo.create({ ...dto, userId: user.id });
  await this.eventBus.publish('order.created', { orderId: order.id });
  return order;
}

Step 4 — Unit Tests (Service Layer)
describe('orderService.createOrder', () => {
  it('throws EmptyCartError when items array is empty', async () => {
    await expect(service.createOrder({ items: [] }, user))
      .rejects.toThrow(EmptyCartError);
  });

  it('throws InsufficientInventoryError when stock < requested', async () => {
    inventoryRepo.findByProductId.mockResolvedValue({ stock: 1 });
    await expect(service.createOrder({ items: [{ productId: 'P1', qty: 5 }] }, user))
      .rejects.toThrow(InsufficientInventoryError);
  });
});

Step 5 — API Test (HTTP Contract)
describe('POST /v1/orders', () => {
  it('returns 201 with Location header on success', async () => {
    const res = await request(app).post('/v1/orders')
      .set('Authorization', bearer).set('Idempotency-Key', 'idem-1').send(validPayload);
    expect(res.status).toBe(201);
    expect(res.headers['location']).toMatch(/\/v1\/orders\//);
    expect(res).toSatisfyApiSpec();  // Contract assertion
  });

  it('returns 422 INSUFFICIENT_INVENTORY when stock too low', async () => {
    await seedInventory({ productId: 'P1', stock: 1 });
    const res = await request(app).post('/v1/orders')
      .set('Authorization', bearer).send({ items: [{ productId: 'P1', qty: 5 }] });
    expect(res.status).toBe(422);
    expect(res.body.error.code).toBe('INSUFFICIENT_INVENTORY');
    expect(res.body.error.traceId).toBeDefined();
  });
});

16 · Folder Structure
16.1  Lean
src/
  routes/           # Route definitions
  services/         # Business logic
  repositories/     # DB access
  validators/       # Zod / Joi schemas
  errors/           # Domain error classes
  utils/
  __tests__/        # All tests co-located
  app.ts

16.2  Strict (Feature-First)
src/
  features/
    orders/
      routes/
        orders.routes.ts              # Route + @mode declaration
        orders.routes.test.ts         # API / contract tests
      services/
        orderService.ts               # ALL business logic
        orderService.test.ts          # Unit tests (service)
      repositories/
        orderRepository.ts
        orderRepository.test.ts       # Integration tests
      validators/
        createOrder.schema.ts
        createOrder.schema.test.ts
      errors/
        order.errors.ts
      types/
        order.types.ts
      performance/
        order.budget.ts               # performanceBudget declaration
      index.ts                        # Feature public API
  shared/
    middleware/                       # Auth, logging, error handler
    errors/                           # Base DomainError + errorMap
    database/                         # Connection, migrations
    types/
  test/
    factories/                        # buildUser(), buildOrder() etc.
    helpers/                          # seedDB, createTestToken, withQueryCounter
    msw-handlers/                     # MSW handlers for outbound HTTP
  openapi.yaml                        # Contract — committed before code
  app.ts
  server.ts

17 · Documentation Standards
Item	Required Documentation
Simple CRUD endpoint	OpenAPI docstring: parameters, responses, all error codes
Complex service method (>2 conditions)	JSDoc: what domain rule this enforces, not how the code works
Domain error class	When thrown, what it means, how the consumer should respond
Performance trade-off decision	Comment explaining why denormalisation or caching was chosen
Auth / permission logic	Explicit list of who can and cannot call this — never implicit
Workaround or non-obvious code	MANDATORY: // TODO: [JIRA-XXX] + one-line reason
Retry / backoff configuration	Document max retries, backoff formula, and DLQ behaviour

ANTI-PATTERN:  Never document what the code does — document why it does it. Outdated comments are worse than no comments. Delete stale comments on sight.

18 · Quick Decision Checklist
Run this before opening a PR. Every item must be ticked or explicitly justified as N/A.

	Checklist Item	Mode
☐	Mode declaration (@mode LEAN / STRICT) present in PR comment block	Both
☐	All business logic in the service layer — none in routes or repositories	Both
☐	OpenAPI spec updated to reflect any response shape or status code change	Strict
☐	Every mutation endpoint implements and tests idempotency (Idempotency-Key)	Strict
☐	All 6 auth states tested: missing, invalid, expired, wrong role, cross-user, correct	Strict
☐	Status code coverage matrix complete (200/201/400/401/403/404/409/422/500)	Strict
☐	Every domain error class has at least one test asserting it is thrown correctly	Both
☐	Error assertions use error.code, not error.message strings	Both
☐	traceId present in all error responses — verified in at least one API test	Both
☐	No hardcoded sleep() / setTimeout in test files	Both
☐	Each test seeds its own data in beforeEach — no shared DB state between tests	Both
☐	Performance budget declared; DB query count test passes	Strict
☐	Migration: apply, rollback, and idempotency tested	Both
☐	Side effects (email, events) are injectable and tested for rollback scenarios	Strict
☐	Security baseline: IDOR, mass assignment, oversized payload tests passing	Strict
☐	Coverage meets threshold: 60% (LEAN) or 95% branch on service (STRICT)	Both

19 · Do's and Don'ts
19.1  API Design
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
✅  DO
Use x-operation-id in OpenAPI for stable test targeting
Return consistent error envelope with typed code + traceId
Version breaking changes before shipping
Design the OpenAPI spec before writing any implementation	❌  DON'T
Return 200 OK with errors buried in the body
Assert on error.message — use error.code
Change response schema without deprecation + versioning
Let operationId be auto-generated from method + path

19.2  Business Logic
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
✅  DO
Put all business rules in the service layer exclusively
Throw typed domain errors — register them in errorMap
Keep calculation functions stateless and 100% unit tested
Map API responses to domain objects at the boundary	❌  DON'T
Write if/else business logic in route handlers
Throw generic new Error('something went wrong')
Hardcode HTTP status codes inside service methods
Let ORM entity objects leak into service return types

19.3  Testing
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
✅  DO
Follow Arrange-Act-Assert in every single test
Seed data in beforeEach with factory builders
Assert on DB state after mutations, not just return values
Use jest-openapi or equivalent for contract assertions	❌  DON'T
Snapshot test API response bodies
Share DB state between test files
Mock your own service methods in another service's test
Use E2E tests to validate business logic branches

19.4  Observability & Resilience
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
✅  DO
Include traceId in every request log and response
Declare and test performanceBudget on STRICT endpoints
Make worker handlers idempotent by design
Propagate W3C TraceContext headers across service calls	❌  DON'T
Return stack traces or internal paths in error responses
Emit side effects before persistence succeeds
Retry domain validation errors (4xx) — send to DLQ
Test log volume — test log field presence and correctness

Appendix · Key Terms
Term	Definition
LEAN Mode	Approach for low-risk, low-complexity backend code with minimal structural overhead
STRICT Mode	Fully validated, observable, contract-tested approach for critical or consumer-facing services
operationId	OpenAPI field used as a stable identifier for automation test runners to reference endpoints
Domain Error	Typed, named error class representing a business rule violation — independent of HTTP
errorMap	Central registry mapping domain error codes to HTTP status codes and retry behaviour
Idempotency	Property where running an operation multiple times produces the same result as once
Contract Test	Test that verifies API response shape matches the committed OpenAPI specification
Performance Budget	Declared limits: p95 response time, max DB queries, max external calls per request
Repository Pattern	Layer that isolates database access from business logic
IDOR	Insecure Direct Object Reference — accessing another user's resource via a guessable ID
DLQ	Dead Letter Queue — holds messages that failed after maximum retry attempts
Testcontainers	Library that spins up Docker containers (Postgres, Redis) for isolated integration tests
MSW	Mock Service Worker — intercepts outbound HTTP requests at the network level in tests
Schemathesis	Tool that auto-generates test cases from OpenAPI spec to find schema violations and edge cases
W3C TraceContext	Standard for propagating distributed trace identifiers across service boundaries via HTTP headers

— End of Document  ·  Backend SOP v2.0  ·  April 2026 —
