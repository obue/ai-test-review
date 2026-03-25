# AI-Generated Test Anti-Patterns

This file is a reference for the `ai-test-review` skill. Scan every test file for these patterns **before** applying the step-by-step review. They are statistically overrepresented in AI-generated test suites.

Research context: CodeRabbit's December 2025 analysis found AI-authored pull requests average **10.83 issues each** versus 6.45 in human-only submissions — logic and correctness errors are 75% more prevalent, and AI-generated tests achieve only ~40% mutation kills because they avoid edge cases and stick to training-data-frequent patterns.

---

## The Mirror (Tautological Test)

**What it is:** The assertion re-computes the same expression as the implementation instead of comparing against a hardcoded expected value. It always passes, including when the code is wrong.

```python
# BAD — mirrors the implementation, always passes
def test_get_full_name():
    user = User(first="John", last="Doe")
    assert user.full_name == user.first + " " + user.last  # just restates the code

# GOOD — asserts a concrete expected outcome
def test_get_full_name():
    user = User(first="John", last="Doe")
    assert user.full_name == "John Doe"
```

**Detection:** If you replaced the production function with `return input_a + input_b`, would the test still pass? → Tautological.

**Fix:** Replace the right-hand side of every assertion with a hardcoded expected value derived from the specification, not the implementation.

---

## The Liar (Evergreen Test)

**What it is:** A test that passes in every scenario, including when the feature is completely broken. The test is green, but it is lying — it has no power to detect regressions.

Common causes:
- `except Exception: pass` swallowing all failures
- Asserting on a variable that is always truthy (e.g. a non-empty list object, a class instance)
- Asserting `is not None` after constructing an object that can never be `None`
- The assertion condition is a Python truism (e.g. `assert True`, `assert len(x) >= 0`)

```python
# BAD — always passes; len() is never negative
def test_results_returned():
    results = search(query="test")
    assert len(results) >= 0

# GOOD — asserts something that can actually fail
def test_search_returns_matching_results():
    results = search(query="apple")
    assert len(results) > 0
    assert all("apple" in r.title.lower() for r in results)
```

**Detection:** Mentally delete the entire body of the production function (make it `return None` or `pass`). Does the test still pass? → Evergreen.

**Fix:** Find the assertion that is always true and replace it with one that would fail if the code were broken.

---

## Testing the Mock

**What it is:** The test sets up a mock with a return value and then asserts that the mock returned that value. This only proves the mock framework works — it says nothing about the unit under test.

```python
# BAD — asserts on the mock's own configured return value
def test_get_price():
    mock_service = Mock()
    mock_service.get_price.return_value = Decimal("9.99")
    result = mock_service.get_price(item_id=1)
    assert result == Decimal("9.99")  # we literally just set this

# GOOD — the real unit under test uses the mock as a dependency
def test_checkout_total_uses_price_service():
    mock_price_service = Mock()
    mock_price_service.get_price.return_value = Decimal("9.99")
    checkout = Checkout(price_service=mock_price_service)
    total = checkout.calculate_total(item_id=1, quantity=3)
    assert total == Decimal("29.97")  # tests Checkout logic, not the mock
```

**Detection:** Remove the unit under test entirely and call the mock directly. Does the test still pass? → Testing the mock.

**Fix:** Ensure the test exercises real production code that *uses* the mock as a collaborator. Assert on the production code's output.

---

## The Dodger

**What it is:** The test has many assertions about trivial side effects (logging calls, attribute assignments, event emissions) but never verifies the core intended behaviour of the function — the thing it was actually built to do.

**Detection:** Ask *"What is this function's primary job?"* Then check whether any assertion actually verifies that job.

**Fix:** Add an assertion on the primary return value or the most important observable state change. Trim or remove assertions about incidental side effects.

---

## The Happily Naive (Happy-Path Only)

**What it is:** AI models are biased toward the common case in their training data. Functions with more than 5 branches average under 30% coverage from AI-generated tests. Null inputs, empty collections, zero values, and error paths are routinely absent.

**Detection:** For every tested function, check for the following missing cases:
- `None` / `null` input where the parameter is not marked non-nullable
- Empty string, empty list, empty dict
- Zero or negative numbers where positive numbers are expected
- Input that exceeds a maximum length or value
- Input of the wrong type or format
- The error / exception path

**Fix:** Add the missing tests. See `unit-test-best-practices.md` § Edge Cases for a complete checklist.

---

## The Bug Validator

**What it is:** A test written to match the current (incorrect) behaviour of the code. When the bug is later fixed, this test fails — causing confusion and undermining trust in the test suite.

**Detection:** Ask for every assertion: *"Is this what the code **should** do, or merely what it **currently** does?"* Cross-reference the specification, the docstring, or the ticket.

**Fix:** Correct the assertion to match the specified behaviour. If the specification is unclear, add a comment flagging the ambiguity rather than encoding the bug.

---

## The Structural Inspector

**What it is:** The test accesses private fields, internal dict structures, or implementation details via reflection or direct attribute access. It breaks on any internal refactor, even when observable behaviour is unchanged.

```python
# BAD — accesses private internal dict
def test_user_data():
    user = User(name="Alice", age=30)
    assert user._data["name"] == "Alice"

# GOOD — tests via the public API
def test_user_serialisation():
    user = User(name="Alice", age=30)
    assert user.to_dict() == {"name": "Alice", "age": 30}
```

**Fix:** Rewrite to use only the public API. If there is no public API to observe the behaviour, that is a design smell in the production code — flag it.

---

## The Analyst (Overloaded Test)

**What it is:** One test that verifies 5 different, unrelated behaviours across multiple Act steps. Failures are hard to diagnose because the test name describes none of the specific scenarios.

**Detection:** Count the number of distinct Act + Assert groups. More than one → split.

**Fix:** Split into one test per behaviour. Give each a descriptive name.

---

## The Sleeper (Hidden I/O)

**What it is:** A unit test that secretly makes real network calls, real database queries, or real file-system writes — either because dependencies were not mocked or because a fixture with the wrong scope wires up a real connection.

**Detection:** Run the test with the network disabled or the database offline. Does it fail with a connection error? → The Sleeper.

**Fix:** Mock at the boundary (see `unit-test-best-practices.md` § Mocking). Move to an integration suite if real I/O is genuinely required.

---

## Quick Classification Table

| Name | One-line symptom | Primary fix |
|---|---|---|
| The Mirror | Assertion re-computes the implementation | Use a hardcoded expected value |
| The Liar | Always passes, even if code is deleted | Find and replace the always-true assertion |
| Testing the Mock | Asserts on mock's own return value | Assert on the real unit's output |
| The Dodger | Tests side effects, misses core behaviour | Assert on primary return value or state change |
| The Happily Naive | Only the happy path is covered | Add null, empty, boundary, and error tests |
| The Bug Validator | Encodes current wrong behaviour | Fix assertion to match the spec |
| The Structural Inspector | Accesses private/internal members | Use the public API only |
| The Analyst | Multiple unrelated behaviours in one test | Split into one test per behaviour |
| The Sleeper | Hidden real network or DB calls | Mock at the boundary |