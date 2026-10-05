# Integration and Database Testing

An integration test is any test that isn't a unit test (slower, or touching
out-of-process dependencies, or not isolated from other tests). Integration
tests cover controllers; unit tests cover the domain. They buy better
regression protection at the price of speed and maintenance, so the bar for
writing one is higher.

## What to cover

- **One happy path per business scenario — the LONGEST one**, chosen so it
  exercises every out-of-process dependency the operation touches. If no
  single path does, add the fewest extra tests that complete the set.
- **Edge cases unit tests can't reach** (behavior that only exists in the
  wiring). Everything reachable by a unit test stays in unit tests.
- If an edge case's failure mode is immediate and obvious, Fail Fast
  (crash early on bad input/config) is a legitimate substitute for a test.
- Don't test repositories directly — they're covered by the overarching
  scenarios. Test reads only when complex or important; the bar for reads is
  higher than for writes (no state to corrupt).

## The test must cross all layers

Assert against the final state of the database, retrieved independently —
never by reusing the objects (or the transaction/context) from the arrange
section, or you verify the cache, not the write.

## Database rules

- Schema (tables, views, indexes, sprocs, reference data) lives in source
  control; changes ship as explicit migrations (migration-based over
  state-based delivery — data motion matters more than merge conflicts).
- Separate DB instance per developer, ideally local.
- **Same DBMS as production. No SQLite/in-memory stand-ins** — a different
  vendor means you're testing a different system.
- **Clean up at the START of each test**, not teardown: fast, consistent,
  and can't be skipped by a crash.
- One transaction / unit of work PER AAA SECTION — never shared across
  arrange/act/assert.
- Run integration tests sequentially; parallelizing them is rarely worth
  the effort.

## Keeping them short

- Arrange: Object Mother (preferred here over Test Data Builder) — factory
  methods with sensible defaults, significant values as arguments.
- Act: one-line decorator methods that hide transaction/session plumbing.
- Assert: fluent helpers that re-read state and compare.

Expected values stay hardcoded in the test — independent of production
constants and of the arrange data.
