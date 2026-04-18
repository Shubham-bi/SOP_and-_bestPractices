
STANDARD OPERATING PROCEDURE
Software Quality &
Testing Engineering
ISTQB-Aligned · Agile SDLC/STLC · Automation-First · Team-Scalable
Level 1 (Basic)  ·  Level 2 (Intermediate)  ·  Level 3 (Advanced)

Version 3.0  ·  April 2026
 
0 · 1-Page Quick Start Guide
Before reading the full SOP, use this guide to understand what applies to your team right now. Start at your adoption level and build up over time.

E2E  ·  2–10%
Performance & Security  ·  ~5%
API / Contract  ·  ~8%
Integration  ·  ~20%
Unit Tests  ·  ~65%

⚡ Level 1 — Basic (Start Here)
•	Write unit tests for every function
•	One API test per endpoint (happy path)
•	Manual exploratory testing in staging
•	Basic CI: lint + unit tests pass
•	Error handling with traceId in responses
•	GWT acceptance criteria on all stories
	🔧 Level 2 — Intermediate
•	Integration tests with real DB (Testcontainers)
•	Full status code matrix for APIs
•	Contract tests with jest-openapi
•	E2E golden path tests (Playwright)
•	Auth matrix: all 6 states tested
•	Session-based exploratory testing


🏆 Level 3 — Advanced Full Engineering Maturity
•	Mutation testing with Stryker (validates assertion quality)
•	Observability testing: traceId, metrics, distributed traces
•	Performance budgets enforced in CI per endpoint
•	Consumer-driven contract testing with Pact
•	Automated DAST (OWASP ZAP) in deployment pipeline
•	Defect escape analysis mapped to test gaps

Who Does What — At a Glance
Activity	Dev	QA	DevOps	Tech Lead	PO
Write unit tests	R	C	—	A	—
Write API tests	R	R	—	A	—
Write E2E tests	C	R	—	A	—
Manual exploratory testing	C	R	—	A	I
Set up CI pipeline	C	C	R	A	—
Fix flaky tests	R	R	—	A	—
Define acceptance criteria	C	C	—	A	R
Sign off before release	I	R	I	A	R

R = Responsible  ·  A = Accountable  ·  C = Consulted  ·  I = Informed  ·  — = Not Involved

Pre-PR Checklist — 60-Second Version
	Level 1 & 2 (Always)		Level 2 & 3 (STRICT)
	☐ Unit tests for every new function		☐ Full status code matrix (API)
	☐ Happy path API test		☐ Auth matrix: all 6 states
	☐ Error states handled		☐ Contract test: toSatisfyApiSpec()
	☐ No hardcoded sleep() in tests		☐ DB state asserted after mutations
	☐ Test data in beforeEach		☐ traceId verified in error response
	☐ No PII in fixtures		☐ Security baseline (IDOR, mass assign)
	☐ jest-axe on components		☐ Golden Path passes in staging

1 · Adoption Level System
This SOP is designed to scale with your team. Start at Level 1 — get that right first. Only move to the next level when the current level is stable and consistently practiced.

PRINCIPLE:  A team that reliably does Level 1 is more effective than a team that attempts Level 3 inconsistently. Quality is about discipline, not tool count.

⚡ Level 1 — Basic Automation  (All teams — mandatory from day one)
•	Unit tests for all new business logic (Jest/Vitest)
•	At least one API test per endpoint covering the happy path
•	Manual exploratory testing session in staging before every release
•	Basic CI pipeline: lint + unit tests gate every PR
•	Error handling: every endpoint returns error.code + traceId
•	Acceptance criteria written in GWT format before coding

🔧 Level 2 — Intermediate Quality  (Teams with 3+ months of Level 1 consistency)
•	Integration tests with real isolated DB (Testcontainers or Docker Compose)
•	Full API status code matrix (200 through 500 range)
•	Contract testing with jest-openapi on every happy-path API test
•	E2E golden path tests with Playwright for critical user journeys
•	Auth test matrix: all 6 states for every protected endpoint
•	Session-based exploratory testing with charters
•	Test case traceability: story → test case → defect in Jira/Xray

🏆 Level 3 — Advanced Maturity  (Teams with 6+ months of Level 2 consistency — Optional)
•	Mutation testing with Stryker — validates assertion quality, not just coverage
•	Observability testing: assert traceId, errorCode, userId in all log entries
•	Per-endpoint performance budgets enforced in CI (p95 + DB query count)
•	Consumer-driven contract tests with Pact for multi-team APIs
•	Automated DAST (OWASP ZAP) running on every staging deployment
•	Defect escape post-mortems mapped to specific test layer gaps
•	Weekly test health metrics reviewed in Sprint Retrospective

Level	Tool Stack Required
Level 1	Jest/Vitest · Supertest · basic CI (GitHub Actions / GitLab CI)
Level 2	+ Testcontainers · Playwright · jest-openapi · MSW · Jira/Xray or TestRail
Level 3	+ Stryker · k6 / Artillery · OWASP ZAP · Pact · Prometheus/Grafana (optional)

2 · Testing Philosophy & ISTQB Principles
This SOP is grounded in the seven ISTQB testing principles. Each is mapped to a concrete practice enforced in this document.

ISTQB Principle	Practical Enforcement
1. Testing shows presence of defects, not absence	Coverage thresholds are floors. Assertion quality validated via mutation testing (Level 3) and PR review (all levels).
2. Exhaustive testing is impossible	Risk-based via Service Tier (Section 4). Tier 0 gets deeper coverage. Tier 2 gets LEAN treatment.
3. Early testing saves time (Shift Left)	Tests planned in sprint refinement. GWT acceptance criteria before coding. SAST on every commit.
4. Defects cluster (Pareto)	Tier 0 services — payments, auth — get more tests, more E2E coverage, and more security scrutiny.
5. Beware of pesticide paradox	Test suites reviewed quarterly. Mutation testing at Level 3 validates tests still detect changes.
6. Testing is context dependent	Tier and Level system means standards scale to service criticality and team maturity.
7. Absence-of-errors fallacy	100% coverage with weak assertions fails users. PR review checks assertion quality, not just presence.

Agile Testing Quadrants (Brian Marick)
Quadrant	Tests	Purpose	Owner
Q1 — Technology, Support	Unit, Integration	Guide development; fast feedback loop	Developer
Q2 — Business, Support	API contract, Functional, BDD	Validate acceptance criteria against requirements	Dev + QA
Q3 — Business, Critique	Exploratory, Usability, UAT, E2E	Find defects humans notice; validate real user flows	QA + PO
Q4 — Technology, Critique	Performance, Security, Load	Non-functional quality attributes	Dev + DevOps

3 · Agile SDLC & STLC Integration
3.1  Testing Activities per Sprint Phase
Sprint Phase	Testing Activities (STLC Touchpoints)
Backlog Refinement	QA reviews stories for testability. GWT criteria defined. Non-testable stories rejected. Test conditions identified. Effort estimated.
Sprint Planning	Test cases linked to acceptance criteria. Non-functional requirements (perf, security) flagged per service tier.
Development (Daily)	TDD/BDD encouraged: red-green-refactor. Unit + integration tests written alongside code. CI gates block failing PRs.
Code Review / PR	Dev reviews logic. QA (or dev) reviews test quality using Section 19 checklist. Assertions, edge cases, boundary values verified.
Definition of Done	All DoD criteria verified. Coverage met. Contract tests passing. GWT criteria mapped to passing tests.
Sprint Review	Exploratory testing session against new features in staging. QA executes session-based testing. Findings logged.
Retrospective	Test health reviewed: coverage delta, flaky count, defect escape rate. Process improvements identified.

3.2  GWT Acceptance Criteria Format
All user stories must have GWT criteria before development begins. These map directly to automated test cases.

# Story: User applies a discount coupon at checkout

Given: cart has items totalling £100
  And: valid 10% coupon SAVE10 exists
When:  user applies SAVE10
Then:  cart total updates to £90
  And: applied coupon shown with expiry date
  And: order summary reflects the £10 discount

# This maps directly to:
it('reduces cart total by 10% when valid PERCENT coupon applied', ...)

3.3  Definition of Done — Test Requirements
Mode	DoD Test Requirements
LEAN / Level 1	Unit tests for core logic · At least one happy-path API test · Error handling with traceId · GWT mapped to passing tests
STRICT / Level 2	All layers: unit + integration + API · Coverage per tier · OpenAPI updated · Auth matrix complete · Acceptance criteria verified
Full / Level 3	All Level 2 requirements + observability assertions + performance budget verified + security baseline passing

4 · Service Tier Strategy
Test depth scales with service criticality. This three-tier system ensures teams invest in testing proportionally to risk — avoiding both under-testing of critical paths and over-engineering of low-risk CRUD.

Tier	Applies To
Tier 0 ★ — Critical	Payments · Authentication · Billing · Compliance · PII data · Financial calculations
Tier 1 — Core	Main product features: checkout, onboarding, search, notifications, user management
Tier 2 — Standard	Internal tools · Admin dashboards · Low-risk CRUD · Reporting · Config management

Tier	Coverage & Standards
Tier 0	90–95% branch on service layer · 100% domain errors · 100% auth paths · Golden Path E2E · Perf budget enforced
Tier 1	80–85% branch on service layer · Full API status matrix · Auth matrix · Basic perf target
Tier 2	60–80% statement overall · Happy path + basic error tests · LEAN acceptable
TIER 0 RULE:  Any service touching money, auth, PII, or a regulatory requirement is Tier 0. This cannot be overridden without CTO approval.

5 · Roles & Responsibilities (RACI)
Clear ownership prevents gaps. This matrix defines who is Responsible, Accountable, Consulted, and Informed for every major testing activity.

Activity	Dev	QA	DevOps	Tech Lead	PO
Write unit tests for features	R	C	—	A	—
Write integration tests	R	C	—	A	—
Write API / contract tests	R	R	—	A	—
Write E2E / golden path tests	C	R	—	A	—
Manual exploratory testing	C	R	—	A	I
UAT support & coordination	C	R	—	A	R
Maintain test data factories	R	R	—	A	—
Set up and maintain CI pipeline	C	C	R	A	—
Monitor and fix flaky tests	R	R	C	A	—
Define acceptance criteria (GWT)	C	C	—	A	R
Maintain test case traceability	C	R	—	A	—
Performance / load testing	R	C	R	A	—
Security baseline testing	R	R	C	A	—
Sign off on release quality	I	R	I	A	R
SOP maintenance and updates	C	C	—	R	—
Review test health in retrospective	C	R	—	R	I

R = Responsible (does the work)  ·  A = Accountable (owns the outcome)  ·  C = Consulted (input required)  ·  I = Informed (kept updated)  ·  — = Not involved

6 · Unit Testing Standards
6.1  What to Test
Test This	Do NOT Test This
Business logic: calculations, validations, transformations	Framework internals (Express routing, React reconciliation)
Every conditional branch	Third-party library implementations
Every domain error condition	Simple pass-through getters with no logic
Edge cases: nulls, empty arrays, max/min, type coercions	Auto-generated code or pure configuration objects

6.2  AAA Structure — Mandatory
describe('calculateDiscount', () => {
  it('should apply PERCENT discount to subtotal', () => {
    // ── Arrange ─────────────────────────────────────────
    const subtotal = 10000; // pence — avoids float errors
    const coupon   = { type: 'PERCENT', value: 10 };

    // ── Act ──────────────────────────────────────────────
    const result = calculateDiscount(subtotal, coupon);

    // ── Assert ───────────────────────────────────────────
    expect(result.discountAmount).toBe(1000);
    expect(result.finalTotal).toBe(9000);
  });
});

7 · Manual & Exploratory Testing Strategy
Manual testing is not a lesser form of testing — it complements automation by finding defects that scripted tests miss. It covers user experience, edge cases, and combinations that are impractical to automate.

PRINCIPLE:  Automation confirms that what was working before still works. Exploratory testing discovers what was never tested in the first place.

7.1  When Manual Testing Is Required
Scenario	Manual Test Type
New feature or major change going to staging	Session-based exploratory testing using a charter
Pre-release regression validation	Structured manual regression on critical flows not covered by E2E
UAT (User Acceptance Testing)	Stakeholder-driven acceptance testing with PO + selected users
Accessibility usability check	Screen reader + keyboard navigation — not just automated axe
Mobile / device-specific behaviour	Device lab or BrowserStack exploratory session
Complex UI interactions (drag-drop, rich text)	Charter-based exploration — automation brittle here

7.2  Session-Based Exploratory Testing (SBET)
SBET is the standard format for exploratory testing in this team. Every manual test session follows this structure.

Element	Description
Charter	The mission: what area to explore and what questions to answer. Written before the session starts.
Duration	Time-boxed: 60–90 minutes. No session shorter than 30 minutes (not enough depth).
Test Notes	Free-form notes taken during the session: what was tried, what was found, what was skipped.
Bug Reports	Any defects found are logged in Jira immediately with steps to reproduce, severity, and screenshot.
Debrief	5-minute debrief after the session: findings summary, coverage gaps, follow-up sessions needed.

# ✅ Sample Exploratory Testing Charter
Mission:    Explore the coupon application flow during checkout
Focus:      Edge cases, error messages, concurrency, expired coupons
Duration:   60 minutes
Tester:     QA Engineer (Priya)
Build:      Staging — commit abc1234

Questions to answer:
  - Can a coupon be applied after the cart is modified?
  - What happens with two coupons applied at once?
  - Is the error message clear when the coupon is expired?
  - Does the discount persist through the full checkout flow?

7.3  Bug Discovery Techniques
Technique	How to Apply
Boundary Value Analysis	Test at and just outside every limit: min, min-1, max, max+1. E.g., order quantity 0, 1, 99, 100, 101.
Equivalence Partitioning	Divide inputs into valid and invalid classes, test one value from each. Don't test every invalid input.
Error Guessing	Use experience to guess likely defects: empty fields, special characters, very long strings, negative numbers.
State Transition	Map all states a feature can be in and test every transition. E.g., cart: empty → has items → checked out → confirmed.
Decision Table Testing	For features with multiple conditions, create a table of all combinations and test each row.
Negative Testing	Deliberately try to break the system: skip required fields, use wrong data types, interrupt mid-flow.

7.4  UAT Support Process
•	QA prepares UAT test scenarios derived from acceptance criteria and business workflows
•	Test scenarios are written in plain English — no technical jargon — for stakeholder execution
•	QA facilitates but does not execute UAT — stakeholders (PO, business users) run the tests
•	Defects found in UAT are logged in Jira with UAT label and immediately prioritised
•	UAT sign-off from Product Owner is required before any Tier 0 or Tier 1 feature ships to production
•	UAT test results and sign-off are stored in Jira linked to the release version

7.5  Regression Testing Strategy
Layer	What Gets Regression Tested and How
Automated regression (Level 1+)	Every CI run re-executes unit + API tests. This is the primary regression layer.
E2E golden paths (Level 2+)	Golden path Playwright tests run on every staging deployment — they are the regression suite for critical journeys.
Manual regression	Triggered by: major architectural change, large refactor, or Tier 0 feature. QA executes a structured checklist of known risk areas.
Risk-based manual regression	Before every release: QA identifies the top 5 risk areas based on what changed, and explores those areas for 30 minutes each.

8 · Test Case Management & Traceability
Test cases must be stored, linked to requirements, and traceable to defects. This enables impact analysis, gap detection, and release confidence.

8.1  Tool Recommendation by Team Size
Team Size / Maturity	Recommended Tool
Small team (<10 devs), Level 1	Jira issues with QA label — test cases as subtasks of user stories. Simple, no extra tool.
Mid-size team (10–30), Level 2	Jira + Xray plugin — test cases as first-class Jira entities with execution tracking and traceability.
Larger team (30+), Level 2–3	TestRail or Zephyr Scale — full test management with version-based test plans and reporting.
API-heavy teams	Postman collections stored in version control — linked to Jira via test run comments.

8.2  Traceability Matrix
Every test case must trace back to a requirement and forward to any defect it uncovers. This is the traceability chain.

Business Requirement	→	User Story (GWT criteria)	→	Test Case (manual/auto)	→	Automation (test ID)	→	Defect (if found)

8.3  Test Case Minimum Template
Field	Description & Example
ID	TC-001 (sequential, unique per feature)
Title	User can apply a valid PERCENT coupon at checkout
Story Link	JIRA-1234 (linked to parent story)
Preconditions	User is logged in. Cart has items. Valid coupon SAVE10 exists.
Steps	1. Go to cart. 2. Enter SAVE10 in coupon field. 3. Click Apply.
Expected Result	Cart total reduces by 10%. Coupon label displays with expiry date.
Test Type	Manual / Automated / Both
Automation ID	checkout::applyCoupon::validPercent (links to test file + it() name)
Status	Pass / Fail / Blocked / Not Executed
Defect Link	BUG-5678 (if test failed and a defect was raised)

8.4  Jira + Xray Workflow (Level 2 Standard)
1.	Create Test issue in Jira linked to the parent story
2.	Write test steps using the TC template above
3.	Link automation test ID to the Xray test issue
4.	On PR merge: CI reports automated test results back to Xray automatically
5.	For manual tests: QA executes and marks Pass/Fail in Xray test execution
6.	Any Fail raises a Bug issue linked to both the Test and the Story
7.	Bug is triaged, prioritised, and fixed; re-test closes the loop

9 · API Testing — Beginner to Advanced
This section provides progressive API testing guidance — from a Postman manual test (Level 1) to a fully automated Supertest contract test (Level 2+).

9.1  Level 1 — Postman Manual API Testing
Start here if your team is transitioning from manual testing. Postman tests run from the UI and can be saved as collections in version control.

// ✅ Postman Pre-request Script — generate auth token
pm.sendRequest({
  url:    pm.environment.get('BASE_URL') + '/v1/auth/login',
  method: 'POST',
  body:   { mode: 'raw', raw: JSON.stringify({ email: 'test@example.com', password: 'Test1234!' }) },
}, (err, res) => {
  pm.environment.set('auth_token', res.json().data.token);
});

// ✅ Postman Test Script — assertions on GET /v1/orders/:id
pm.test('Status is 200', ()      => pm.response.to.have.status(200));
pm.test('Has data field', ()     => pm.expect(pm.response.json()).to.have.property('data'));
pm.test('Has traceId', ()        => pm.expect(pm.response.json().meta.traceId).to.be.a('string'));
pm.test('Schema is valid', ()    => {
  const json = pm.response.json();
  pm.expect(json.data).to.have.all.keys(['id','status','items','total','createdAt']);
});

// ✅ Postman — test 401 when no auth token
pm.test('Returns 401 without token', () => pm.response.to.have.status(401));
pm.test('Has MISSING_TOKEN code', ()  => {
  pm.expect(pm.response.json().error.code).to.eql('MISSING_TOKEN');
});

9.2  Level 2 — Supertest Automated API Tests
Supertest tests run in CI on every PR. They are self-contained — no external server required.

// ✅ Self-contained setup — in-process server
import { createApp } from '../app';
import request       from 'supertest';
import { db }        from '../database';

let app, server;
beforeAll(async () => {
  await db.migrate.latest();     // Real migrations
  app    = await createApp();
  server = app.listen(0);        // Ephemeral port
});
afterAll(async  () => { await server.close(); await db.destroy(); });
beforeEach(async () => { await db.truncateAllTables(); await seedFixtures(); });

// ✅ Happy path — GET /v1/orders/:id
it('returns order for authenticated owner', async () => {
  const order = await seedOrder({ userId: 'u1' });
  const token = createTestToken({ userId: 'u1' });

  const res = await request(app)
    .get(`/v1/orders/${order.id}`)
    .set('Authorization', `Bearer ${token}`);

  expect(res.status).toBe(200);
  expect(res.body.data.id).toBe(order.id);
  expect(res.body.meta.traceId).toBeDefined();
  expect(res).toSatisfyApiSpec();  // Contract assertion
});

// ✅ Auth: returns 401 when token missing
it('returns 401 with MISSING_TOKEN when no auth header', async () => {
  const res = await request(app).get('/v1/orders/any-id');
  expect(res.status).toBe(401);
  expect(res.body.error.code).toBe('MISSING_TOKEN');
  expect(res.body.error.traceId).toBeDefined();
});

// ✅ IDOR: user cannot access another user's order
it('returns 403 when user accesses another users order', async () => {
  const order      = await seedOrder({ userId: 'victim-user' });
  const attackToken = createTestToken({ userId: 'attacker-user' });

  const res = await request(app)
    .get(`/v1/orders/${order.id}`)
    .set('Authorization', `Bearer ${attackToken}`);

  expect(res.status).toBeOneOf([403, 404]);
  expect(res.body).not.toHaveProperty('data');
});

9.3  Status Code Coverage Matrix
Status Code	Trigger Condition	Assert On
200 OK	Valid request, resource exists	Schema match + traceId
201 Created	Valid POST creates new resource	Location header + DB row created
400 Bad Request	Missing or wrong-type field	error.code = VALIDATION_ERROR + field
401 Unauthorized	No token / invalid / expired	error.code = MISSING_TOKEN / INVALID_TOKEN
403 Forbidden	Valid token, wrong role or non-owner	error.code = INSUFFICIENT_PERMISSIONS
404 Not Found	Resource ID does not exist	error.code = [RESOURCE]_NOT_FOUND
422 Unprocessable	Business rule violation	error.code matches domain error constant
500 Internal	Forced throw in test environment	No stack trace in response body

10 · Security Testing — QA-Friendly Guide
Security testing can feel intimidating. This section provides step-by-step, practical examples that any QA engineer can execute — no security background required to start.

10.1  What to Test and Why
Attack	Why It Matters
SQL / NoSQL Injection	An attacker sends malicious input that alters a database query — potentially exposing all data or deleting records.
IDOR	User A can access or modify User B's data simply by changing an ID in the URL or request body.
Mass Assignment	An attacker sends extra fields (like role: 'admin') hoping the API will accept and persist them.
Oversized Payloads	An attacker sends a huge request body to crash the server or cause a denial-of-service.
Auth Bypass	An attacker accesses protected resources without valid authentication by exploiting token gaps.

10.2  SQL / NoSQL Injection — Step by Step
Test every field that accepts text input — search, filters, IDs, usernames.

Step	What to Do
1. Identify input fields	List every query parameter and request body field that accepts a string value.
2. Send injection payload	Replace the normal value with: ' OR 1=1-- or ' OR '1'='1 or 1; DROP TABLE users--
3. Assert safe response	API must return 400 Bad Request or 422 Unprocessable — never 200 with data or a 500 error.
4. Check response body	Error message must not reveal database type, table names, or query structure.

// ✅ Postman — SQL injection test
// Request: GET /v1/users?search=' OR 1=1--
pm.test('Rejects SQL injection', () => {
  pm.expect(pm.response.code).to.be.oneOf([400, 422]);
  pm.expect(pm.response.text()).to.not.include('syntax error');
  pm.expect(pm.response.text()).to.not.include('SQL');
});

// ✅ Supertest — NoSQL injection test
it('rejects NoSQL injection in filter param', async () => {
  const res = await request(app)
    .get('/v1/orders')
    .query({ userId: { '$gt': '' } })   // MongoDB operator injection
    .set('Authorization', bearer);
  expect(res.status).toBeOneOf([400, 422]);
  expect(res.body.error.code).toBe('VALIDATION_ERROR');
});

10.3  IDOR (Insecure Direct Object Reference) — Step by Step
Step	What to Do
1. Create two test users	User A (attacker) and User B (victim) — use your API or DB seed to create both.
2. Create a resource as User B	Create an order, profile, file, or document owned by User B via your API.
3. Try to access it as User A	Make the same GET, PUT, DELETE request using User A's auth token and User B's resource ID.
4. Assert 403 or 404 response	The API must return 403 (access denied) or 404 (resource not found, to prevent enumeration).
5. Assert no data leaked	The response body must not contain any of User B's data fields.

// ✅ IDOR test — Postman
// Step 1: login as User B, create order, save orderId
// Step 2: login as User A, copy auth token
// Step 3: GET /v1/orders/{User B's orderId} with User A's token
pm.test('IDOR blocked: 403 or 404 returned', () => {
  pm.expect(pm.response.code).to.be.oneOf([403, 404]);
});
pm.test('No data leaked in body', () => {
  pm.expect(pm.response.json()).to.not.have.property('data');
});

10.4  Mass Assignment — Step by Step
Step	What to Do
1. Identify protected fields	Check the user model or entity for fields that should never be set by clients: role, id, balance, isVerified, createdAt.
2. Include them in POST/PUT body	Send a create or update request with the protected field included: { name: 'Test', role: 'admin' }.
3. Assert field is ignored	The API must return 201/200 — but the protected field must not be persisted.
4. Verify the DB value	Check the created record — the role (or other protected field) must remain at its safe default.

// ✅ Mass assignment test — Supertest
it('ignores role field in registration body', async () => {
  const res = await request(app)
    .post('/v1/users')
    .send({ name: 'Eve', email: 'eve@example.com', password: 'Pass1234!', role: 'admin' });

  expect(res.status).toBe(201);
  expect(res.body.data.role).toBe('customer');  // Must be ignored — default applied
  const dbUser = await db.users.findById(res.body.data.id);
  expect(dbUser.role).toBe('customer');  // Verify DB state directly
});

10.5  Quick Security Test Payload Reference
Attack Type	Sample Payloads to Try
SQL Injection	' OR 1=1--  ·  ' OR '1'='1  ·  1; DROP TABLE users--  ·  ' UNION SELECT * FROM users--
NoSQL Injection	{ "$gt": "" }  ·  { "$where": "1==1" }  ·  { "$regex": ".*" }
Oversized Payload	Send a string of 100,000+ characters in any text field. Send a JSON array with 10,000 items.
Special Characters	<script>alert(1)</script>  ·  ../../../etc/passwd  ·  %00  ·  null  ·  undefined
QA TIP:  Start with IDOR and mass assignment — they are the most common API vulnerabilities and the easiest to test manually. Spend 15 minutes per endpoint on these two checks before any release.

11 · Integration, Contract & E2E Testing
11.1  Integration Testing
•	Use Testcontainers or Docker Compose for real isolated DB — never mock the DB in integration tests
•	Seed data in beforeEach with factory builders — assert on DB state after mutations
•	Run migrations fresh before each test run — test against production-identical schema

// ✅ Integration test with DB state assertion
it('persists order and deducts inventory', async () => {
  await seedInventory({ productId: 'P1', stock: 5 });

  await orderService.createOrder({ productId: 'P1', qty: 2 }, testUser);

  const inv = await db.inventory.findOne({ productId: 'P1' });
  expect(inv.stock).toBe(3);  // DB state verified directly
});

11.2  Golden Path E2E Tests — Lifecycle Management
Golden paths are not just tests — they are living documents of your most critical user journeys. Each one has a defined owner, run frequency, and mapping to features.

ID	Journey	Owner	Run Frequency	Maps to Features
GP-01	Register → verify email → first login	QA	Every staging deploy	Auth, Email, Onboarding
GP-02	Browse → add to cart → guest checkout → confirm	QA	Every staging deploy	Catalogue, Cart, Checkout, Payments
GP-03	Logged-in user → checkout → saved card	QA	Every staging deploy	Auth, Checkout, Payments
GP-04	Admin creates product → publish → visible on store	QA	Every staging deploy	Admin, Catalogue, CMS
GP-05	Password reset full flow	QA	Every release	Auth, Email, User management

DEPLOYMENT RULE:  Any Golden Path failure in staging blocks deployment. Golden Path maintenance is a QA responsibility — review after every major feature change.

11.3  Playwright E2E Example
test('GP-03: authenticated checkout', async ({ page }) => {
  // API-driven setup — NOT UI form filling
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
  // ✅ asserting final stable state, not intermediate loading state
});

12 · Performance Testing — All Levels
12.1  Level 1 — Basic Performance Check (Postman / Newman)
Start here. No infra setup required. Validates that response times are within acceptable limits before every release.

// ✅ Postman — basic response time assertion
pm.test('Responds within 500ms', () => {
  pm.expect(pm.response.responseTime).to.be.below(500);
});

// ✅ Newman CLI — run collection against staging (in CI)
newman run postman_collection.json \
  --environment staging.env.json \
  --reporters cli,junit \
  --reporter-junit-export results.xml

12.2  Level 2 — Simple Load Test Before Release (k6)
A 5-minute k6 smoke test before every major release. No complex setup required.

// ✅ k6 smoke test — simple, fast, pre-release gate
import http  from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus:      10,        // 10 virtual users
  duration: '1m',      // Run for 1 minute
  thresholds: {
    http_req_duration: ['p(95)<500'],  // p95 must be under 500ms
    http_req_failed:   ['rate<0.01'],  // Error rate under 1%
  },
};

export default function () {
  const res = http.get(`${__ENV.BASE_URL}/v1/orders`, {
    headers: { Authorization: `Bearer ${__ENV.TEST_TOKEN}` },
  });
  check(res, { 'status 200': (r) => r.status === 200 });
  sleep(1);
}

12.3  Level 3 — Per-Endpoint Budget Enforcement
// ✅ Performance budget declared per endpoint
export const orderBudget = { p95ResponseMs: 300, maxDbQueries: 3 };

// Integration test enforces query budget
it('resolves within query budget', async () => {
  const { queryCount } = await withQueryCounter(() => orderService.listByUser(userId));
  expect(queryCount).toBeLessThanOrEqual(orderBudget.maxDbQueries);
});

12.4  Response Time SLA Targets
Endpoint Type	p95 Target
Read — simple by ID	< 100ms
Read — filtered list	< 300ms
Write — simple create/update	< 500ms
Auth endpoint	< 300ms
Search / full-text	< 800ms
Report / aggregate	< 3,000ms

13 · Observability Testing
Observability signals — logs, metrics, traces — are part of the production contract. They must be tested with the same rigour as functional behaviour.

13.1  What to Assert
Signal	Mandatory Fields to Assert
Request log	traceId · userId (if authenticated) · method · endpoint · statusCode · latencyMs
Error log (4xx/5xx)	traceId · errorCode · errorMessage (not stack trace) · userId
Business event log	eventType · entityId · actorId · timestamp

// ✅ Assert traceId present in error response (Level 1+)
it('includes traceId in every error response', async () => {
  const res = await request(app).get('/v1/orders/nonexistent').set('Authorization', bearer);
  expect(res.status).toBe(404);
  expect(res.body.error.traceId).toMatch(/^[0-9a-f-]{36}$/);
});

// ✅ Level 3 — Assert structured log fields
it('emits error log with errorCode and userId', async () => {
  const logSpy = jest.spyOn(logger, 'error');
  await request(app).get('/v1/orders/x').set('Authorization', bearer);
  expect(logSpy.mock.calls[0][0]).toMatchObject({
    errorCode: 'ORDER_NOT_FOUND',
    traceId:   expect.stringMatching(/^[0-9a-f-]{36}$/),
    userId:    expect.any(String),
  });
});

14 · Test Data Management
14.1  Data Builders vs Object Mothers
Pattern	When to Use
Data Builder (per-test fine-grained control)	Unit and integration tests. Each test overrides only what it cares about.  buildUser({ role: 'admin' }) buildCoupon({ expiresAt: yesterday })
Object Mother (pre-built named scenarios)	E2E and API tests where multi-entity setup is needed. Hides complexity behind a descriptive name.  Scenarios.checkoutReady() Scenarios.adminModeration()

// ✅ Data Builder — minimal, overridable defaults
const buildUser = (overrides = {}) => ({
  id:        `user-${crypto.randomUUID()}`,
  email:     `test+${Date.now()}@example.com`,
  role:      'customer',
  createdAt: new Date('2024-01-15T10:00:00Z'),  // Fixed date — never new Date()
  ...overrides,
});

// ✅ Object Mother — named scenario for E2E reuse
export const Scenarios = {
  checkoutReady: async () => {
    const user    = await db.users.create(buildUser());
    const product = await db.products.create(buildProduct({ stock: 10 }));
    return { user, product };
  },
};

15 · Mocking & Stubbing Standards
15.1  Mock Decision Matrix
Test Layer	What to Mock
Unit tests	Mock: repository, external HTTP, email, queue. Real: your own business logic.
Integration tests	Mock: nothing — real DB container is the entire point of integration testing.
API tests	Mock: external downstream services (via MSW/nock). Real: DB, auth, business logic.
E2E tests	Mock: third-party providers (Stripe, SendGrid) via page.route(). Real: your app stack.
GOLDEN RULE:  Mock at system boundary — external DB, HTTP, queue, clock. Never mock your own service classes or domain logic.

// ✅ MSW network mock — shared across tests
export const server = setupServer(
  rest.post('https://api.stripe.com/v1/charges', (req, res, ctx) =>
    res(ctx.json({ id: 'ch_test_123', status: 'succeeded' }))),
);
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(()  => server.close());

16 · Mutation Testing (Level 3 — Optional)
LEVEL 3 ONLY:  Mutation testing with Stryker is an advanced practice for teams at Level 3 maturity. Do not attempt to implement this at Level 1 or 2 — establish a stable test suite first, then use Stryker to validate its quality.

Coverage metrics tell you which lines ran. Mutation testing tells you whether your tests would detect a real change. Stryker introduces small code mutations and checks if any test fails.

Tier	Target Mutation Score
Tier 0 — service layer	≥ 85% mutation score — run weekly or on STRICT PRs
Tier 1 — service layer	≥ 75% mutation score — run before major releases
Tier 2	Optional — run quarterly as a health check

// stryker.config.json — minimal setup
{
  "testRunner": "jest",
  "mutate":     ["src/features/**/services/*.ts"],
  "thresholds": { "high": 85, "low": 75, "break": 70 }
}

17 · Test Smell Detection
17.1  Test Smell Catalogue
Smell	How to Detect	Fix
Over-Mocking	Mocks > 3 dependencies. jest.mock() on own code.	Mock only external boundary. Refactor for smaller units.
Assertion Roulette	Multiple unrelated assertions in one test.	One test per behaviour. Split into individual it() blocks.
Mystery Guest	Test depends on global state or DB rows set elsewhere.	Seed in beforeEach. Make every test self-contained.
Brittle Selector	E2E uses CSS class, nth-child, or text content.	Replace with getByRole, getByLabel, or data-testid.
Dead Test	Skipped test with no ticket reference.	Add JIRA ref or delete. A permanently skipped test is noise.
False Positive	Test passes even when the tested code is deleted.	Check with Stryker. Add a specific, falsifiable assertion.
Slow Test	Unit test > 100ms. Integration test > 2s.	Fake timers, mock heavy I/O, or move to correct test layer.
Iceberg Test	Name says 'create order' but quietly tests email, inventory, and payment.	Each test covers one behaviour. Name it precisely.

18 · CI/CD Quality Gates
18.1  Fail-Fast Pipeline
Stage	Gates — Pipeline Stops if Any Gate Fails
1. Static Analysis	Lint · TypeScript check · Secret scan · SAST (Semgrep) · Runtime check
2. Unit Tests	All pass · Coverage threshold per tier · Suite runtime < 30s
3. Contract Validation	OpenAPI schema diff · Mode declaration · Breaking change detection
4. Integration Tests	Migrations apply cleanly · All pass · DB constraints verified · Runtime < 3m
5. API / Contract Tests	Status code matrix · jest-openapi schema · Auth matrix · Runtime < 60s
6. Security Scan	npm audit no critical CVEs · Semgrep no high severity
7. Perf Smoke (staging)	p95 within declared budget · Query count within budget
8. E2E Golden Paths	All Golden Path tests pass — any failure blocks deployment

18.2  Flaky Test Policy
Flaky Test Severity	SLA
P1 — Blocking release or deployment	Quarantine within 4h · Root cause and fix within 1 day
P2 — Failing in 20–50% of runs	Quarantine within 24h · Fix within current sprint
P3 — Failing in < 20% of runs	Quarantine within 24h · Fix within next 2 sprints
Team flaky rate > 3%	Halt feature work. Fix test suite before new features are started.
RULE:  Retries mask bugs — they do not fix them. Playwright retries (max 2) exist for genuine infrastructure flakiness. Application-level flakiness must be root-caused and fixed.

19 · PR Review Standards
Reviewing whether tests exist is not enough. Every reviewer must assess whether tests are meaningful.

	Checklist Item	When
☐	Do the tests fail if the implementation is removed or reverted?	Review
☐	Are edge cases and boundary values covered — not just the happy path?	Review
☐	Is each test asserting on behaviour described in its name?	Review
☐	Are assertions specific and falsifiable — not expect(true) or toBeDefined()?	Review
☐	Are mocks at system boundaries — not on own business logic?	Review
☐	Is the test self-contained — no shared state or order dependency?	Review
☐	Does the test cover the error path, not just success?	Review
☐	Are acceptance criteria (GWT) from the story mapped to a test case?	Review
☐	Are test names in should X when Y format?	Review
☐	Are any test smells from Section 17 present?	Review
REVIEWER AUTHORITY:  A reviewer can request better test quality before approving — even if CI gates pass. Test quality is a merge criterion, not just a metric.

20 · Defect Classification & Management
Severity	Definition	Response SLA	Example
P1 — Critical	Production down or data corrupted	Hotfix within 2h	Payments broken; user data overwritten
P2 — High	Core feature broken or major security vulnerability	Fix in current sprint	Login fails 20%; IDOR vulnerability found
P3 — Medium	Degraded but workaround exists	Next sprint	Pagination wrong order; misleading error message
P4 — Low	Minor UX or cosmetic issue	Backlog	Placeholder typo; mobile button slightly misaligned

TEST-FIRST FIX:  Before fixing any bug: (1) Write a failing test that reproduces it. (2) Confirm test fails. (3) Fix the code. (4) Confirm test passes. This prevents silent regression.

21 · Exception Handling Framework
Teams will sometimes have legitimate reasons to deviate temporarily from this SOP.

8.	Engineer documents: which rule, the justification, the risk, and the mitigation
9.	Tech lead reviews and approves — all exceptions require explicit sign-off
10.	Exception recorded in PR description and in the team's exception register
11.	Time-bound waiver: max 1 sprint for coverage gaps, max 2 weeks for process exceptions
12.	At expiry: remediate or resubmit. Expired exceptions escalate to engineering leadership

/**
 * EXCEPTION REQUEST
 * Rule:         Section 4 — Tier 1 service layer 80–85% branch coverage
 * Justification: Legacy PaymentProcessor needs full refactor (JIRA-5501, Q3)
 * Risk:          Medium — integration tests cover happy path
 * Mitigation:   5 new unit tests added for 3 highest-risk branches
 * Waiver Expiry: 2026-05-01
 * Approved by:   @techLead (2026-04-10)
 */

22 · SOP Governance
Responsibility	Owner & Cadence
SOP primary owner	Staff Engineer / Head of Quality — reviews and approves all changes
Quarterly full review	SOP owner + one tech lead per squad — full relevance check
Change proposals	Any engineer via PR against SOP repository
Change approval	SOP owner approval + one senior engineer review
Emergency updates	SOP owner can publish urgent patches within 48h — flagged as hotfix

Version	Summary
v1.0 — Feb 2026	Initial release. Unit, integration, API, E2E, performance, security, accessibility standards.
v2.0 — Apr 2026	ISTQB/Agile alignment. Service tier system. Observability. PR review. Test smells. Exception process. Mutation testing. SOP governance.
v3.0 — Apr 2026	Adoption Level system (L1/L2/L3). Manual & exploratory testing. Test case management + traceability. QA-friendly security guide. Beginner API test examples. RACI matrix. Performance fallback for small teams. Mutation testing marked Level 3. Golden path lifecycle management. 1-page quick start guide.

23 · Full Decision Checklist

	Checklist Item	When
☐	Unit tests for every new function, hook, or service method	Mandatory
☐	Every conditional branch has a test — happy path alone is not enough	Mandatory
☐	Tests follow AAA with explicit Arrange/Act/Assert separation	Mandatory
☐	Test names: should [behaviour] when [condition]	Mandatory
☐	No hardcoded sleep() or setTimeout in any test file	Mandatory
☐	Each test seeds own data in beforeEach via factory builder	Mandatory
☐	No PII, real emails, or production data in any fixture	Mandatory
☐	jest-axe runs on every component render test	Mandatory
☐	GWT acceptance criteria mapped to at least one test case	Mandatory
☐	Service tier declared — coverage threshold confirmed	Mandatory
☐	Any SOP deviation: exception request documented with tech lead approval	Mandatory
☐	Full auth matrix: missing, invalid, expired, wrong role, cross-user, correct	Strict
☐	Full status code matrix for all STRICT endpoints	Strict
☐	jest-openapi contract assertion on every happy-path API test	Strict
☐	Integration tests assert on DB state after mutations	Strict
☐	traceId present in all error responses — verified in API test	Strict
☐	performanceBudget declared and DB query count assertion passing	Strict
☐	Security: IDOR, mass assignment, injection, oversized payload tested	Strict
☐	Test smells checked — no Over-Mocking, Dead Tests, or Brittle Selectors	Mandatory
☐	E2E test data created via API — not via UI form interactions	E2E
☐	All Golden Path tests pass in staging before release	E2E
☐	k6 smoke test passes p95 threshold on staging deploy	Perf
☐	Manual exploratory charter completed for new features before release	Mandatory
☐	Test case in Jira/Xray linked to story and automation ID	L2

Appendix · Key Terms
Term	Definition
AAA	Arrange–Act–Assert — the mandatory structure for every test case
Adoption Level	L1/L2/L3 maturity classification determining which SOP sections apply to a team right now
ISTQB	International Software Testing Qualifications Board — globally recognised testing standards
STLC	Software Testing Life Cycle — testing phases within the Agile SDLC
GWT / BDD	Given–When–Then — acceptance criteria format that maps directly to automated test cases
SBET	Session-Based Exploratory Testing — time-boxed, chartered, structured exploratory testing
Testing Quadrants	Brian Marick's model: Q1/Q2 (support team), Q3/Q4 (critique product)
Service Tier	Risk classification (T0/T1/T2) determining coverage depth and test requirements per service
Golden Path	Critical user journeys that must always pass — GP failure blocks any deployment
Object Mother	Pre-built named test scenario combining multiple entities for E2E/API reuse
Data Builder	Minimal factory function with safe defaults and per-test overrides
Mutation Testing	Practice using Stryker to introduce code mutations to validate assertion strength
Test Smell	Pattern indicating low quality — e.g., Over-Mocking, Dead Tests, Brittle Selectors
Mutation Score	Killed mutations ÷ total mutations × 100 — measures assertion quality
IDOR	Insecure Direct Object Reference — accessing another user's resource via a guessable ID
RACI	Responsible · Accountable · Consulted · Informed — responsibility assignment matrix
MSW	Mock Service Worker — intercepts HTTP at network level for realistic test mocking
Testcontainers	Library that spins up disposable Docker containers for integration testing
Contract Test	Verifies API response matches a declared OpenAPI specification
p95	95th percentile response time — 95% of requests complete within this duration
MTTD	Mean Time to Detect — average time from bug introduction to discovery
MTTR	Mean Time to Resolve — average time from detection to production fix
Traceability	Linkage: requirement → user story → test case → automation → defect

— End of Document  ·  Testing SOP v3.0  ·  April 2026 —
