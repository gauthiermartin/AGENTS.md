# AGENTS.md

Version-controlled source of my global (user-level) agent instructions, with the research and decisions behind them.

Agents served: Claude Code and omp.

## Layout

- `global/AGENTS.md`: the global file. This is the only thing agents load.
- `AGENTS.md`: this repo's own project instructions (not global).
- `project/AGENTS.md`: the project template, copied into new projects (not loaded from here).
- `docs/research/`: sources behind the rules.
- `docs/adr/`: decisions that are hard to reverse and surprising without context.

## Wiring

omp reads `~/.agents/AGENTS.md` natively. Claude Code reads `~/.claude/CLAUDE.md`.

```sh
ln -sfn "$PWD/global/AGENTS.md" ~/.agents/AGENTS.md
ln -sfn ~/.agents/AGENTS.md ~/.claude/CLAUDE.md
```

Verify (run outside this repo so the project file doesn't interfere):

```sh
cd /tmp
claude -p "Without using any tools, quote verbatim the first line of the Git section from your user-level memory instructions, or say NONE."
omp -p "Without using any tools, quote verbatim the first sentence of the Git section in your user-level AGENTS.md instructions, or say NONE."
```

Both must print the Conventional Commits line.

## Live file warning

`~/.agents/AGENTS.md` resolves through this checkout's working tree, so whichever branch is checked out here is what every agent on the machine loads. A branch that edits or deletes `global/AGENTS.md` goes live the moment you switch to it. Edit on a branch, but return to `main` before starting agent sessions. If that becomes a habit problem, point the symlink at a separate `git worktree` pinned to `main`.

## Rules for the file

- A rule is added only if the agent can't infer it, it applies to every project, and an agent had to be corrected on it twice.
- State only what differs from harness defaults.
- 300 lines maximum, for the global file and for project files.
- No secrets, hostnames, or agent-written memory. The repo stays private.
- Every rule traces to a commit or an ADR.
- Prune on a major model change or when the file hits the line budget.

## Project template

`project/AGENTS.md` is the project template: a stack-neutral skeleton of placeholders. Copy it once into a new project and fill it in; the project owns the copy from then on.

```sh
cp /path/to/AGENTS.md/project/AGENTS.md ./AGENTS.md
```

- Copy, not import or symlink: a committed file is self-contained, and `@` imports from outside a repo prompt for approval in Claude Code and are not loaded by Copilot.
- `AGENTS.md` only, no `CLAUDE.md`: Claude Code (2.1.277+) loads a project `AGENTS.md` when no `CLAUDE.md` exists in or above the project; omp loads it natively. Checked with `claude -p` and `omp -p` in a fresh `git init` directory. Add `@AGENTS.md` in a `CLAUDE.md` only for an older Claude or a parent `CLAUDE.md`.
- No "Agent skills" block: the setup skill writes it per project.
- Placeholders are plain text, not HTML comments, which omp may load.
- Content follows the rules above: state only what differs from harness defaults. Keep a project file under 300 lines.
- New projects only; existing repos adopt it by choice.
