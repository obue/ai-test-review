# AI Test Case Review

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Skill Status](https://img.shields.io/badge/Skill-Compliant-brightgreen)](#)

A Claude Code skill that systematically reviews test suites for dead code, Hypothesis misuse, and quality issues - with special focus on AI-generated test anti-patterns.

## Overview

`ai-test-review` is a skill designed to audit and improve test suites. It identifies and eliminates:

- **Dead Tests** - Tests that validate test infrastructure rather than production code
- **Hypothesis Misuse** - Property-based tests applied to finite input spaces better suited for parametrized tests
- **Quality Violations** - Unit tests that violate established quality rules and best practices
- **AI Anti-Patterns** - Common failure modes specific to AI-generated test code

The skill processes your entire test codebase in a structured order (infrastructure > backend > frontend) and emits detailed reports for each file reviewed.

## Features

✅ **Systematic Codebase Scanning**
Inventories and processes all test files in logical order

✅ **Three-Stage Review Process**
1. Delete dead test code (meta-tests)
2. Replace Hypothesis with parametrized tests where appropriate
3. Apply comprehensive unit test quality rules

✅ **Anti-Pattern Detection**
Scans for 9 named failure modes common in AI-generated test suites

✅ **Actionable Reports**
Per-file completion checklist with clear violations and fixes

✅ **Reference Materials**
Detailed guides for anti-patterns, best practices, and review criteria

## Quick Start

### Prerequisites

- A test codebase to review (Python test suites using `pytest`, `Hypothesis`, or frontend tests written with `vitest`)
- Claude Code with skill support

### Installation

#### Option 1: Claude Code (Recommended)

1. Clone the repository:
   ```bash
   git clone https://github.com/obue/ai-test-review.git
   ```

2. Place it in your Claude Code skills directory:
   ```bash
   # Global skills (all projects)
   mv ai-test-review ~/.claude/skills/

   # Or project-specific (this project only)
   mv ai-test-review .claude/skills/
   ```

3. The skill is automatically discovered and ready to use:
   ```
   /ai-test-review
   ```

For detailed information, see the [official Claude skills documentation](https://support.claude.com/en/articles/12512180-use-skills-in-claude).

#### Option 2: Vercel Skills CLI

Install this skill using the [Vercel Skills CLI](https://github.com/vercel-labs/skills), a command-line tool for managing skills across AI platforms.

**Installation:**
```bash
npx skills add https://github.com/obue/ai-test-review.git
```

**Invoke:**
```
/ai-test-review
```

For more information, see the [Vercel Skills CLI documentation](https://github.com/vercel-labs/skills).

#### Option 3: Manual Installation from GitHub

1. Clone the repository:
   ```bash
   git clone https://github.com/obue/ai-test-review.git
   cd ai-test-review
   ```

2. Copy the entire directory to your agent's skills folder (varies by platform)

3. Verify `SKILL.md` is present in the root directory

## Usage

### Invoking the Skill (All Platforms)

Use the skill command in any supported agent:

```
/ai-test-review
```

Then describe what you want reviewed:

```
Review my test suite for dead code and AI anti-patterns.
The tests are in `/project/tests/`.
```

**The skill will:**
1. Read `SKILL.md` and the three companion reference files
2. Locate all test files in your codebase
3. Process them systematically in this order:
   - Infrastructure tests
   - Backend unit and integration tests
   - Frontend unit and component tests
4. Apply the three-stage review process
5. Emit a detailed report for each file

### Examples

**Example 1: Review a Django Test Suite**

```
/ai-test-review

Review my Django test suite for dead code and quality issues.
The tests are in `/project/tests/`.
```

**Expected output:**
- List of all test files processed
- Dead tests identified and deleted
- Hypothesis tests that should be parametrized
- Quality violations found and fixed
- Per-file report card

**Example 2: Review Frontend Tests with Vitest**

```
/ai-test-review

Review my frontend test suite written with vitest for dead code and quality issues.
The tests are in `/src/tests/`.
```

**Expected output:**
- List of all test files processed
- Dead tests identified and deleted
- Tests with unnecessary mocking or assertions
- Quality violations found and fixed
- Per-file report card

**Example 3: Audit AI-Generated Tests**

```
/ai-test-review

I used an AI to generate tests for my API. Please review the suite
at `/src/tests/api/` and flag any AI anti-patterns.
```

**Expected output:**
- Anti-patterns identified by name (e.g., "Mock Overuse", "Magic Values")
- Recommended fixes for each violation
- Summary compliance report

## File Structure

```
ai-test-review/
├── SKILL.md                          # Main skill instructions
├── README.md                         # This file
├── LICENSE                           # MIT License
└── references/
    ├── ai-antipatterns.md           # named anti-patterns with examples
    ├── unit-test-best-practices.md  # Quality rules and violations
    └── review-checklist.md          # Per-file checklist and report format
```

## Skill Capabilities

- **Designed for:** AI-generated test codebase reviews
- **Supports:** Python test suites (pytest, Hypothesis, unittest) and frontend tests with vitest
- **Best suited for:** Reviewing large test suites before production or after AI generation
- **Test frameworks:** pytest, unittest, Hypothesis, parametrized tests, vitest

## Reference Materials

The skill uses three companion guides:

- **`references/ai-antipatterns.md`** - Catalog of anti-patterns with real examples
- **`references/unit-test-best-practices.md`** - Quality rules and enforcement guidance
- **`references/review-checklist.md`** - Report format and completion criteria

These are automatically loaded when you invoke the skill.

## Compatibility

This skill follows the [Agent Skills standard](https://agentskills.io). Skills are automatically discovered when placed in agent specific directories at project, personal, or enterprise scope.

## License

This skill is licensed under the [MIT License](LICENSE).

## Author

**Oliver Bühler**
Version 1.0

**Best Practice:** Process the entire codebase in one session to maintain consistency across reports.
