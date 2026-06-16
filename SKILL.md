---
name: new-project
description: Scaffold a new project's CLAUDE.md, ToDo.md, and LearnedPatterns.md from the workspace's unified conventions. Use when starting a new device-driver, integration-layer, or general Python/C project in this workspace and you want the standard ruleset bootstrapped. Asks a few questions (name, language, git-workflow strength, authority, domain) then writes the three files with the shared core baked in.
---

# new-project — bootstrap a project's Claude docs

This skill creates three files in a target folder from the workspace's
**unified conventions** (extracted from the existing sibling projects):

- `CLAUDE.md` — the project ruleset
- `ToDo.md` — append-only task log
- `LearnedPatterns.md` — Problem/Cause/Fix/Rule log (seeded empty)

The **shared core** (code style, lint, debug-file management, testing,
task management, learned-patterns, research-before-coding) is identical
across every project in this workspace and is baked into the templates.
Only the **variable parts** change per project — those are the questions
below.

## Step 1 — gather inputs

Before writing anything, ask the user with **AskUserQuestion** (skip any
question the user already answered in their prompt):

1. **Project name + one-line overview** — the folder name and what the
   project controls/does. (Free text; if not given in the prompt, ask.)
2. **Target folder** — default: a new folder named after the project,
   one level under the workspace root (`/workspace/<Name>`). Confirm.
3. **Language** — `Python only` or `C + Python` (C adds the Google-C++
   adapted naming table and the C build/lint section).
4. **Git workflow strength**:
   - `Full` — ToDo.md + `gh issue` + working branch + PR mandatory
     (§4 full workflow, §11–§17). Choose when the folder has / will have
     a GitHub remote.
   - `Light` — ToDo.md only; no mandatory issue/branch/PR. Choose when
     there is no remote (like PipetteLiquidHandler / SyringeLiquidHandler).
5. **Authority** — does it inherit a parent ruleset?
   - `Independent` — this CLAUDE.md is self-contained (canonical).
   - `Inherit siblings` — defers to one or more sibling CLAUDE.md files
     (ask which). Used by integration layers that import sibling drivers
     via `sys.path`.

## Step 2 — assemble CLAUDE.md

Read `templates/CLAUDE.template.md`. It is the full canonical ruleset.
Fill the placeholders and prune the conditional blocks:

- Replace `{{PROJECT_NAME}}`, `{{PROJECT_OVERVIEW}}`.
- `{{LANGUAGE_NAMING}}`: always insert `templates/python-conventions.md`;
  if language is `C + Python`, append `templates/c-conventions.md` after it.
- `{{TASK_WORKFLOW}}`: if `Full`, insert `templates/full-workflow.md` and
  append `templates/full-git-sections.md` after §8; if `Light`, insert
  `templates/light-workflow.md` and add no git sections.
- `{{AUTHORITY_SECTION}}`: if `Independent`, state this file is canonical;
  if `Inherit siblings`, insert an "Authority order" section listing the
  named sibling files and "when this file is silent, follow those".
- `{{DOMAIN_SECTION}}`: leave a stub heading (`## Hardware / domain notes`)
  with a one-line TODO for the user to fill — never invent hardware facts.

Keep the house formatting: 80-col prose, markdown tables, `[text](link)`
relative links.

## Step 3 — write ToDo.md and LearnedPatterns.md

Copy `templates/ToDo.template.md` and `templates/LearnedPatterns.template.md`
verbatim into the target folder, substituting `{{PROJECT_NAME}}` and
today's date (get it from the environment's current-date context; do NOT
shell out for it).

## Step 4 — report

Print the three created paths and a one-line note on what the user still
needs to fill (the domain section, and the overview if it was a guess).
Do not run git, do not create issues — this skill only writes the files.
