# test-design

A Claude Code plugin encoding the test-design principles from Vladimir
Khorikov's *Unit Testing Principles, Practices, and Patterns* (Manning, 2020).
All text is an original condensed paraphrase of the book's ideas — buy the
book, it's worth it.

It complements process-oriented TDD skills (obra/superpowers
`test-driven-development`, pstack `tdd`): those govern **when** tests are
written; this governs **what** a good test looks like.

## What's inside

- `skills/test-design/` — auto-triggering skill: code-classification quadrant,
  non-negotiable design rules, four-pillars check, red flags
  - `references/four-pillars.md` — the value rubric with failure examples
  - `references/mocking.md` — managed vs unmanaged table; mocks only in
    integration tests, at the system edge, on types you own
  - `references/code-classification.md` — the quadrant + Humble Object refactor
  - `references/integration-and-db.md` — happy-path selection, real-DB rules
  - `references/anti-patterns.md` — ch. 11 anti-patterns with fixes
- `commands/test-design.md` — explicit `/test-design` trigger with
  `review | write | classify` modes

## Install

In Claude Code:

```
/plugin marketplace add mrpupkeen/test-design
/plugin install test-design@test-design-marketplace
```

Or from a shell:

```bash
claude plugin marketplace add mrpupkeen/test-design
claude plugin install test-design@test-design-marketplace
```

To try a local checkout without installing:

```bash
claude --plugin-dir /path/to/test-design
```

The skill then activates automatically on test-related work, and the command
is available as a slash command (namespaced under the plugin).

## Suggested pairing

With superpowers: let its TDD skill run the red-green-refactor loop; this
plugin's classification step decides what gets a test at all, and the
four-pillars check runs at REFACTOR and in code review.

With pstack: `tdd`'s "skip when impractical" escape hatch should route
through this plugin's classification — "impractical to test" usually means
quadrant-4 code, where the answer is a Humble Object refactor, not a skip.
