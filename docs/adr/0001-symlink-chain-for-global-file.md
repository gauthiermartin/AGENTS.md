# Wire the global file with symlinks, no install script

`~/.claude/CLAUDE.md` → `~/.agents/AGENTS.md` → `<repo>/global/AGENTS.md`. omp reads `~/.agents/AGENTS.md` natively and Claude Code follows the chain; both were checked with `claude -p` and `omp -p` from outside the repo. The wiring is two `ln -s` commands in the README, not a script, because the script would be longer than what it automates.

## Considered Options

- **`@` stub** (`~/.claude/CLAUDE.md` containing `@~/.agents/AGENTS.md`): the documented import route, and it avoids the Claude Code rule that a symlinked `~/.claude/CLAUDE.md` is skipped in Cowork sessions. Rejected for now: it still passes through a symlink at the import step (untested), and only the CLI is in use.
- **Copy on sync:** avoids symlink behaviour entirely but reintroduces drift.

## Revisit when

Claude Desktop or Cowork sessions are adopted. Then switch `~/.claude/CLAUDE.md` to the stub and re-run the verification in the README.

## Consequences

The live file is whatever the checkout has on disk. Checking out a branch without `global/AGENTS.md`, or with a different version, changes or removes the global rules for every agent immediately. The alternative of a `git worktree` pinned to `main` as the link target avoids this and was not adopted, because each edit would then need a merge before it takes effect.
