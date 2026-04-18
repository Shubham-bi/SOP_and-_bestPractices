
STANDARD OPERATING PROCEDURE
Backend Development
API Testing & Business Logic Validation

Lean · Strict · API-First · Test-Driven
Version 1.0  ·  April 2026
 
Purpose & Scope
This SOP establishes standards for backend engineers to design, build, and validate APIs and business logic in a way that is testable by default, reliable under automation, and scalable without over-engineering.
It applies to all HTTP APIs (REST/GraphQL), background jobs, event-driven services, and the business logic layers that power them.

GOAL:  Every API endpoint and every business rule must be independently verifiable. Testability is not an afterthought — it is a design constraint.

Who This Applies To
•	Backend engineers (all experience levels)
•	QA engineers writing API automation tests
•	Tech leads and architects reviewing system design
•	DevOps engineers wiring CI/CD pipelines
•	Any engineer who ships code that another service or client consumes

1 · Mode Definitions — Lean vs Strict
Every endpoint, service, or module you build falls into one of two modes. Decide the mode before you design the interface.

Mode Decision Matrix
Criterion	→ Use Strict Mode When...
Endpoint complexity	3+ branching conditions, multi-step orchestration, or saga patterns
Business criticality	Payments, auth, billing, compliance, data mutation, or financial calculation
Consumer count	Endpoint is consumed by 2+ clients (mobile, web, third-party)
Contract stability	External partners or public API consumers depend on the response schema
Failure blast radius	A bug causes data corruption, incorrect charges, or security breach
Regulatory requirement	GDPR, PCI-DSS, HIPAA, SOC2 or any compliance framework applies
SLA target	Endpoint has a defined response time or availability SLA
Reusability	Business logic is reused across multiple endpoints or services

Mode Comparison
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
Internal or admin tools
Single-client endpoints (internal dashboard)
CRUD with no branching logic
Short-lived or prototype services
Test coverage: happy path only acceptable	Consumer-facing or critical services
Multi-client, versioned, or external APIs
Complex orchestration or domain logic
Shared business rules (pricing, eligibility)
Test coverage: full matrix required

RULE:  If an endpoint touches money, auth, user data, or another team's contract — it is Strict. No exceptions.

2 · API Design & Contract Standards
2.1  Design-First Principle
APIs must be designed as contracts before implementation begins. The contract is the source of truth — not the code.

⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
Contract defined informally in PR description or README
Schema can be inferred from implementation
Breaking changes acceptable in early iterations
No formal review required	OpenAPI 3.x / GraphQL schema committed before code
Schema reviewed and approved before implementation
Breaking changes require versioning and migration plan
Contract tests run on every CI build

2.2  REST Endpoint Naming & HTTP Standards
Rule	Correct Example
Use nouns for resources, never verbs	GET /orders — not GET /getOrders
HTTP method carries the action semantics	POST /orders (create), DELETE /orders/:id (delete)
Use plural resource names consistently	/users/:id/orders — not /user/:id/order
Nested resources for ownership only	/users/:id/addresses — not /addresses?userId=...
Return 201 Created for successful POST	with Location header pointing to new resource
Return 204 No Content for DELETE success	never return 200 with an empty body for deletes
Use 4xx for client errors, 5xx for server	409 Conflict for duplicate — not 500 Internal Error
Pagination on all list endpoints	?page=1&limit=20 or cursor-based for large datasets

2.3  Response Contract Standards
Every API response must follow a consistent envelope. Consistency enables generic error handling in all consumers and simplifies contract testing.

// ✅ Standard success response
{
  "data": { ... },           // Actual payload
  "meta": {                  // Pagination, timing, version
    "total": 150,
    "page": 1,
    "limit": 20
  }
}

// ✅ Standard error response
{
  "error": {
    "code": "ORDER_NOT_FOUND",    // Machine-readable, stable
    "message": "Order 123 was not found.",  // Human-readable
    "field": "orderId",            // Optional: for validation errors
    "traceId": "abc-xyz-123"       // Correlates to logs
  }
}

2.4  API Versioning
•	Use URL versioning for major breaking changes: /v1/orders, /v2/orders
•	Never remove or rename a field in a minor version — add a new field alongside
•	Mark deprecated fields with a sunset date in OpenAPI docs
•	Maintain at minimum N-1 version in production until all consumers migrate
•	Version headers (Accept: application/vnd.api+json;version=2) are acceptable for internal-only APIs

3 · API Testing Standards
3.1  Test Pyramid for Backend
Backend testing follows a four-layer pyramid. Each layer has a specific role — do not substitute one for another.

Layer	Purpose & Scope
Unit Tests (70% of suite)	Pure functions, domain logic, validators, transformers. No I/O. Millisecond execution.
Integration Tests (20% of suite)	Service + repository + database together. Validates that layers compose correctly. Uses real DB in a container.
API / Contract Tests (8% of suite)	Full HTTP request → response cycle. Validates status codes, schema, headers, and error responses. Uses test DB or MSW.
E2E / Journey Tests (2% of suite)	Multi-endpoint user journeys (register → checkout → order confirmation). Run in staging only.

3.2  Unit Test Standards
•	Every pure function, validator, and transformer must have unit tests
•	Test one behaviour per test — never assert multiple unrelated things in one test case
•	Follow Arrange–Act–Assert structure explicitly in every test
•	Use descriptive test names: 'should return 400 when email is missing' not 'test email'
•	Cover: happy path, all boundary values, all error branches, and null/undefined inputs
•	Never test framework internals — test the behaviour your code implements

// ✅ Correct unit test structure (Arrange-Act-Assert)
describe('calculateOrderTotal', () => {
  it('should apply percentage discount to subtotal', () => {
    // Arrange
    const items  = [{ price: 10000, qty: 2 }]; // cents
    const coupon = { type: 'PERCENT', value: 10 };

    // Act
    const result = calculateOrderTotal(items, coupon);

    // Assert
    expect(result.discount).toBe(2000);
    expect(result.total).toBe(18000);
  });
});

3.3  Integration Test Standards
•	Use an in-process test database (Docker Compose or testcontainers) — never a shared dev DB
•	Seed test data in beforeEach — never rely on leftover state from a prior test
•	Test the full service method including repository calls and transaction handling
•	Assert on database state after mutations, not just return values
•	Reset DB between test suites — use transactions that rollback or a truncation helper

// ✅ Integration test with DB assertion
it('should persist order and deduct inventory', async () => {
  await seedInventory({ productId: 'P1', stock: 5 });

  await orderService.create({ productId: 'P1', qty: 2 });

  const inv = await db.inventory.findOne({ productId: 'P1' });
  expect(inv.stock).toBe(3);  // Verify DB state directly
});

3.4  API / Contract Test Standards
API tests make real HTTP calls to a running server instance with a seeded test database. They own the response contract.

•	Test every documented status code for every endpoint (200, 201, 400, 401, 403, 404, 409, 500)
•	Assert on response body schema using a JSON Schema validator or contract library (Pact, Dredd)
•	Assert on response headers: Content-Type, Cache-Control, X-Request-Id
•	Include negative tests: malformed input, missing required fields, invalid types, oversized payloads
•	Test rate limiting headers (X-RateLimit-Remaining) and 429 responses
•	Run contract tests as part of CI before any deployment

// ✅ API test — covers status + schema + error code
it('returns 404 with ORDER_NOT_FOUND when order missing', async () => {
  const res = await request(app)
    .get('/v1/orders/non-existent-id')
    .set('Authorization', `Bearer ${testToken}`);

  expect(res.status).toBe(404);
  expect(res.body.error.code).toBe('ORDER_NOT_FOUND');
  expect(res.body.error.traceId).toBeDefined();
});

3.5  Test Naming Convention
Pattern	Example
[endpoint] [condition] → [expected outcome]	POST /orders with duplicate idempotency key → returns 409
should [behaviour] when [condition]	should reject request when auth token is expired
[method] returns [status] for [scenario]	GET /users/:id returns 403 for non-owner

4 · Business Logic Layer Design & Validation
4.1  Layer Separation (Non-Negotiable)
Business logic must never live in route handlers, middleware, or database queries. It lives exclusively in the service layer.

Layer	Responsibility — What Belongs Here
Route / Controller	Parse request, call service, return response. Zero business logic. Max 15 lines.
Service / Use Case	All business rules, validations, orchestration, and decisions. No HTTP context.
Repository / DAO	Database access only. No business conditions or if/else based on domain rules.
Domain Model / Entity	Data shape + invariant enforcement (e.g., cannot set quantity below zero).
Validator	Input schema validation only — not business rule validation.

WHY THIS MATTERS FOR TESTING:  When business logic is in the service layer only, you can unit test every rule in isolation — no HTTP server, no database, no framework needed. This is the single most impactful structural decision for test speed and reliability.

// ❌ Bad — business logic leaking into controller
router.post('/orders', async (req, res) => {
  if (req.body.items.length === 0) return res.status(400).json({...});
  if (req.user.creditLimit < total) return res.status(402).json({...});
  // 50 more lines of rules...
});

// ✅ Good — controller delegates to service
router.post('/orders', async (req, res) => {
  const result = await orderService.createOrder(req.body, req.user);
  res.status(201).json({ data: result });
});

4.2  Business Rule Validation vs Input Validation
Type	Where It Lives & How to Test
Input / Schema Validation (Is the data well-formed?)	Validator layer — test with unit tests using valid/invalid schema fixtures. Use Zod, Joi, or class-validator.
Business Rule Validation (Is the operation allowed?)	Service layer — test with unit tests using mock repositories. Assert thrown domain errors.
Cross-Entity Validation (Does state allow this?)	Service layer — test with integration tests using seeded DB state.
Authorization Check (Is this user allowed?)	Service layer (not middleware alone) — test with unit + API tests covering all permission combinations.

4.3  Domain Error Design
Business rule violations must throw typed, named errors — not generic Error objects or HTTP status codes from within the service.

•	Create a domain errors module with named error classes for every failure case
•	Each domain error carries: code (machine-readable), message (human-readable), and optional context
•	The controller layer maps domain errors to HTTP status codes — not the service
•	Domain errors are fully unit-testable without an HTTP context

// ✅ Domain error definition
export class InsufficientInventoryError extends DomainError {
  constructor(productId: string, requested: number, available: number) {
    super('INSUFFICIENT_INVENTORY',
      `Product ${productId} has ${available} units, ${requested} requested`);
    this.context = { productId, requested, available };
  }
}

// ✅ Unit test for domain error
it('throws InsufficientInventoryError when stock is too low', async () => {
  inventoryRepo.findById.mockResolvedValue({ stock: 1 });
  await expect(orderService.create({ qty: 5 }))
    .rejects.toThrow(InsufficientInventoryError);
});

4.4  Boundary Value & Edge Case Matrix
For every business rule, derive a test matrix covering all meaningful input boundaries. Document it.

Rule	Boundary Cases to Test	Expected Outcome
Minimum order value	total = 0, total = 0.01, total = minimum - 0.01, total = minimum	Error on 0 and below minimum; pass at minimum
Discount code expiry	now = expiry - 1s, now = expiry, now = expiry + 1s	Pass before expiry; reject at and after
Max items per order	qty = 0, qty = 1, qty = max, qty = max + 1	Error at 0 and above max; pass at max
Duplicate order prevention	Same idempotency key × 1, × 2, × 3 concurrent calls	First succeeds; subsequent return 409
User permission scope	owner, admin, other user, unauthenticated	403 or 401 for non-owner non-admin

4.5  Immutability & Side Effect Rules
•	Service methods that mutate state must be explicit: createOrder, cancelOrder — never processOrder (ambiguous)
•	Pure calculation functions (pricing, tax, eligibility) must be stateless and have 100% unit test coverage
•	Side effects (emails, webhooks, events) must be invokable through an injectable interface for test mocking
•	Publish events after persistence succeeds — never before. Test that events are not emitted when persistence fails

5 · Test Data Management
5.1  Principles
•	Tests must create their own data — never assume pre-existing database rows
•	Test data must be minimal — seed only the fields relevant to the test scenario
•	Use factory functions or builder patterns for creating test fixtures — not hardcoded objects
•	Never use real email addresses, phone numbers, or PII in test fixtures
•	Production data must never be used in automated tests

5.2  Factory Pattern for Fixtures
// ✅ Factory with sensible defaults and overrides
const buildUser = (overrides = {}) => ({
  id:        faker.string.uuid(),
  email:     faker.internet.email(),
  role:      'customer',
  createdAt: new Date('2024-01-01'), // Deterministic date
  ...overrides,                       // Caller provides only what matters
});

// Usage in test
const adminUser = buildUser({ role: 'admin' });
const frozenUser = buildUser({ status: 'suspended', role: 'customer' });

5.3  Database Seeding Strategy
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
Seed once at test suite start using a shared fixture file
Acceptable if tests are read-only
Faster execution for simple CRUD suites
Use a global test DB reset between runs	Seed per test (beforeEach) using factory functions
Required for any write/mutation test
Use DB transactions that rollback after each test
Alternatively: truncate tables + reseed in teardown

NEVER:  Use a shared, persistent test database that multiple developers or CI pipelines access simultaneously. Concurrent tests corrupt each other's data and produce non-deterministic failures.

6 · Authentication & Authorization Testing
6.1  Authentication Test Matrix (Mandatory)
Every protected endpoint must have tests covering all four auth states. These are not optional.

Auth State	Expected Behaviour & Assertions
No token (unauthenticated)	Returns 401 Unauthorized. Error code: MISSING_TOKEN. No data leaked.
Invalid / malformed token	Returns 401 Unauthorized. Error code: INVALID_TOKEN.
Expired token	Returns 401 Unauthorized. Error code: TOKEN_EXPIRED.
Valid token, insufficient role	Returns 403 Forbidden. Error code: INSUFFICIENT_PERMISSIONS.
Valid token, correct role	Returns expected success response.
Valid token, accessing other user's resource	Returns 403 Forbidden (or 404 to avoid enumeration).

6.2  Authorization Test Patterns
•	Test resource ownership: a user must never access another user's data, even with a valid token
•	Test role escalation: a customer-role token must not be able to call admin endpoints
•	Test scope creep: verify tokens are scoped correctly (e.g., read-only tokens cannot mutate)
•	In unit tests, mock the auth context directly — do not spin up an auth server
•	In API tests, use pre-signed test tokens with known expiry, role, and scope

// ✅ Authorization test pattern
describe('DELETE /v1/orders/:id authorization', () => {
  it('returns 403 when requester is not the order owner', async () => {
    const order = await seedOrder({ userId: 'user-A' });
    const token = createTestToken({ userId: 'user-B', role: 'customer' });

    const res = await request(app)
      .delete(`/v1/orders/${order.id}`)
      .set('Authorization', `Bearer ${token}`);

    expect(res.status).toBe(403);
    expect(res.body.error.code).toBe('INSUFFICIENT_PERMISSIONS');
  });
});

7 · Error Handling Standards
7.1  Error Handling Architecture
•	Use a centralised error handler middleware — never write res.status(500).json() inline
•	The error handler maps domain errors → HTTP codes via a registered error map
•	Unhandled errors must default to 500 with a safe message (no stack traces in response)
•	Always include a traceId in every error response for log correlation
•	Log the full error (with stack) server-side — never surface it to the client

7.2  HTTP Status Code Reference
Status Code	Use Case — When to Return This Code
200 OK	Successful GET, PUT, PATCH
201 Created	Successful POST that creates a new resource
204 No Content	Successful DELETE with no response body
400 Bad Request	Client sent malformed data or failed schema validation
401 Unauthorized	Missing, invalid, or expired authentication token
403 Forbidden	Authenticated but lacks permission for this operation
404 Not Found	Resource does not exist (or is hidden from this user)
409 Conflict	Duplicate resource, stale version, or violated uniqueness constraint
422 Unprocessable Entity	Schema valid but business rule violation (use domain error code)
429 Too Many Requests	Rate limit exceeded — include Retry-After header
500 Internal Server Error	Unexpected server-side failure — never expose internals
503 Service Unavailable	Downstream dependency failure or maintenance mode

7.3  Testing Error Paths
•	Every error code in your domain error map must have a corresponding API test
•	Test that 500 errors do not leak stack traces or internal paths in the response body
•	Simulate downstream failures (DB timeout, third-party 503) and assert graceful degradation
•	Test that error responses always include traceId regardless of error type

8 · Database & Repository Layer
8.1  Repository Pattern Rules
•	All database access is behind a repository interface — never call ORM/query builder from services
•	Repository methods are named after domain operations: findActiveOrdersByUser — not SELECT * WHERE
•	Repositories return domain objects, not ORM entity instances
•	Complex queries are encapsulated in named repository methods with individual integration tests
•	Transactions are managed at the service layer, not inside repositories

8.2  Testing the Database Layer
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
Mock the repository in service unit tests using jest.fn()
Verify service logic without hitting DB
Test only what the service does with the data returned	Test repository methods with a real DB container
Verify SQL correctness, indexes, and constraints
Test transaction rollback on failure
Test concurrent write scenarios

8.3  Migration & Schema Testing
•	Run migrations against a clean schema in CI before any integration tests execute
•	Test that migrations are idempotent — running them twice must not corrupt the schema
•	Include a rollback test: apply migration, roll back, verify schema is clean
•	Never skip migrations in test environments — test against the same schema as production

CRITICAL:  Schema drift between test and production databases is the #1 cause of 'works in tests, breaks in prod' incidents. Migrations must run in CI.

9 · Async Workers & Event-Driven Services
9.1  Design Rules for Testability
•	Worker logic must be extracted into a pure handler function separate from the queue consumer
•	The pure handler receives the message payload as input and returns a result — no queue SDK imports
•	Queue producers and consumers are thin wrappers around the pure handler
•	All external calls within workers (DB, HTTP) must be injected or mockable

// ✅ Testable worker pattern
// worker/handlers/sendOrderConfirmation.ts  ← pure, unit testable
export const handleOrderConfirmation = async (payload, deps = defaultDeps) => {
  const order = await deps.orderRepo.findById(payload.orderId);
  await deps.emailService.send({ to: order.email, template: 'order-confirmed' });
  return { sent: true };
};

// worker/consumers/orderConsumer.ts  ← thin wrapper
queue.process('order.confirmed', (job) => handleOrderConfirmation(job.data));

9.2  Testing Async Workers
•	Unit test the handler function directly — no queue infrastructure needed
•	Integration test the full consumer → handler → DB + service chain with a real queue container
•	Test dead letter queue (DLQ) handling: what happens when the handler throws after max retries
•	Test idempotency: processing the same message twice must not create duplicate side effects
•	Assert on the final state in the database — not on mock call counts alone

10 · Mocking Strategy
10.1  What to Mock and When
Dependency Type	Mocking Strategy
Repository / DB (unit tests)	jest.fn() / vi.fn() returning resolved promises with fixture data
External HTTP APIs (unit + integration)	nock, msw/node, or WireMock — intercept at HTTP level
Message queues / event bus	In-process mock broker (e.g. bull-board test mode) or full container
File system / blob storage	Dependency-injected storage interface — mock in unit, real bucket in integration
Email / SMS providers	Injectable interface mock in unit tests; capture mode in integration
Clock / time (Date.now, new Date)	jest.useFakeTimers() or @sinonjs/fake-timers — mandatory for any time-based logic
Randomness (Math.random, UUID generation)	Seed-controlled or injectable generator — never allow true randomness in tests

10.2  Mock Boundaries — The Golden Rule
RULE:  Mock at the boundary of your system, not inside it. Mock external dependencies (DB, HTTP, queue) — never mock your own service methods or domain logic. If you mock your own code, you are testing nothing.

// ❌ Wrong — mocking your own service in another service's test
jest.mock('./inventoryService');  // This tests nothing real
inventoryService.check.mockResolvedValue(true);

// ✅ Correct — mock only the external boundary (repository)
inventoryRepo.findByProductId.mockResolvedValue({ stock: 5 });
// Now your actual service logic runs against mock data

11 · Performance & Load Testing Guidelines
11.1  Performance Baselines (Mandatory)
Every Strict-mode endpoint must have a defined and tested performance baseline before going to production.

Endpoint Category	Target Response Time (p95)
Read — simple lookup (by ID)	< 100ms
Read — filtered list with pagination	< 300ms
Write — simple create / update	< 500ms
Write — multi-step transaction	< 1000ms
Background job / async task	< 5s (SLA on job completion, not enqueue)
Report / aggregate query	< 3s (with explicit user-facing loading state)

11.2  Performance Test Rules
•	Use k6, Artillery, or Locust — not JMeter (deprecated in modern pipelines)
•	Run load tests in an isolated staging environment — never against production
•	Load test must include ramp-up, sustained load, and spike scenarios
•	Define and test breaking point: at what concurrency does p99 exceed SLA?
•	Assert on error rate (< 0.1%) not just response time during load runs

11.3  Query Performance Standards
•	Every database query used in a production endpoint must use an index — verified with EXPLAIN ANALYZE
•	N+1 query patterns are a bug — detect with query count assertions in integration tests
•	Paginate all list endpoints — no unbounded queries
•	Add query count assertions to critical integration tests using a query counter interceptor

// ✅ N+1 detection in integration test
it('fetches order list without N+1 queries', async () => {
  await seedOrders(20);
  const queryCount = await countQueries(() =>
    orderService.listByUser(userId));
  expect(queryCount).toBeLessThanOrEqual(2); // 1 order + 1 items join
});

12 · Security Testing Baseline
12.1  Mandatory Security Tests
These are minimum required tests for every external-facing API. They are not security audits — they are baseline sanity checks.

•	SQL / NoSQL injection: send ' OR 1=1 --, $where, and operator payloads in every string parameter
•	Mass assignment: attempt to write protected fields (role, id, createdAt) via POST/PUT body
•	IDOR (Insecure Direct Object Reference): attempt to access resource IDs belonging to other users
•	Oversized payloads: send requests exceeding configured body size limit and assert 413
•	Header injection: test for CRLF injection in any endpoint that reflects user input in headers
•	Sensitive data leakage: assert that passwords, tokens, and secrets never appear in response bodies

12.2  Security Test Implementation
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
Manual spot-checks are acceptable for internal tools
Basic IDOR checks in existing test suite
No automated security scan required	OWASP ZAP or Semgrep in CI pipeline
Full IDOR test matrix covering every resource type
Automated secret scanning (TruffleHog, GitLeaks)
Dependency vulnerability scan (npm audit, Snyk)

13 · CI/CD Integration & Enforcement
13.1  Pipeline Gate Requirements
Tests are not useful unless they block merges and deployments. The following gates are mandatory.

Pipeline Stage	Required Gates
Pull Request	Unit tests pass · Integration tests pass · Coverage threshold met · Contract tests pass · Linting & type check pass
Merge to Main	All PR gates + API test suite pass + Security baseline scan pass + Migration validation pass
Deploy to Staging	All main gates + Load test smoke pass + E2E journey tests pass
Deploy to Production	All staging gates + Manual QA sign-off for Strict-mode features + Rollback plan confirmed

13.2  Coverage Thresholds
Mode / Layer	Minimum Coverage Requirement
Lean — overall	60% statement coverage
Strict — overall	80% statement coverage
Strict — service / business logic	95% branch coverage (mandatory)
Strict — domain error paths	100% — every domain error must have a test
Auth & permissions	100% — every permission state must be tested

13.3  Test Reporting
•	Output test results in JUnit XML format for CI dashboard visibility
•	Track flaky test rate per suite — flag suites where > 2% of runs are non-deterministic
•	Publish coverage delta per PR — block merge if coverage drops more than 2%
•	Archive test run artifacts (results, logs, screenshots for E2E) with a 30-day retention

FLAKY TEST POLICY:  A test that fails without a code change is a defect. Quarantine it within 24 hours. Root cause and fix within one sprint. Accumulating flaky tests will erode team confidence and the entire CI investment.

14 · Folder Structure
14.1  Lean Structure
For internal tools, prototypes, or services with fewer than 10 endpoints.

src/
  routes/           # Express/Fastify route definitions
    orders.ts
  services/         # Business logic
    orderService.ts
  repositories/     # DB access
    orderRepository.ts
  validators/       # Zod/Joi schemas
  errors/           # Domain error classes
  utils/
  __tests__/        # All tests co-located
  app.ts

14.2  Strict Structure (Feature-First)
For production services, multi-team APIs, or codebases expected to scale.

src/
  features/
    orders/
      routes/
        orders.routes.ts          # Route definitions only
        orders.routes.test.ts     # API / contract tests
      services/
        orderService.ts           # Business logic
        orderService.test.ts      # Unit tests
      repositories/
        orderRepository.ts
        orderRepository.test.ts  # Integration tests
      validators/
        createOrder.schema.ts
        createOrder.schema.test.ts
      errors/
        order.errors.ts
      types/
        order.types.ts
      index.ts                   # Feature public API
  shared/
    middleware/                  # Auth, logging, error handler
    errors/                      # Base DomainError class
    database/                    # Connection, migrations
    types/
  test/
    factories/                   # buildUser(), buildOrder() etc.
    helpers/                     # seedDB, createTestToken, countQueries
    msw-handlers/                # MSW handlers for external APIs
  app.ts
  server.ts

15 · Documentation Standards
15.1  What Requires Documentation
Item	Required Documentation Level
Simple CRUD endpoint	OpenAPI docstring on route — parameters, responses, error codes
Complex business rule / service method	JSDoc explaining the domain rule being enforced, not the code
Domain error class	When this error is thrown, what it means, and how the client should respond
Repository method with complex query	Comment explaining why this query structure was chosen
Auth / permission logic	Explicit list of who can and cannot call this — do not leave implicit
Workaround or non-obvious code	MANDATORY: // TODO: [JIRA-XXX] comment with reason and ticket
Performance trade-off	Comment explaining why denormalisation, caching, or skip-logic was chosen

15.2  OpenAPI Documentation Rules
•	Every endpoint must have a summary, description, request schema, and response schemas
•	Every error response must be documented with its error.code value
•	Mark deprecated endpoints with deprecated: true and include a sunset date
•	Generate OpenAPI spec from code annotations — never maintain it manually alongside code
•	Publish the spec to an internal developer portal or Swagger UI on every main branch deploy

16 · Quick Decision Checklist
Run through this checklist before marking any endpoint or service as complete.

	Checklist Item	Mode
☐	Have I decided Lean vs Strict using the mode decision matrix?	Both
☐	Is all business logic in the service layer — none in routes or repositories?	Both
☐	Does every endpoint return a consistent response envelope?	Both
☐	Is every domain rule violation throwing a typed domain error?	Both
☐	Have I covered all 4 authentication states: missing, invalid, expired, insufficient role?	Strict
☐	Does every async operation (DB, HTTP, queue) have error handling and is it tested?	Both
☐	Are repositories returning domain objects, not ORM entities?	Strict
☐	Are Date.now(), Math.random(), and UUIDs injectable or mockable?	Both
☐	Does the test suite cover loading, success, and every failure branch?	Both
☐	Is there a boundary value test matrix for every business rule with thresholds?	Strict
☐	Has the migration been tested: apply, rollback, and idempotency?	Both
☐	Is the OpenAPI spec updated to reflect all changes including new error codes?	Strict
☐	Are performance baselines defined and tested for all Strict endpoints?	Strict
☐	Have I run the security baseline checks: IDOR, mass assignment, oversized payload?	Strict
☐	Is coverage at or above the threshold for this mode?	Both

17 · Do's and Don'ts
17.1  API Design
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
✅  DO
Use HTTP methods semantically (GET reads, POST creates)
Return consistent error envelopes with typed codes
Version breaking changes — never silently change contracts
Design the OpenAPI spec before writing implementation code	❌  DON'T
Return 200 OK with error details buried in the body
Use verb-based URLs: /getUser, /createOrder, /deleteItem
Return stack traces or internal error messages to clients
Change response schema without a deprecation notice

17.2  Business Logic
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
✅  DO
Put all business rules in the service layer exclusively
Throw typed domain errors for every rule violation
Keep pure calculation functions stateless and 100% tested
Map API responses to domain objects at the boundary	❌  DON'T
Write if/else business logic inside route handlers
Throw generic new Error('something went wrong')
Embed raw SQL conditions that encode business rules
Let ORM entity objects leak into service layer return values

17.3  Testing
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
✅  DO
Follow Arrange–Act–Assert in every test
Test one behaviour per test case
Seed test data in beforeEach with factories
Assert on database state after mutations, not just return values	❌  DON'T
Test framework code (Express routing internals)
Share state between tests or rely on test order
Use real email addresses or PII in test fixtures
Mock your own service methods within another service's test

17.4  Mocking & Isolation
⚡  LEAN  —  Fast & Pragmatic	🏗️  STRICT  —  Robust & Validated
✅  DO
Mock at the system boundary: DB, HTTP, queue
Use jest.useFakeTimers() for time-sensitive logic
Use injectable dependencies for third-party services
Use MSW or nock to mock outbound HTTP at network level	❌  DON'T
Mock internal service or domain methods
Use setTimeout delays in tests for async stabilisation
Import live API keys or credentials in test environments
Let one test's DB writes bleed into another test's reads

Appendix · Key Terms
Term	Definition
Lean Mode	Development approach for low-complexity, low-risk backend code with minimal structure overhead
Strict Mode	Structured, fully validated approach for critical, consumer-facing, or complex backend services
Domain Error	A typed, named error class that represents a business rule violation — independent of HTTP
Repository Pattern	Abstraction layer that isolates database access from business logic
Contract Test	A test that verifies a service's API response matches a declared schema or Pact contract
Idempotency	Property where running an operation multiple times produces the same result as running it once
IDOR	Insecure Direct Object Reference — accessing another user's resource via a guessable ID
N+1 Query	Anti-pattern where a list query fires one additional query per item returned
Test Pyramid	Model prescribing more unit tests than integration tests and more integration than E2E tests
MSW	Mock Service Worker — intercepts outbound HTTP requests at the network level in Node.js tests
DLQ	Dead Letter Queue — holds messages that failed processing after maximum retry attempts
Testcontainers	Library that spins up Docker containers (Postgres, Redis, etc.) programmatically for tests

— End of Document —
