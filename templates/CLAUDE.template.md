# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working
with code in the **{{PROJECT_NAME}}** project.

## Overview

{{PROJECT_OVERVIEW}}

{{AUTHORITY_SECTION}}

## Environment

This project runs inside a **Docker container** with Claude Code as the
primary development tool.

| Item    | Detail                                |
|---------|---------------------------------------|
| Runtime | Docker container (`--privileged`)     |
| OS      | Ubuntu 24.04 (Noble)                  |
| Dev tool| Claude Code (CLI / VS Code extension) |

## Commands

| Purpose          | Command                          |
|------------------|----------------------------------|
| Lint a Python file | `ruff check <file>.py`         |
| Check formatting | `ruff format --check <file>.py`  |
| Auto-format      | `ruff format <file>.py`          |

---

## 1. Rule Priority

Project-level `CLAUDE.md` sections take precedence over any inherited
ruleset. Specific rules beat general ones; the more-specific context wins.

---

## 2. Code Convention

All code follows the [MIT CommLab Coding and Comment Style](https://mitcommlab.mit.edu/broad/commkit/coding-and-comment-style/).

### Naming

- **Variables and classes** are nouns; **functions and methods** are verbs.
- Names must be pronounceable and straightforward.
- Name length is proportional to scope: short for local, descriptive for
  broad. Avoid abbreviations unless self-explanatory.

{{LANGUAGE_NAMING}}

### Structure

- **80-column limit** for all new code.
- One statement per line.
- Indent with **4 spaces** (never tabs).
- Place operators on the **left side** of continuation lines.

### Spacing

- One space after commas, none before: `foo(a, b, c)`.
- One space on each side of `=`, `==`, `<`, `>`, etc.

### Comments

- Use complete sentences. Only comment for **context** or **non-obvious
  choices** — never restate what the code already says.
- Outdated comments are worse than none. Keep them current or delete them.

### Language

- All code comments, docstrings, commit messages, documentation (including
  README), GitHub issues, and pull requests are written in **English**.

### Documentation

- All public functions and classes have **docstrings** (PEP 257 / Google
  style). A docstring states **what** and **why**, not **how**. Include
  `Args:`, `Returns:`, and `Raises:` when applicable.

---

## 3. Debug File Management

| Location       | What goes there                                     |
|----------------|-----------------------------------------------------|
| `tests/`       | Production-quality tests that are part of CI.       |
| `claude_test/` | Debug scripts, one-off experiments, diagnostics.    |

- Create debug files directly in `claude_test/` with a one-line purpose
  docstring at the top.
- `claude_test/README.md` is the index — add a row per new file describing
  what it does and what was learned.
- `claude_test/` is **exempt** from the 80-column limit and mandatory
  docstrings. Anything promoted into `tests/` must conform fully.

---

## 4. Task Management

{{TASK_WORKFLOW}}

---

## 5. Testing Rules

1. **No magic numbers** — define meaningful constants, never bare literals.
2. **No hardcoding to match test inputs** — fix the logic, not the branch.
3. **Code quality first** — readability and correctness beat green CI.

---

## 6. Linting

All Python code must pass **Ruff** before committing.

1. **Line length**: 80 columns (`line-length = 80`).
2. Before committing, run `ruff check <file>.py` and
   `ruff format --check <file>.py`.
3. Fix before committing; use `ruff format <file>.py` to auto-format.

---

## 7. Research Before Coding

Before calling into an unfamiliar library, API, or CLI, verify its actual
interface rather than guessing from memory.

1. Consult official documentation first (Context7 MCP or web search).
2. Search the repository for prior implementations of the same interface.
3. Trust documentation over intuition; read the source over guessing.

---

## 8. Learned Patterns

Treat [`LearnedPatterns.md`](LearnedPatterns.md) as part of the workflow.

1. **Before drafting a `ToDo.md` entry**, read the relevant sections.
2. **Reference applicable patterns** in the ToDo entry as `(see LP §X)`.
3. **After a task**, append any new recurring issue, gotcha, library quirk,
   workflow lesson, or environment note using the **Problem / Cause / Fix /
   Rule** format, classified into §1–§5 (or §99 Uncategorized).
4. **Promote** patterns that stabilize across many tasks into a formal rule
   in this file, and remove them from `LearnedPatterns.md`.

---

## Hardware / domain notes

<!-- TODO: Fill in this project's hardware, protocol, ports, and gotchas.
     Never let Claude invent these — they must come from real specs and
     bench observation. Identify devices by stable USB VID:PID, document
     any protocol quirks, and cross-link to LearnedPatterns.md. -->
