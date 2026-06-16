<!-- Appended after §8 when git workflow is "Full". -->

---

## 11. Commit Messages

Follow **Conventional Commits**: `<type>(<scope>): <description>`.

- Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `style`, `perf`.
- Imperative mood ("Add", not "Added"); subject under 50 chars; no trailing
  period; wrap body at 72 chars; body explains what and why.
- Breaking changes: `feat(api)!: ...` plus a `BREAKING CHANGE:` footer.

## 12. Branching Strategy

**GitHub Flow**: `main` is always deployable; work on `<type>/<short-desc>`
branches cut from `main`; merge via PR; delete branches after merge; open
PRs even when working solo.

## 13. .gitignore

Cover Python (and C, if present) build/cache artifacts, editor/OS files,
and secrets. Base on GitHub's official templates.

## 14. Versioning

**Semantic Versioning** (`MAJOR.MINOR.PATCH`). `fix:` → PATCH, `feat:` →
MINOR, `BREAKING CHANGE` → MAJOR. `0.y.z` is unstable; first stable is
`1.0.0`. Tag with annotated tags (`git tag -a vX.Y.Z`).

## 15. Pull Request Guidelines

PR title uses Conventional Commits format. Description: Changes / Why /
Testing / Related Issues. Keep PRs under 400 lines when possible.

## 16. Git Automation (optional)

Use `pre-commit` with `ruff` / `ruff-format` (and `clang-format` for C).
