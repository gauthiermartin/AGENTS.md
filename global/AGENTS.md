# Git

Follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) and [Conventional Branch 1.1.0](https://conventionalbranch.org/). Repo-local conventions (local `AGENTS.md`, commitlint config, CONTRIBUTING, a visibly different `git log`) override this section.

## Commits

- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
- Scope only when the repo's `git log` already uses scopes; reuse the existing names.
- Description: lowercase, no trailing period; subject ≤72 chars. Body only when the why is not obvious; it explains why, not what.
- Known ticket: `Refs: #123` footer. No known ticket, no footer.
- One logical change, one type, per commit; split mixed changes.
- Commit messages carry no `Co-authored-by` or "Generated with" trailers.

## Branches

- Format: `<type>/<description>`. Types: `feat`, `fix`, `hotfix`, `release`, `chore`. Docs, refactor, test, and other maintenance work goes under `chore/`.
- Purpose prefix always; agent prefixes (`ai/`, `claude/`, ...) stay unused.
- Known ticket: `feat/issue-123-add-login`.

# Agency

- Commit when asked. Push, force-push, and commits to trunk happen only on explicit request.
