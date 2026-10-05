# Anti-Patterns and Their Fixes

Each entry: the smell, why it fails the pillars, the fix.

## Testing private methods
Couples the test to internals → false positives on refactor.
Fix: test them indirectly through the public API's observable behavior. If
the private logic is too complex to reach that way, it's a missing
abstraction: extract it into its own class with a public API and unit test
that. (Rare legitimate exception: a private member that is itself an external
contract, e.g. a private constructor used by an ORM.)

## Exposing private state for tests
Widening visibility (or adding getters) solely so a test can inspect state
gives tests privileges production code doesn't have.
Fix: assert only through what production clients can observe. If nothing
observable changes, the test is testing an implementation detail — delete it.

## Leaking domain knowledge into tests
Computing the expected value with the same algorithm/constants as production
(`expected = PRICE * (1 - DISCOUNT)`), producing a tautology that passes even
when the algorithm is wrong.
Fix: hardcode independently derived expected values; duplicate literals from
production if needed — tests are the independent checkpoint.

## Code pollution
Production flags, hooks, `isTestMode`, test-only constructors.
Fix: invert the dependency — inject the collaborator (logger, clock, bus)
and pass a fake in tests; production code stays single-purpose.

## Mocking a concrete class to keep part of it real
Needing `mock(RealClass, { keep: someMethods })` means the class has two
responsibilities.
Fix: split it — domain logic in one class (unit tested), out-of-process
communication in another (mocked at the edge if unmanaged).

## Ambient time
Reading `Date.now()` / `DateTime.Now` (or a static clock context) inside
domain code pollutes it and makes tests nondeterministic.
Fix: inject time explicitly; prefer passing the time as a plain VALUE into
the operation over injecting a clock service.

## Multiple act sections
A test doing act → assert → act → assert verifies several behaviors and
usually signals an e2e test in disguise.
Fix: split into one test per behavior. Multiple acts are tolerable only in
integration/e2e tests where bringing an out-of-process dependency to state is
genuinely expensive.

## Assertion-free / low-signal tests
A test that merely runs code (or asserts `result != null`) scores near zero
on regression protection.
Fix: assert the actual outcome, all of it that matters — for mocks, expected
calls present AND unexpected calls absent.

## Coverage targets
A mandated number creates perverse incentives (assertion-free tests, testing
trivial code). Coverage is a negative indicator: low means untested core,
high proves nothing.
Fix: measure, don't target. Direct effort by the code-classification
quadrant instead.
