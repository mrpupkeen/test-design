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
them.

## The one decision table

| Dependency | Examples | In tests |
| --- | --- | --- |
| In-process collaborator | your own classes, domain objects | Use the real thing, always. Never mock. |
| Managed out-of-process | your application's database, file store only you touch | Real instance in integration tests. Interactions with it are implementation details; verify its final STATE, not calls. |
| Unmanaged out-of-process | third-party API, message bus, SMTP, anything other systems observe | Mock it — this is the ONLY legitimate mock target. Its messages are observable behavior: backward-compatible contracts you must keep. |
| Mixed (e.g. a DB other apps also read) | shared tables | Split: mock the externally observed part, treat the rest as managed. |

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
