# AGENTS.md

Version-controlled source of my global (user-level) agent instructions, with the research and decisions behind them.

Agents served: Claude Code and omp.

## Layout

- `global/AGENTS.md`: the global file. This is the only thing agents load.
- `AGENTS.md`: this repo's own project instructions (not global).
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

## Rules for the file

- A rule is added only if the agent can't infer it, it applies to every project, and an agent had to be corrected on it twice.
- State only what differs from harness defaults.
- 100 lines maximum.
- No secrets, hostnames, or agent-written memory. The repo stays private.
- Every rule traces to a commit or an ADR.
- Prune on a major model change or when the file hits the line budget.
