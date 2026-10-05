---
name: test-design
description: Use when writing, reviewing, or refactoring tests (unit, integration, e2e), or deciding what to test and what to mock. Khorikov-based test design — four pillars, code classification, strict mocking policy. Complements TDD skills, which govern when tests are written.
---

# Test Design (Khorikov)

Goal of testing: sustainable growth of the project. Every test has a cost; both
production code and test code are liabilities. A bad test is worse than no test.
A test's value is the PRODUCT of four pillar scores — zero on one pillar means
zero value (see `references/four-pillars.md`).

The TDD skill decides WHEN you write a test. This skill decides WHAT it tests
and HOW. Apply the classification below before the first RED test is written.

## Step 1 — Classify the code (always, before writing any test)

Two axes: complexity/domain-significance, and number of collaborators.

1. **Domain model & algorithms** (complex, few collaborators)
   → unit test, exhaustively. Highest return on effort.
2. **Controllers / orchestration** (simple, many collaborators)
   → integration test only: one happy path + edge cases unit tests can't reach.
3. **Trivial code** (simple, few collaborators: getters, DTOs, mappers, config)
   → do NOT test. State this explicitly and move on.
4. **Overcomplicated code** (complex AND many collaborators)
   → STOP. Do not test it as-is. Refactor first with the Humble Object pattern:
   extract decisions into pure domain code, leave a thin humble controller,
   then apply 1 and 2. See `references/code-classification.md`.

Rule of thumb: code can be deep (complex) or wide (many collaborators), never both.

## Step 2 — Non-negotiable design rules

- **Unit = unit of behavior, not a class** (classical school). A unit test may
  touch several of your own classes; that's fine. Isolate tests from each
  other, not classes from each other.
- **Test observable behavior through the public API.** Never test private
  methods. If a private method is too complex to reach through the public API,
  that's a missing abstraction — extract a class, don't widen access.
- **No mocks in unit tests. Ever.** Mocks are for unmanaged out-of-process
  dependencies only (external APIs, message bus, SMTP), and only controllers
  touch those — so mocks appear only in integration tests. Your own database
  is a managed dependency: use the real one. Full rules and the
  managed/unmanaged table: `references/mocking.md`.
- **Never assert interactions with stubs** (calls that fetch input data). An
  incoming call is an implementation detail, not an outcome. Over-specification
  is the #1 cause of fragile tests.
- **One Arrange-Act-Assert per test, one act line.** Multiple acts = this is an
  e2e test or it must be split. No if/for statements inside tests.
- **Prefer output-based assertions** (return values) over state-based, over
  communication-based. If output-based is impossible, push the logic toward
  pure functions (functional core, imperative shell) until it is.
- **Name tests as plain-language facts** a domain expert would understand,
  words separated by underscores. Never include the method name under test;
  never use a rigid `Method_State_Outcome` template.
  Good: `delivery_with_a_past_date_is_invalid`.
- **Assertions use their own hardcoded expected values.** Never recompute the
  expected result with production code or production constants — that's a
  tautology test (it leaks domain knowledge and verifies nothing).
- **Inject time explicitly** (prefer a plain value over a clock service).
  Never read ambient time (`DateTime.Now`, `Date.now()`) in domain code.
- **No code pollution**: no production members, hooks, or flags that exist
  only for tests. Tests get no special privileges.

## Step 3 — Four-pillars check on every test you wrote

1. **Protection against regressions** — does it execute meaningful logic?
   (Trivial code scores ~0 here; that's why it isn't tested.)
2. **Resistance to refactoring** — would it survive a rename/restructure that
   keeps behavior identical? This pillar is NON-NEGOTIABLE: a test that can
   false-positive gets rewritten against outcomes or deleted.
3. **Fast feedback** — unit tests run in milliseconds; anything slow belongs
   in the integration suite.
4. **Maintainability** — short arrange via factory methods / Object Mother;
   no walls of duplicated setup; as few out-of-process deps as possible.

## Integration & database tests

One happy path through all out-of-process dependencies, plus the edge cases
unit tests can't reach. Real database (same vendor as prod — no SQLite
stand-ins), cleanup at the START of each test, separate transaction/unit of
work per AAA section, sequential execution. Don't test repositories directly;
higher bar for testing reads than writes. Details:
`references/integration-and-db.md`.

## Red flags — stop and fix

- A mock in a unit test, or a mock of anything in-process
- Asserting that a method was called on a stub
- A test that breaks after a refactor while behavior is unchanged
- Test name mirrors a method name
- `if`/`for` inside a test; more than one act
- A single-implementation interface for an in-process dependency
  (its only purpose is to enable a mock you shouldn't have)
- Mocking a concrete class to keep part of it real (SRP violation — split it)
- Expected value computed by calling production code
- Chasing a coverage number (coverage is a NEGATIVE indicator only: low is
  bad, high proves nothing; never a target)

Anti-pattern fixes: `references/anti-patterns.md`.

## Explicit invocation modes

When the user invokes this skill directly (e.g. `/test-design:test-design`),
interpret the arguments as one of three modes:

- **review `<path>`** (or a path to existing tests): audit each test against
  the four-pillars check and the red-flags list. Report per test: verdict
  (keep / rewrite / delete), the pillar or rule violated, and the concrete
  fix. Flag resistance-to-refactoring violations as highest severity.
- **write `<path>`** (or a path to production code): run Step 1
  classification first and state the quadrant. Only then write tests of the
  type the quadrant prescribes — or decline (trivial code), or propose the
  Humble Object refactor before any test (overcomplicated code).
- **classify `<path>`**: classify the code into the quadrant and recommend
  the testing approach without writing tests.
- No arguments: ask which mode, or infer from the current task.

When a TDD skill is also active, it governs the red-green-refactor sequence;
this skill governs test design within that sequence.
