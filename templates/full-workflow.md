<!-- {{TASK_WORKFLOW}} — "Full" git workflow. Replaces §4 body; keep the
     §11–§17 git sections that ship in full-git-sections.md. -->

> **MANDATORY**: Applies to every task that touches code or docs. No task
> begins without a `ToDo.md` entry and a GitHub issue.

### Rules

1. **Write ToDo.md** for every task and confirm contents before starting.
2. **Accumulate** — append new tasks below existing ones; never overwrite.
   `ToDo.md` is a cumulative command history.
3. **Register a GitHub issue** via `gh issue create`.

### Command input validation

Before writing ToDo.md, confirm the request is explicit (target / method /
purpose) and check for reference materials (PDFs, docs). Do not proceed if
either is missing.

### Workflow

1. Validate the command input.
2. Organize the task list in `ToDo.md`.
3. Get the user's confirmation on the `ToDo.md` contents.
4. Create a GitHub issue via `gh issue create`.
5. Cut a working branch from `main`: `<type>/<short-description>`.
6. Check items off as work progresses; commit per Conventional Commits.
7. Update the issue via `gh issue edit` for completed items.
8. Push the branch.
9. Open a PR via `gh pr create` (Changes / Why / Testing / Related Issues).
10. After merge, delete the local branch.

> Steps 2, 4, 5, and 9 are non-negotiable for any code/doc change.

Checkbox flips in `ToDo.md` (`- [ ]` → `- [x]`, appending a commit hash or
issue link) are permitted; prose rewrites, reordering, and deletion are not.
