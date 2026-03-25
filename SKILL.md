---
name: ai-test-review
description: "Iteratively review an entire test codebase for dead code, Hypothesis misuse, and unit test quality issues — with special focus on AI-generated test anti-patterns."
license: MIT License
compatibility: Designed for ai-generated test codebase reviews using pytest, Hypothesis, and similar testing frameworks
metadata:
  author: Oliver Bühler
  version: "1.0"
---

You are performing an **iterative, systematic test codebase review**. Before starting any work, read all three companion files in this skill folder:

- `references/ai-antipatterns.md` — Named failure modes common in AI-generated test suites. Scan every file for these first.
- `references/unit-test-best-practices.md` — Detailed quality rules to apply to every remaining test.
- `references/review-checklist.md` — Per-file completion checklist and output report format.

---

## Scope and File Ordering

Review all test files in this order:
1. Infrastructure tests
2. Backend tests (unit → integration)
3. Frontend unit and component tests

Begin by listing all test files in the codebase so you have a complete inventory. Then process them one by one. Do not skip files.

---

## Step 1 — Delete Dead Test Code (Meta-Tests)

Remove any test that tests another test function, fixture, or test utility instead of production code.

**Decision rule:** *"Does this test, if it passes, give any confidence about production code?"* → No → delete.

What to look for:
- Subject under test is a `test_*` function, a fixture, or a mock factory
- Test calls another test function directly
- Test asserts on test-helper structure rather than application behaviour
- Entire file contains only dead tests → delete the file

Do not refactor. Delete only.

---

## Step 2 — Replace Hypothesis Tests With Known-Space Functional Tests

Remove `@given`-decorated Hypothesis tests where the input space is finite or enumerable. Replace with explicit parameterised tests.

**Replace when:**
- Input is drawn from a closed enum, fixed string literals, a boolean, or a small integer range
- Strategy is so restricted it adds no value over `@pytest.mark.parametrize`
- Property is a concrete business rule, not a universal invariant

**Keep when:**
- Input space is genuinely unbounded (arbitrary strings, large integers, nested structures)
- Test verifies a true invariant (idempotency, commutativity, round-trip serialisation)

```python
# BEFORE — Hypothesis with known input space
@given(st.sampled_from(["draft", "published", "archived"]))
def test_status_is_valid(status):
    assert Article(status=status).status in ArticleStatus.values()

# AFTER — explicit parameterised test
@pytest.mark.parametrize("status", ["draft", "published", "archived"])
def test_status_is_valid(status):
    assert Article(status=status).status in ArticleStatus.values()
```

Remove unused Hypothesis imports after replacement.

---

## Step 3 — Unit Test Quality Review

Apply every rule in `references/unit-test-best-practices.md` to each remaining test. Fix violations in place.

---

## After Each File

Emit the report format defined in `references/review-checklist.md`, then continue to the next file automatically.