
STANDARD OPERATING PROCEDURE
Frontend Development
Testability & Quality Standards

Lean · Strict · Balanced
Version 1.0  ·  March 2026
 
Purpose & Scope
This SOP defines how frontend developers at all experience levels should design, build, and structure code so that it is testable, maintainable, and scalable — without sacrificing development speed. It establishes a dual-mode system (Lean vs Strict) that maps to feature complexity and business criticality.

GOAL:  Enable teams to ship fast, test reliably, scale confidently, and never over-engineer.

Who This Applies To
•	All frontend engineers (junior, mid, senior)
•	QA engineers writing or reviewing automation tests
•	Tech leads making architecture decisions
•	Code reviewers enforcing standards

1 · Mode Definitions — Lean vs Strict
Every feature falls into one of two operational modes. Choose the mode before you write a single line of code.

Mode Decision Matrix
Criterion	→ Use Strict Mode When...
Feature complexity	Multiple states, branching logic, or 3+ async operations
Reusability	Component used in 3+ places or across teams
Business impact	Revenue, auth, payments, checkout, onboarding, or compliance
User-facing criticality	Any failure causes immediate UX breakdown or data loss
Test coverage requirement	QA automation coverage is mandatory
Codebase longevity	Expected to live beyond 6 months and receive regular changes

Mode Summary
⚡  LEAN  —  Fast & Simple	🏗️  STRICT  —  Scalable & Testable
For small, low-risk, single-use features
Prototypes, quick iterations, internal tools
Components that are unlikely to change
Features with minimal state and no branching	For complex, reusable, or critical features
Public-facing flows: auth, checkout, forms
Shared component library pieces
Any feature under automated test coverage

RULE:  When in doubt, start Lean but write code as if it will become Strict. Never refactor to testability — build it in from day one.

2 · Component Design Guidelines
2.1  Lean Mode — Component Rules
•	Co-locate logic and UI in a single file if the component is under 120 lines
•	Accept props directly — no intermediate interfaces unless type safety demands it
•	Inline simple conditional rendering — no sub-component splits needed
•	No custom hooks unless logic exceeds 20 lines or repeats in 2+ components

// ✅ Lean: acceptable co-location
export const StatusBadge = ({ status }) => (
  <span className={`badge badge--${status}`}>{status}</span>
);

2.2  Strict Mode — Component Rules
•	Separate concerns: UI component, business logic hook, and data layer
•	Use a Container / Presenter pattern for complex, data-driven components
•	Keep presentational components pure — no direct API calls or global state access
•	Extract custom hooks (useFeatureName) for any reusable stateful logic
•	Type all public props with TypeScript interfaces, not inline types

// ✅ Strict: separated concerns
// hooks/useUserProfile.ts  — logic only
// components/UserProfile/UserProfile.tsx  — UI only
// components/UserProfile/UserProfile.test.tsx
// components/UserProfile/index.ts  — public export

2.3  When to Split UI from Logic (Universal Rule)
Signal	Action
Component exceeds 80 lines	Extract logic into a custom hook
Component makes direct API calls	Move to a service + hook layer
Same logic used in 2+ components	Extract to shared hook immediately
Component has 3+ distinct visual states	Create separate state-driven sub-components
Unit test requires mocking React state directly	Refactor: logic belongs in a hook, not the component

3 · Testability Standards
3.1  Selector Strategy
QA automation tests are only as stable as the selectors they use. Follow this priority order:

Priority	When to Use
1. Semantic HTML roles (button, input, heading)	Preferred for interactive elements — always attempt this first
2. aria-label / aria-labelledby attributes	When semantic role alone is ambiguous
3. data-testid attributes	Use only when semantic + aria options are unavailable
4. CSS class names or IDs	NEVER use for test selectors — fragile and breaks with refactors

// ✅ Preferred — semantic selector
screen.getByRole('button', { name: /submit order/i })

// ✅ Acceptable — aria-label
screen.getByLabelText('Email address')

// ✅ Fallback — data-testid (non-critical elements only)
<div data-testid="order-summary-panel">

// ❌ Never — class selector in tests
document.querySelector('.btn-primary')  // FORBIDDEN

3.2  data-testid Rules
•	Use kebab-case: data-testid="checkout-submit-button"
•	Include the component name as a prefix: data-testid="cart-item-remove"
•	Never use auto-generated IDs (Math.random, Date.now) as test identifiers
•	For dynamic lists, include a stable key: data-testid={`product-card-${product.id}`}
•	Strip data-testid in production builds using Babel or compiler transforms

3.3  Isolation Design Rules
•	All business logic in custom hooks must be independently unit testable
•	API calls must be injectable or mockable — never import fetch/axios directly into components
•	Global state slices must be testable in isolation with a minimal store mock
•	Side effects (timers, websockets, subscriptions) must be encapsulated and cleanup-able

PRINCIPLE:  If you cannot test a unit in complete isolation, its dependencies are not properly abstracted. Refactor the abstraction, not the test.

4 · State Management
4.1  State Layer Decision
State Type	Recommended Solution
UI toggle / local component state	useState (both modes)
Shared state across 2-3 siblings	Lift state up to common parent (Lean)
Cross-cutting UI state (modals, toasts)	useContext + custom provider (Lean/Strict)
Server cache, async data fetching	React Query / SWR (Strict)
Complex global state with side effects	Redux Toolkit / Zustand (Strict only)
URL-driven state (filters, pages)	URLSearchParams / router state (both modes)

4.2  Testability Requirements per State Layer
•	useState: test through component render — assert UI changes not state values directly
•	useContext: wrap in a test provider with injected mock values
•	React Query: mock the query client or use MSW for network-level mocking
•	Redux/Zustand: test reducers as pure functions; test components with a real store instance

4.3  Anti-Patterns to Avoid
•	Do not derive state that can be computed from existing state (causes double-truth bugs)
•	Do not store server response objects directly in state — map to a domain model
•	Do not bypass the state layer for component-to-component communication via DOM refs
•	Never mutate state directly — always return new objects/arrays

5 · API & Data Layer
5.1  When to Abstract APIs
⚡  LEAN  —  Fast & Simple	🏗️  STRICT  —  Scalable & Testable
Direct fetch call inside component or hook is acceptable
No dedicated service file required
Inline error handling with try/catch is fine
Only abstract if the same endpoint is called in 2+ places	All API calls live in /services/ or /api/ layer
Each service returns typed domain objects (not raw API shapes)
Services are dependency-injectable for test mocking
Use adapter pattern to decouple API schema from UI models

5.2  Mocking Strategy
Test Environment	Recommended Approach
Unit tests (hooks/components)	jest.fn() or vi.fn() to mock service modules directly
Integration tests	Mock Service Worker (MSW) — intercept at network level
E2E tests (Playwright/Cypress)	MSW or dedicated test API server with seeded data
Storybook development	MSW or static fixture files per story

// ✅ Strict — injectable service pattern
// services/orderService.ts
export const orderService = {
  getOrder: (id: string) => fetch(`/api/orders/${id}`).then(r => r.json()),
};

// hooks/useOrder.ts — receives service as default param (testable)
export const useOrder = (id, service = orderService) => {
  return useQuery(['order', id], () => service.getOrder(id));
};

6 · Error, Loading & Edge Cases
6.1  Minimum Requirements (Both Modes)
•	Every async operation must handle at least 3 states: loading, success, and error
•	Loading state must be visually represented — never leave a blank UI
•	Error state must be user-legible — no raw error objects displayed
•	Empty state must be handled explicitly — never assume data will always exist

6.2  Lean vs Strict Handling
⚡  LEAN  —  Fast & Simple	🏗️  STRICT  —  Scalable & Testable
Inline conditional rendering for error/loading
Single generic error message is acceptable
console.error is an acceptable temporary logger	Dedicated ErrorBoundary wrapping critical sections
Typed, user-actionable error messages
Integrated error tracking (Sentry, Datadog)
Empty state: simple 'No data found' text
Retry logic optional for non-critical features	Empty state: full illustrated empty state component
Retry logic mandatory with exponential back-off

6.3  Testability Requirement
•	Every loading, error, and empty state must have at least one corresponding automated test
•	Use MSW to simulate 4xx and 5xx responses in integration tests
•	Test error boundaries with deliberate component throws in test environments

MANDATORY:  No feature can be marked 'done' unless its loading, error, and empty states are both implemented and have test coverage.

7 · Automation Stability Rules (Mandatory)
These rules are non-negotiable. Violating them causes flaky tests that erode team trust in automation.

7.1  Deterministic UI Behavior
•	Never use Math.random(), Date.now(), or new Date() directly in rendered output
•	Use a deterministic ID generator (e.g. nanoid with seeded test config) or sequential keys
•	CSS animations that affect element visibility must be disableable in test environments
•	Use a CSS class on <html> (no-animation) to suppress transitions in test runs

7.2  Selector Stability
•	data-testid values are treated as public contracts — never rename without a migration plan
•	Warn in PRs if a data-testid is removed or renamed without a corresponding test update
•	Do not use index-based selectors (nth-child, array index) for dynamic lists
•	Ensure test IDs are unique within a page — use scoped prefixes for repeated patterns

7.3  Async Handling in Tests
•	Never use arbitrary sleep() or setTimeout delays in tests — always wait on DOM state
•	Use waitFor, findBy*, or await act() patterns in React Testing Library
•	Mock timers (jest.useFakeTimers) for components with debounce or polling logic
•	Always assert on the final stable state, not intermediate loading states

7.4  Test Environment Isolation
•	Each test must set up its own data — never rely on test execution order
•	Clean up DOM, timers, and event listeners in afterEach or using testing-library cleanup
•	Use beforeEach to reset MSW handlers — avoid state bleed between tests
•	Mock dates and times consistently using a library like @sinonjs/fake-timers

FLAKY TEST POLICY:  Any test that fails intermittently with no code change must be immediately quarantined and fixed before the next release cycle. Flaky tests are bugs.

8 · Accessibility (Non-Negotiable)
Accessibility is not optional. It also directly enables better test automation through semantic selectors.

8.1  Mandatory Accessibility Rules
•	All interactive elements must be keyboard-focusable and operable
•	All images must have meaningful alt text (or alt="" for decorative images)
•	Color must not be the sole conveyor of information — always include text or icon
•	Focus must be visibly indicated — never suppress outline without a custom replacement
•	All form inputs must have associated labels (via htmlFor or aria-labelledby)
•	Dynamic content changes (alerts, toasts, live regions) must use aria-live appropriately

8.2  ARIA Usage Rules
•	Use native HTML elements before reaching for ARIA — <button> beats role="button"
•	Do not add ARIA roles to elements that already have an implicit role
•	aria-label and aria-describedby values must be human-readable and tested
•	Modal dialogs must trap focus and restore focus on close

8.3  Testing Accessibility
•	Integrate jest-axe or vitest-axe in unit tests — run on every component render test
•	Include keyboard navigation scenarios in E2E test suites
•	Run axe-core or Lighthouse CI in the deployment pipeline — fail on critical violations

// ✅ Accessibility unit test with jest-axe
it('has no accessibility violations', async () => {
  const { container } = render(<CheckoutForm />);
  expect(await axe(container)).toHaveNoViolations();
});

9 · Folder Structure
9.1  Lean Project Structure
Suitable for prototypes, single-team apps, or projects under 20 components.

src/
  components/          # All components flat
    Button.tsx
    UserCard.tsx
  pages/               # Route-level components
    Dashboard.tsx
  hooks/               # Shared hooks only
    useAuth.ts
  utils/               # Pure utility functions
  types/               # Global TypeScript types
  App.tsx

9.2  Strict Project Structure
For production apps, shared component libraries, or multi-team codebases.

src/
  features/                    # Feature-first grouping
    checkout/
      components/              # Feature-local components
        CheckoutForm/
          CheckoutForm.tsx     # Presentational only
          CheckoutForm.test.tsx
          CheckoutForm.stories.tsx
          index.ts             # Public barrel export
      hooks/                   # Feature-local hooks
        useCheckout.ts
        useCheckout.test.ts
      services/                # API abstraction layer
        checkoutService.ts
      types/                   # Feature-local types
        checkout.types.ts
      index.ts                 # Feature public API
  shared/                      # Cross-feature components
    components/
    hooks/
    services/
    utils/
    types/
  pages/                       # Route-level composition only
  store/                       # Global state (if needed)
  test/                        # MSW handlers, fixtures, test utils

BARREL FILES:  Always export through index.ts — this creates a stable public API for each feature and prevents deep import paths that break with refactors.

10 · Documentation Standards
10.1  What to Document and When
Item	Required Documentation
Simple utility function (<10 lines)	Descriptive function name only — no comments
Complex utility / algorithm	JSDoc comment with @param, @returns, and a usage example
Public component props interface	JSDoc on each non-obvious prop
Custom hook	Inline comment explaining the problem it solves, not how it works
API service method	Parameter types, return type, and thrown error contract
State shape / store slice	Comment describing the purpose of each non-obvious field
Workaround or hack	MANDATORY: // TODO: [reason] comment with ticket reference

10.2  Documentation Anti-Patterns
•	Never document what the code does — document why it does it
•	Never leave TODO comments without a ticket reference (TODO: JIRA-1234)
•	Never write comments that are out of date with the actual code — delete stale comments
•	Avoid inline comments inside render functions — if it needs a comment, extract it

// ❌ Bad — documents 'what', not 'why'
// Loop through items and add to total
const total = items.reduce((acc, item) => acc + item.price, 0);

// ✅ Good — documents 'why' (only if non-obvious)
// Prices are stored in cents to avoid floating-point precision errors
const totalCents = items.reduce((acc, item) => acc + item.priceInCents, 0);

11 · Quick Decision Checklist
Run through this checklist before writing any new component or feature.

	Checklist Item	Mode
☐	Have I identified whether this is Lean or Strict based on the decision matrix?	Both
☐	Is the component separated into UI and logic if it's complex or reusable?	Strict
☐	Are interactive elements accessible via keyboard and have proper ARIA roles?	Both
☐	Does every async operation handle loading, error, and empty states?	Both
☐	Have I avoided CSS class names as test selectors?	Both
☐	Are data-testid values stable, meaningful, and prefixed?	Strict
☐	Is my API call abstracted or at minimum mockable without module rewiring?	Strict
☐	Have I avoided Math.random() or Date.now() in rendered output?	Both
☐	Is there a test for each loading, error, and empty state?	Both
☐	Are animations and transitions suppressible in the test environment?	Strict
☐	Are any workarounds documented with a ticket reference?	Both
☐	Have I run jest-axe or equivalent on my component render tests?	Both

12 · Do's and Don'ts
12.1  Component Design
⚡  LEAN  —  Fast & Simple	🏗️  STRICT  —  Scalable & Testable
✅  DO
Export pure presentational components
Use descriptive, intention-revealing names
Keep components under 150 lines; split if larger	❌  DON'T
Import API clients directly inside render functions
Use generic names: Item, Card, Box, Wrapper
Put all app logic in a single god component

12.2  Selectors & Test IDs
⚡  LEAN  —  Fast & Simple	🏗️  STRICT  —  Scalable & Testable
✅  DO
getByRole('button', { name: /submit/i })
getByLabelText('Email')
data-testid="cart-total-price" (stable, scoped)	❌  DON'T
querySelector('.btn.primary')
data-testid="div-3" (generic, meaningless)
Use text content as primary selector for i18n apps

12.3  State & Data
⚡  LEAN  —  Fast & Simple	🏗️  STRICT  —  Scalable & Testable
✅  DO
Map API responses to a domain model immediately
Use React Query for server state
Keep URL state in the router, not useState	❌  DON'T
Store raw API objects in component state
Use useEffect chains to synchronize state
Share state via DOM refs between components

12.4  Automation & Async
⚡  LEAN  —  Fast & Simple	🏗️  STRICT  —  Scalable & Testable
✅  DO
await screen.findByText('Success')
Use MSW to simulate error responses
Mock Date and Math.random in test setup	❌  DON'T
await new Promise(r => setTimeout(r, 2000))
Rely on test execution order for state setup
Leave animated elements enabled during test runs

12.5  Accessibility
⚡  LEAN  —  Fast & Simple	🏗️  STRICT  —  Scalable & Testable
✅  DO
Use <button> for actions, <a> for navigation
Associate labels with inputs via htmlFor
Support focus-visible for keyboard users	❌  DON'T
Use <div onClick> for interactive elements
Remove :focus outline without a replacement
Rely on color alone to convey status

Appendix · Key Terms
Term	Definition
Lean Mode	Development approach prioritising speed and minimal structure for low-risk features
Strict Mode	Structured, scalable approach for complex, critical, or widely-reused features
Container/Presenter	Pattern separating data-fetching logic (container) from rendering (presenter)
data-testid	HTML attribute used as a stable, semantics-independent hook for automation selectors
MSW	Mock Service Worker — intercepts network requests at the Service Worker level for testing
ErrorBoundary	React component that catches rendering errors in its subtree
Barrel file	An index.ts that re-exports multiple modules to create a stable public API for a folder
Domain Model	An application-specific data structure that maps from an API schema to the UI's needs

— End of Document —
