# Code Classification and the Humble Object Refactor

## The quadrant

Classify on two axes before testing anything:

- **Vertical: complexity or domain significance.** Complexity = number of
  decision points (yours + implicit ones in libraries you call). Domain
  significance = how directly it implements business rules. Either one alone
  earns the vertical.
- **Horizontal: number of collaborators** (mutable or out-of-process
  dependencies the code touches).

| | Few collaborators | Many collaborators |
| --- | --- | --- |
| **Complex / domain-significant** | Domain model & algorithms → unit test exhaustively | Overcomplicated → refactor first, never test as-is |
| **Simple** | Trivial → no tests | Controllers → integration tests, briefly |

Design invariant: code may be deep or wide, never both. Everything in the
overcomplicated quadrant must be split into a deep part and a wide part.

## Humble Object pattern (the quadrant-4 exit)

1. Identify the decisions buried in the orchestration code.
2. Extract them into domain classes / pure functions that take values in and
   return values (or domain events) out — no out-of-process access inside.
3. What remains is a humble controller: load inputs → call domain code →
   apply its decisions to dependencies. No branching beyond trivial guards.
4. Unit test the extracted core; cover the controller with the integration
   happy path.

Hexagonal architecture is this pattern applied to out-of-process
collaborators; functional architecture (functional core / imperative shell)
applies it to all collaborators. Prefer functional where the domain is
complex enough to pay for it.

## When orchestration needs mid-operation decisions

You can hold at most two of: domain testability, controller simplicity,
performance. Never sacrifice domain testability (never inject out-of-process
dependencies into the domain model). The usual best trade: split the
decision process into granular steps and accept a slightly smarter
controller, mitigated by:

- **CanExecute/Execute** — domain exposes `canDo(x)` checked by the
  controller before `do(x)`, keeping the rule in the domain.
- **Domain events** — the domain records what happened; the controller
  translates events into calls to unmanaged dependencies (bus, DomainLogger).

## Preconditions and logging

- Test a precondition/guard only if it carries domain meaning; skip
  null-checks and plumbing guards.
- Support logging (required by business/support staff) is observable
  behavior: route it through a DomainLogger via domain events and test it
  like any unmanaged dependency. Diagnostic logging is an implementation
  detail: never test it, use it sparingly.
