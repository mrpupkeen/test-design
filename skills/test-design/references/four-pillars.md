# The Four Pillars of a Good Test

A test's value = regression protection × refactoring resistance × feedback
speed × maintainability. Multiplication, not addition: score zero on any one
and the test is worth zero. No test maxes all four — fast feedback and
regression protection trade off against each other — but resistance to
refactoring is binary and non-negotiable: a test either can produce false
positives or it can't.

## 1. Protection against regressions

How likely the test is to catch a real bug. Grows with the amount of real
logic executed (yours and the libraries you use), its complexity, and its
domain significance.

Failing example: a test over an auto-property / trivial getter. It executes
no logic, can catch no bug, scores ~0, and therefore has zero value no matter
how fast or readable it is. This is the formal reason trivial code is not
tested.

## 2. Resistance to refactoring

Whether the test survives any change that keeps observable behavior intact.
A failure on an unchanged behavior is a false positive, and false positives
destroy a suite: people learn to ignore red builds, lose trust, and stop
refactoring. Early in a project false positives feel harmless; as the project
grows they become as damaging as missed bugs.

False positives come from one source: coupling to implementation details.
The fix is always the same — verify the end result the user/client cares
about, not the steps taken to get there.

Failing example: a test asserting `repository.save()` was called with a
specific object. Rename the method, batch the save, or switch persistence
strategy and the test fails while the behavior (order is persisted) is
intact. Rewrite: run the operation, then read the state back and assert on it.

## 3. Fast feedback

Milliseconds for unit tests. A slow test runs rarely, and a test that runs
rarely protects nothing. Slowness in a "unit" test is usually a smell of an
out-of-process dependency that belongs in the integration suite instead.

## 4. Maintainability

Two components: how hard the test is to read (size — keep arrange short via
factory methods / Object Mother) and how hard it is to run (every
out-of-process dependency is operational overhead).

## How the trade-off resolves per test type

- End-to-end: max regression protection and refactoring resistance, terrible
  speed — keep very few.
- Trivial tests: fast and resistant but protect nothing — keep none.
- Brittle tests: fast and may catch bugs but false-positive on refactors —
  keep none.
- Everything you keep sits between these, with refactoring resistance held
  at maximum and the real dial being speed vs. protection: that dial is the
  unit/integration split.
