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

- Format: `<type>/<description>`. Types: `feature`, `bugfix`, `hotfix`, `release`, `chore`. Docs, refactor, test, and other maintenance work goes under `chore/`.
- Purpose prefix always; agent prefixes (`ai/`, `claude/`, ...) stay unused.
- Known ticket: `feature/issue-123-add-login`.

# Flow

Trunk-based. Every PR lands on trunk as one squash commit.

- Branch from trunk and keep it short-lived. Update a branch from trunk by rebasing onto trunk.
- The PR title is the Conventional Commit that lands on trunk. Branch commits follow the rules in Git.
- Work in your own worktree and leave other worktrees untouched.
- Tickets built in parallel touch disjoint files. Tickets that share files are chained with blocking edges.

# Agency

- Commit when asked. Merging into trunk and commits to trunk happen only on explicit request.
- Push only on explicit request. A request to push a rebased branch covers `--force-with-lease` on that branch. Plain `--force` and any force-push to trunk are off-limits.
- Invoking a skill that commits, pushes or opens a PR is the explicit request for those steps.
