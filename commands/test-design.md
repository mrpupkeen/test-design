---
description: Apply Khorikov test-design principles to write, review, or refactor tests
argument-hint: "[review <path> | write <path> | classify <path>] or free-form"
---

Read and follow ${CLAUDE_PLUGIN_ROOT}/skills/test-design/SKILL.md (references
under ${CLAUDE_PLUGIN_ROOT}/skills/test-design/references/), then act on:
$ARGUMENTS

Mode selection:
- **review** (or a path to existing tests): audit each test against the
  four-pillars check and the red-flags list. Report per test: verdict
  (keep / rewrite / delete), the pillar or rule violated, and the concrete fix.
  Flag resistance-to-refactoring violations as highest severity.
- **write** (or a path to production code): run Step 1 classification first
  and state the quadrant. Only then write tests of the type the quadrant
  prescribes — or refuse (trivial code) or propose the Humble Object refactor
  (overcomplicated code) before writing any test.
- **classify**: classify the given code/module into the quadrant and
  recommend the testing approach, without writing tests yet.
- No arguments: ask which of the three modes, or infer from the current task.

Constraints: follow the skill's non-negotiable rules exactly (no mocks in
unit tests, never assert on stubs, behavior-based names, hardcoded expected
values). When a TDD skill is also active, it governs the red-green-refactor
sequence; this command governs test design within that sequence.
