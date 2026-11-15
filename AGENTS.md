## Agent skills

### Issue tracker

Issues live in GitHub Issues (`gauthiermartin/AGENTS.md`), via the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Default vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `GLOSSARY.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.

## Repo rules

`global/AGENTS.md` is the user-level file loaded in every project; this file is not. Rules for editing `global/AGENTS.md` are in `README.md`: 400 lines max, no secrets, hostnames, or agent-written memory. The repo stays private.
