# Mocking Policy

## Vocabulary

- **Mock** — a test double that emulates and verifies OUTGOING interactions
  (commands: calls that produce a side effect in the dependency). Spies are
  handwritten mocks.
- **Stub** — a test double that emulates INCOMING interactions (queries:
  calls the SUT makes to obtain input data). Dummies and fakes are stubs.
- CQS mapping: doubles for commands = mocks; doubles for queries = stubs.

Cardinal rule: **asserting interactions with stubs is always wrong.** Getting
input data is a means to an end, not an outcome. Set stubs up; never verify
them — no call history, counts, or order.

The counterweight: what the SUT asks for still matters through what it gets
back. "Fetch the forecast for the snapped week" is an outcome requirement, and
a stub that answers every query the same way cannot test it.

- Key canned data by the arguments the SUT is responsible for choosing:
  `{("SKU-1", "2025-06-02"): rows}`, not one `return_value` for every call.
  A wrong period, ID, or model then yields a wrong or missing result, and the
  test fails on outcome alone.
- Model only the dimensions the scenario protects; don't key every parameter
  in every test.
- Reject a query only the way the real dependency would (not found, empty).
  A stub that raises on anything but one exact call shape is a hidden
  interaction assertion and breaks on harmless refactors (e.g. fetching a
  wider range and filtering locally).
- When the result cannot reflect the argument (heavy payloads, a fake shared
  by many tests), keep one focused assertion on the request at the edge rather
  than lose the protection. Record requests in the fake and assert them in the
  one test that owns that rule.

## The one decision table

| Dependency | Examples | In tests |
| --- | --- | --- |
| In-process collaborator | your own classes, domain objects | Use the real thing, always. Never mock. |
| Managed out-of-process | your application's database, file store only you touch | Real instance in integration tests. Interactions with it are implementation details; verify its final STATE, not calls. |
| Unmanaged out-of-process | third-party API, message bus, SMTP, messages delivered to the user (chat UI/websocket, email, SMS, push), anything other systems or people observe | Mock it — this is the ONLY legitimate mock target. Its messages are observable behavior: backward-compatible contracts you must keep. |
| Mixed (e.g. a DB other apps also read) | shared tables | Split: mock the externally observed part, treat the rest as managed. |

Unmanaged status decides whether you substitute the dependency; CQS decides
whether you verify the interaction. One external API can expose both: stub
its queries, verify its commands. A response your own entry point returns
(an HTTP response body) is direct output, not a dependency: assert it directly.

Consequences that follow mechanically:

- Only controllers talk to out-of-process dependencies → **mocks appear only
  in integration tests, never in unit tests.**
- The number of mocks in a test is not a smell by itself; it equals the number
  of unmanaged dependencies in the operation. One mock of an in-process class
  IS a smell.

## How to mock the unmanaged dependency

- **Mock at the system edge** — the last of your types before the wire (the
  message-bus client wrapper, not the `IMessageBus` interface three layers up).
  More of your code runs under test (better regression protection) and the
  assertion checks the actual outbound message, detached from internals
  (better refactoring resistance).
- **Only mock types you own.** Wrap third-party SDKs in your own adapter and
  mock the adapter. The adapter is also where you encode your understanding of
  the library's behavior.
- **Record at the effect.** A spy appends inside the call that performs the
  side effect (`send`, `publish`, `write`), never when the message or command
  object is constructed — construction is intent, the call is the effect. For
  async APIs record inside the coroutine (or `assert_awaited_once_with`): the
  test must fail if the `await ...send()` is removed.
- **Verify both directions:** the expected calls happened AND no unexpected
  calls happened.
- Prefer handwritten spies at the edge: assertions become reusable fluent
  helpers (`busSpy.shouldSendNumberOfMessages(1).withOrderConfirmation(id)`),
  shrinking tests and keeping expected values independent of production code.
- Exception for low-stakes contracts: where message structure doesn't matter
  and nobody depends on exact shape (typically logging), mocking a higher
  interface is acceptable.

## Interfaces and mocks

An interface with a single implementation is not an abstraction. Its only
legitimate purpose is to enable mocking of an UNMANAGED dependency. A
single-implementation interface over a managed dependency or an in-process
class is a red flag: it exists to enable a mock this policy forbids. Use the
concrete class.
