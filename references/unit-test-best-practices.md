# Unit Test Best Practices

This file is a reference for the `test-review` skill. Apply every rule below to each test during Step 3 of the review. Fix violations in place.

---

## Rule 1 — Naming

Every test name must answer three questions: *what unit?* — *under what condition?* — *what is the expected outcome?*

**Accepted patterns:**
- `test_<unit>_<condition>_<expected_result>` — e.g. `test_invoice_with_zero_items_raises_value_error`
- Given / When / Then — e.g. `test_given_expired_token_when_authenticating_then_returns_401`

**Never acceptable — rename every one:**
`test_1`, `test_it`, `test_ok`, `test_function`, `test_check`, `test_works`, `test_method`, `test_case`

Test body length should rarely exceed 20–30 lines. Longer tests should be split or have their setup extracted to a named fixture.

---

## Rule 2 — Arrange / Act / Assert (AAA) Structure

Every test must have a clear three-part structure with explicit `# Arrange`, `# Act`, `# Assert` comments. This is not optional: the comments make violations immediately visible during review.

```python
def test_invoice_total_includes_tax():
    # Arrange
    line_items = [LineItem(amount=100), LineItem(amount=50)]
    tax_rate = Decimal("0.20")

    # Act
    invoice = Invoice(line_items=line_items, tax_rate=tax_rate)

    # Assert
    assert invoice.total == Decimal("180.00")
```

**Violations to fix:**
- Multiple Act steps in one test → split into separate tests, one per behaviour
- Assertions interleaved with setup → restructure into three clean blocks
- Magic values (e.g. `assert result == 42`) → extract to a named variable that explains intent (e.g. `expected_total = Decimal("42.00")`)

---

## Rule 3 — Test Behaviour, Not Implementation

Assert on **externally observable outcomes**: return values, raised exceptions, state changes visible through the public API, or side effects on collaborators that are part of the contract.

Never assert on: private fields, internal dict structure, the order of internal operations, or intermediate object shapes.

**The refactoring test:** *"Would this assertion need to change if I refactored the internal implementation without changing the observable behaviour?"* → Yes → it is testing implementation. Rewrite it.

```python
# BAD — implementation detail (private internal dict)
def test_user_creation():
    user = User(name="Alice", age=30)
    assert user._internal_dict["name"] == "Alice"

# GOOD — observable public API
def test_user_creation():
    user = User(name="Alice", age=30)
    assert user.to_dict() == {"name": "Alice", "age": 30}
```

---

## Rule 4 — Assertions Must Be Meaningful (Anti-Tautology Check)

For every assertion, ask: *"Could this assertion fail if the production code contained a bug?"*

Reject any assertion that:
- Compares a value to itself
- Compares a mock's return value to what was configured on that mock
- Only checks `is not None` after calling a constructor that never returns `None`
- Is always `True` regardless of what the code under test does

**The mutation test:** Mentally delete the core logic of the function under test (replace it with `return None` or `pass`). Would the test fail? → No → the assertion is tautological and must be replaced or the test deleted.

---

## Rule 5 — Single Responsibility

Each test verifies **exactly one behaviour or one code path**.

**Acceptable:** multiple assertions that collectively verify a single outcome (e.g. asserting several fields of a returned object are correct together).

**Not acceptable:** one test that checks both a success path and an error path, or that tests two independent features.

If a test has multiple Arrange → Act → Assert cycles, split it into that many tests.

---

## Rule 6 — Isolation and Independence

Tests must be hermetically isolated from each other:

- **No execution-order dependency** — every test must be runnable in isolation and in any order
- **No shared mutable state** — no module-level or class-level mutable variables shared across tests
- **Self-contained setup** — each test arranges its own data; fixtures must use `function` scope by default for any mutable state
- **Database tests** — must roll back or use isolated transactions per test; never leave data behind
- **File system / queue / cache** — any test that writes must clean up in teardown, or use a temp directory / in-memory substitute

---

## Rule 7 — Mocking: Boundary Only

| Rule | Detail |
|---|---|
| Mock only at system boundaries | External HTTP calls, real databases, file system, clocks, message queues, third-party SDKs |
| Do not mock internal classes | If you mock a class that lives inside the same codebase, you are testing the wrong layer |
| No over-mocking | If more than half the test body is mock setup, the test is likely testing nothing real |
| Verify interactions sparingly | Only assert `.call_args` / `.call_count` when the interaction itself — not the result — is the contract |
| Prefer fakes over mocks | A hand-written in-memory fake (`FakeEmailService`, `InMemoryRepository`) is more reliable than a deeply configured `Mock` object with many `.return_value` chains |

For every mock in a test, ask: *"Am I mocking this because it has a real side effect (network, disk, time), or because it was convenient?"* If the latter, remove the mock and test with the real object.

---

## Rule 8 — Edge Cases and Boundary Conditions

AI-generated tests are systematically biased toward happy paths. For every function under test, verify the following cases exist. **Add any that are missing.**

| Case | What to test |
|---|---|
| `None` / `null` input | Graceful handling or raises the correct, documented exception |
| Empty input | Empty string `""`, empty list `[]`, empty dict `{}` |
| Zero / negative numbers | Where positive values are expected — does it guard or raise? |
| Boundary values | Minimum and maximum allowed values; the value just outside each boundary |
| Invalid format | A string where a number is expected, a malformed date, an invalid enum value |
| Maximum length / size | Input that exceeds defined limits |
| Concurrent / repeated calls | If the function is stateful — verify idempotency or correct state transitions |

---

## Rule 9 — No Logic Inside Tests

Test bodies must be flat, linear sequences of Arrange → Act → Assert. Remove:

- `if` / `else` / `match` blocks
- Loops that alter the assertion target (a loop that only builds expected data is acceptable; one that also runs assertions is not)
- `try` / `except` blocks that swallow failures or make a failing test look like it passed

If logic is needed to prepare test data, extract it into a clearly named helper function or fixture. The helper must not contain assertions.

---

## Rule 10 — Coverage Targets

After reviewing each module, verify coverage meets these minimums. Note that **line coverage alone is insufficient** — AI-generated tests systematically under-cover conditional branches. Always check branch coverage.

| Layer | Line Coverage | Branch Coverage |
|---|---|---|
| Core domain / business logic | 90% | 80% |
| Application services | 80% | 70% |
| Infrastructure adapters | 70% | 60% |
| Frontend components (unit) | 80% | 70% |

**How to measure:**
- Python: `pytest --cov=<module> --cov-branch --cov-report=term-missing`
- JavaScript / TypeScript: `jest --coverage` with `branches` threshold in `jest.config`

For any module below threshold: add the missing tests, or add a `# TODO(test-coverage): <justification>` comment explaining the gap.

---

## Rule 11 — Mutation Testing (Critical Modules)

Line and branch coverage can reach 100% while tests assert on nothing meaningful. Mutation testing is the only reliable way to detect tautological and evergreen tests. It works by introducing small automated code changes (mutants) and checking whether any test fails — surviving mutants reveal tests that provide no real protection.

**Run on all critical business logic modules:**
- Python: `mutmut run` (or `cosmic-ray` for larger projects)
- JavaScript / TypeScript: Stryker Mutator

**Target score:** ≥ 70% mutation kill rate. Below this threshold, treat the module as insufficiently tested regardless of line coverage.

**For each surviving mutant:**
1. Write a test that kills it (preferred), or
2. Document it with a comment: `# mutant-survivor: <reason this is an acceptable equivalence>`

---

## Rule 12 — Test Speed

| Threshold | Action |
|---|---|
| < 200 ms | Acceptable for a unit test |
| 200 ms – 1 s | Investigate: is it making real I/O? Should it be mocked? |
| > 1 s | Must be refactored (add mocking) or reclassified as an integration test |

Mark slow tests explicitly:
- Python: `@pytest.mark.slow`
- Jest / Vitest: `.slow()` modifier or a custom tag

Slow tests should be excluded from the fast feedback loop (pre-commit, watch mode) but still run in CI.
