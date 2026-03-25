# Per-File Review Checklist and Output Format

This file is a reference for the `ai-test-review` skill. Use this checklist before closing each file and emit the report below after completing it.

---

## Per-File Completion Checklist

Work through every item before moving to the next file. Only mark an item complete when it has been fully verified — not just inspected.

### Step 1 — Dead Code
- [ ] Every test in the file has been checked against the dead-test decision rule
- [ ] All meta-tests (tests of test functions, fixtures, or helpers) have been deleted
- [ ] If the entire file was dead code, the file itself has been deleted

### Step 2 — Hypothesis
- [ ] All `@given`-decorated tests have been identified
- [ ] Each one has been evaluated for known vs. unbounded input space
- [ ] Known-space tests have been replaced with parameterised equivalents
- [ ] Replacement tests cover at least the same concrete cases the strategy would have generated, plus explicit boundary values
- [ ] Unused Hypothesis imports have been removed

### Step 3 — AI Anti-Patterns (from `ai-antipatterns.md`)
- [ ] No tautological / Mirror tests remain
- [ ] No evergreen / Liar tests remain
- [ ] No "Testing the Mock" tests remain
- [ ] No Dodger tests (side-effects only, no core behaviour assertion) remain
- [ ] No Bug Validator tests (encodes wrong behaviour) remain
- [ ] No Structural Inspector tests (private/internal access) remain
- [ ] No Analyst tests (multiple unrelated behaviours in one test) remain
- [ ] No Sleeper tests (hidden real I/O) remain

### Step 3 — Quality Rules (from `unit-test-best-practices.md`)
- [ ] All test names follow the `test_<unit>_<condition>_<expected_result>` or Given/When/Then pattern
- [ ] No generic names (`test_1`, `test_it`, `test_ok`, etc.) remain
- [ ] All tests have `# Arrange`, `# Act`, `# Assert` comments
- [ ] No test has multiple Act steps
- [ ] No magic values in assertions — all expected values are named variables
- [ ] No test asserts on private fields or internal implementation details
- [ ] Each test verifies exactly one behaviour
- [ ] Tests are isolated: no shared mutable state, no execution-order dependencies
- [ ] Mocks are used only at system boundaries; no internal classes are mocked
- [ ] Edge cases covered: null/None, empty, zero, boundary values, invalid input, error path
- [ ] No conditional logic, loops over assertions, or try/except in test bodies
- [ ] Coverage thresholds met (line + branch) or gaps documented with `# TODO(test-coverage):` comment
- [ ] Tests run under 200 ms; slow tests are marked and reclassified if needed
- [ ] Mutation score ≥ 70% for critical business logic (or surviving mutants documented)

### Step 4 — Project-Specific Criterion
- [ ] Step 4 applied (if defined), or noted as pending in the report

---

## Report Format

Emit this report after completing each file. Keep it brief — one line per category.

```
File: <relative path to test file>
──────────────────────────────────────────────────
Deleted (dead/meta-code):      <n> tests  |  reason: <summary>
Deleted (tautological/liar):   <n> tests
Replaced (Hypothesis):         <n> tests → <n> parameterised tests
AI anti-pattern fixes:         <list of named patterns corrected>
Quality fixes:                 <n renamed, n split, n mocks removed, …>
New tests added:               <n>  |  cases: <null input / boundary / error path / …>
Coverage before → after:       <n>% line / <n>% branch  →  <n>% line / <n>% branch
Mutation score:                <n>%  (or: not measured / pending)
Step 4:                        <applied / pending / n/a>
Remaining issues:              <None | description of anything left unresolved>
──────────────────────────────────────────────────
```

**Example:**
```
File: tests/services/test_invoice_service.py
──────────────────────────────────────────────────
Deleted (dead/meta-code):      2 tests  |  reason: testing fixture helpers, not production code
Deleted (tautological/liar):   1 test   |  reason: assertion always True (len >= 0)
Replaced (Hypothesis):         1 test → 3 parameterised tests
AI anti-pattern fixes:         The Mirror (×2), The Happily Naive (×1)
Quality fixes:                 3 renamed, 2 split, 1 internal mock removed
New tests added:               4  |  cases: None input, empty list, negative amount, max boundary
Coverage before → after:       63% line / 41% branch  →  91% line / 83% branch
Mutation score:                79%
Step 4:                        pending
Remaining issues:              None
──────────────────────────────────────────────────
```

---

## Session Summary

After all files have been processed, emit a single session-level summary:

```
══════════════════════════════════════════════════
TEST REVIEW — SESSION SUMMARY
══════════════════════════════════════════════════
Files reviewed:                <n>
Files deleted (all dead):      <n>

Total tests deleted:           <n>  (dead: <n> | tautological/liar: <n>)
Total Hypothesis replacements: <n> tests → <n> parameterised tests
Total new tests added:         <n>

Most common AI anti-patterns:  <ranked list>
Most common quality violations: <ranked list>

Coverage change (aggregate):   <before avg>% line / <before avg>% branch
                             → <after avg>%  line / <after avg>%  branch

Modules below coverage target: <list or None>
Modules with mutation score
below 70%:                     <list or None>

Step 4 status:                 <pending / applied to n files>
══════════════════════════════════════════════════
```