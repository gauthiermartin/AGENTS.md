# Global (user-level) agent instruction files: tool mechanics

Researched 2026-11-15 from vendor docs and source. Every claim carries a URL. `[INFERENCE]` marks my own reasoning; `[UNVERIFIED]` marks claims I could not confirm from a primary source. Docs and versions move fast (Claude Code v2.1.x, OpenCode V1→V2, Windsurf→Devin Desktop), so re-check before relying on edge cases.

## 1. The agents.md open spec

- Format: plain Markdown, "a README for agents". No required fields; any headings. Source: <https://agents.md/> (FAQ "Are there required fields?").
- Suggested sections (non-normative): project overview, build and test commands, code style, testing instructions, security considerations, commit/PR guidelines, deployment steps. Source: <https://agents.md/> ("Cover what matters", "Add extra instructions").
- Precedence: "The closest AGENTS.md to the edited file wins; explicit user chat prompts override everything." Nested files in monorepos are the only layering the spec describes. Source: <https://agents.md/> (FAQ "What if instructions conflict?", "Large monorepo?").
- **User/global level: the spec has none.** The site and README define only repo-root and nested files. Sources: <https://agents.md/>, <https://github.com/agentsmd/agents.md> (repo contents: README, site code, charter; no spec text for home-directory files). Global paths are tool-specific conventions (section 2).
- Migration and symlink guidance: `mv AGENT.md AGENTS.md && ln -s AGENTS.md AGENT.md`. Source: <https://agents.md/> (FAQ "How do I migrate existing docs").
- Per-tool config the spec site documents: Aider `read: AGENTS.md`; Gemini CLI `{"context":{"fileName":"AGENTS.md"}}`. Source: <https://agents.md/> (FAQ).
- Stewardship: Agentic AI Foundation under the Linux Foundation. Source: <https://agents.md/> ("About").

## 2. Per-tool mechanics

### Claude Code

Source for the whole section: <https://code.claude.com/docs/en/memory> unless noted.

- **Global file:** `~/.claude/CLAUDE.md` (scope "User instructions"). Also `~/.claude/rules/*.md` (modular, recursive). Managed policy: `/Library/Application Support/ClaudeCode/CLAUDE.md` on macOS.
- **Filenames:** `CLAUDE.md`, `.claude/CLAUDE.md`, `CLAUDE.local.md`. It reads `AGENTS.md` only as a project file (v2.1.277+) and only when no `CLAUDE.md`/`.claude/CLAUDE.md`/`CLAUDE.local.md` exists in the working directory or above. `~/.claude/CLAUDE.md` and `.claude/rules/` do not count for that check. It does not read `AGENTS.local.md`, `AGENTS.override.md`, or anything under `.agents/`. The docs list no `~/.agents/` or user-level `AGENTS.md` path.
- **Layering:** all discovered files are concatenated, not overridden. Order is managed → user → project (root down to cwd), `CLAUDE.local.md` last per directory. "If a user rule and a project rule conflict, Claude may follow either one."
- **Imports:** `@path` (relative to the importing file, absolute, `~/` as in `@~/.claude/my-project-instructions.md`), recursive to 4 hops, skipped inside code spans/fences. Imports in user-scope files (`~/.claude/CLAUDE.md`, `~/.claude/rules/`) load without the external-import approval dialog (except Cowork desktop sessions).
- **AGENTS.md sharing:** `CLAUDE.md` containing `@AGENTS.md` (plus Claude-specific text below it), or `ln -s AGENTS.md CLAUDE.md`. Symlink caveats: Edit/Write tools refuse to write through a symlink and redirect to the target; Windows needs admin/Developer Mode. In Cowork sessions a `~/.claude/CLAUDE.md` that is itself a symlink or hard link is skipped.
- **Size:** no hard limit documented. Guidance is "target under 200 lines per CLAUDE.md file"; imports do not reduce context cost. Auto memory loads the first 200 lines or 25KB (different mechanism).
- **Overrides:** no `.override` variant. `claudeMdExcludes` setting skips files by glob; `Project instructions` setting (`claude-md-or-agents-md` default, `claude-md-and-agents-md`, `claude-md`, `managed-only`).
- **Other:** block-level HTML comments are stripped before injection; `/context all` and `/doctor prompt-audit` inspect loaded files.

### OpenAI Codex CLI

Sources: <https://learn.chatgpt.com/docs/agent-configuration/agents-md.md>, <https://learn.chatgpt.com/docs/config-file/config-advanced.md> (the openai/codex repo's `docs/agents_md.md` only links to the former; the older URL <https://developers.openai.com/codex/guides/agents-md> serves the same guide).

- **Global file:** `$CODEX_HOME/AGENTS.md` (default `~/.codex/AGENTS.md`). `AGENTS.override.md` in the same dir wins if present. "Codex uses only the first non-empty file at this level."
- **Filenames:** per directory, project scope: `AGENTS.override.md`, then `AGENTS.md`, then names in `project_doc_fallback_filenames` (config.toml), at most one file per directory. Fallbacks are described for project directories, not for the global scope.
- **Layering:** global first, then project root → cwd; "Files closer to your current directory override earlier guidance because they appear later." Joined with blank lines. Project root = `.git` by default (`project_root_markers`).
- **Size:** `project_doc_max_bytes`, 32 KiB default; Codex "stops adding files once the combined size reaches the limit". Raise it or split across directories. Empty files are skipped.
- **Imports:** none documented. `[UNVERIFIED]` that `@path` is unsupported; the docs just never mention it.
- **Symlinks:** no guidance in these docs. `[UNVERIFIED]`.
- **Other:** `CODEX_HOME` relocates the whole home; project `.codex/` layers load only for trusted projects, but the docs do not say whether AGENTS.md is trust-gated. `[UNVERIFIED]`. Codex rebuilds the chain once per run/TUI session.

### Gemini CLI

Sources: <https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/cli/gemini-md.md>, <https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/reference/memport.md>, <https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/reference/configuration.md>.

- **Global file:** `~/.gemini/GEMINI.md`. The configuration reference says the global file is `~/.gemini/<configured-context-filename>`.
- **Filenames:** default `GEMINI.md`. `context.fileName` (string or array, default undefined) changes it, e.g. `["AGENTS.md","CONTEXT.md","GEMINI.md"]`. It is a user/project `settings.json` key.
- **Layering:** concatenation, in order: global → workspace dirs and ancestors (up to `.git` root or home) → just-in-time files discovered when tools touch a directory, up to a trusted root. `/memory show` and `/memory reload` inspect and rescan. `context.discoveryMaxDirs` default 200; `context.memoryBoundaryMarkers` configurable.
- **Imports:** `@file.md` with relative (`./`, `../`) and absolute paths, recursion default max depth 5, circular detection, ignored in code blocks. `validateImportPath` restricts imports to "specified directories"; `~/` is not listed as a supported path form. So whether `@~/.agents/AGENTS.md` works from `~/.gemini/GEMINI.md` is `[UNVERIFIED]`; an absolute path may be rejected by the allowed-directory check.
- **Size/truncation:** none documented in these pages.
- **Override variants / symlink guidance:** none documented.

### Cursor

Sources: <https://cursor.com/docs/rules>, <https://cursor.com/docs/cli/using>.

- **Global:** "User Rules" are defined in the app UI (Customize → Rules → global), not a documented file path. `[UNVERIFIED]` whether they live in a readable file on disk.
- **Project:** `.cursor/rules/*.mdc` (plain `.md` ignored), and `AGENTS.md` at project root and nested subdirectories ("combined with parent directories, with more specific instructions taking precedence"). No global `AGENTS.md` is documented.
- **CLI:** also reads project-root `AGENTS.md` and `CLAUDE.md` as rules.
- **Precedence:** Team Rules → Project Rules → User Rules, all merged; "earlier sources take precedence when guidance conflicts."
- **Imports:** none. A rule can mention `@file`, but Cursor states the file is not inlined; the agent reads it with tools on demand. User-level modular rules directory: none documented.
- **Size:** "Keep rules under 500 lines" (guidance, not enforced).

### GitHub Copilot

Sources: <https://docs.github.com/en/copilot/concepts/prompting-and-context/custom-instructions>, <https://docs.github.com/en/copilot/reference/custom-instructions-support>, <https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions>, <https://code.visualstudio.com/docs/agents/reference/custom-instructions-reference.md>.

- **Global files (CLI):** `$HOME/.copilot/copilot-instructions.md` and `$HOME/.copilot/instructions/**/*.instructions.md`. `COPILOT_HOME` relocates the dir. `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` (comma-separated) adds directories whose `AGENTS.md` and `*.instructions.md` are loaded.
- **Global (other surfaces):** personal instructions are a text box on github.com (Copilot Chat on the web) and in JetBrains; no file. VS Code lists `~/.copilot/...` as the user location for the Copilot harness.
- **Project filenames:** `.github/copilot-instructions.md`, `.github/instructions/**/*.instructions.md` (`applyTo` frontmatter), and agent instructions `AGENTS.md`, `CLAUDE.md`, `GEMINI.md` (CLI and cloud agent; in VS Code Copilot chat only `AGENTS.md`). `CLAUDE.md` also via `.claude/CLAUDE.md` in the CLI.
- **Layering:** the CLI "combines their instructions" and "does not define a general precedence order"; it removes duplicates of identical user-level, repo-wide and agent files. Only Copilot Chat on github.com defines order: personal > path-specific > repo-wide > agent > organization.
- **Imports:** CLI supports `@relative/path` in `.github/copilot-instructions.md`, `AGENTS.md`, `CLAUDE.md` (recursive). "Absolute paths and paths beginning with `~/` are not loaded"; references stay within the repo or the custom instructions directory. Not expanded in `GEMINI.md` or `*.instructions.md`.
- **Size/override variants/symlinks:** none documented. No `AGENTS.override.md`.

### OpenCode

Sources: <https://opencode.ai/docs/instructions.md> (V2, current), <https://opencode.ai/v1/docs/rules.md> (V1). The `/docs/rules` URL now serves V1 text with a pointer to V2.

- **Global file:** `~/.config/opencode/AGENTS.md` (both versions).
- **Layering (V2):** global first, then every `AGENTS.md` from cwd toward home or project root. "OpenCode combines the files and does not resolve conflicts between them." Nested files load as the agent reads below the workspace. Edits to global/upward files are picked up mid-session.
- **Filenames:** **V2 reads `AGENTS.md` only, no `CLAUDE.md` fallback.** V1: project `CLAUDE.md` if no `AGENTS.md`; global `~/.claude/CLAUDE.md` if no `~/.config/opencode/AGENTS.md`; disable via `OPENCODE_DISABLE_CLAUDE_CODE=1` / `OPENCODE_DISABLE_CLAUDE_CODE_PROMPT=1`. V1 first-match-wins per category.
- **Imports:** no `@` expansion in `AGENTS.md` (V1 states so explicitly and suggests telling the model to read `@file` lazily; V2 does not document it). Instead, the `instructions` array in `opencode.json[c]` (also global `~/.config/opencode/opencode.json`) accepts relative paths, globs, `~/` paths, absolute paths, and https URLs (5 s timeout), loaded after `AGENTS.md` files and live-updated.
- **Size/overrides/symlinks:** none documented. Symlinks are mentioned in V1 only as a way to share rule files across projects.

### Amp

Source: <https://ampcode.com/docs/markdown/customize/agents-md> (page `lastModified: 2026-11-13`).

- **Global files:** `$HOME/.config/amp/AGENTS.md` and `$HOME/.config/AGENTS.md`, both always included if present. System-wide: `/etc/ampcode/AGENTS.md`, `/Library/Application Support/ampcode/AGENTS.md`, `%ProgramData%\ampcode\AGENTS.md`. Also editable in the web app (Settings → Instructions → Global AGENTS.md); workspace admins have a workspace-level one, included first. Personal and workspace global files also apply to Claude Code agent threads, not to subagents/Puck.
- **Filenames:** `AGENTS.md`; if a directory has none, `AGENT.md` or `CLAUDE.md` is used instead.
- **Layering:** cwd/workspace roots and parents up to `$HOME` always included; subtree files when the agent reads a file there. Workspace guidance precedes personal and repo guidance.
- **Imports:** `@file` mentions (relative to the mentioning file; `@~/some/path`; globs allowed; absolute paths not supported; ignored in code blocks). Mentioned files with `globs:` YAML frontmatter apply only after Amp reads a matching file, so this doubles as a modular-rules mechanism at user level via `@~/...`.
- **Ignore:** `AMP_IGNORE_GUIDANCE_FILES` (colon-separated globs, `*` for all).
- **Size/override variants/symlinks:** none documented.

### Windsurf (now Devin Desktop) / Devin CLI

Docs have migrated to Devin; the old docs.windsurf.com paths redirect. Sources: <https://docs.devin.ai/windsurf/plugins/cascade/memories.md> (Windsurf Cascade), <https://docs.devin.ai/cli/extensibility/rules.md> and <https://docs.devin.ai/desktop/devin-local/customizations.md> (Devin harness).

- **Windsurf Cascade global:** `global_rules.md` ("rules applied across all workspaces"). The docs do not give the filesystem path. `~/.codeium/windsurf/memories/global_rules.md` appears only in community write-ups (<https://markdific.com/guides/windsurf-rules>, <https://agentdex.app/blog/windsurf-rules/>); `[UNVERIFIED]` against vendor docs. Rule files are limited to 12000 characters each (same Windsurf page; the community 6,000-char figure for the global file is `[UNVERIFIED]`). Workspace: `.windsurf/rules/`. System-level (enterprise): `/Library/Application Support/Windsurf/rules/*.md` on macOS.
- **Windsurf AGENTS.md:** directory-scoped instructions that apply by file location (page summary seen in search results only; the page itself would not load): <https://docs.windsurf.com/windsurf/cascade/agents-md>. A global `AGENTS.md` for Cascade is not documented.
- **Devin CLI / Devin Local (the harness Devin Desktop shares):** global `~/.config/devin/AGENTS.md` (`%APPDATA%\devin\AGENTS.md` on Windows); also `~/.devin/rules/*.md` and `~/.devin/global_rules.md`. Reads `AGENTS.md`, `AGENTS.local.md`, `AGENT.md`, `.windsurfrules`, `CLAUDE.md`. "If you use Claude Code, Devin CLI also reads `~/.claude/CLAUDE.md` as a global rule." Global and project rules are both active. `.devin/` beats `.windsurf/` when both `global_rules.md` exist. `read_config_from` in `config.json` toggles `agents_standard`, `cursor`, `windsurf`, `claude`. Skills read `~/.agents/skills/` (skills only, not instruction files).
- **Imports / size / symlinks:** none documented. Devin docs advise keeping rules small and preferring skills.

### omp (oh-my-pi) and Pi

omp (fork of Pi): <https://github.com/can1357/oh-my-pi/blob/main/docs/context-files.md>. Pi: <https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/configuration.md>. (Pi lives at earendil-works/pi; <https://github.com/badlogic/pi-mono> is the former home.)

**omp**
- **Global file:** `~/.omp/agent/AGENTS.md` (native, priority 100; `PI_CODING_AGENT_DIR` relocates; profiles use `~/.omp/profiles/<name>/agent`). Also `~/.omp/agent/RULES.md` (always-apply "sticky" rules carried on every request).
- **Other user-level sources:** `agents` provider reads `~/.agents/AGENTS.md` and `~/.agent/AGENTS.md`, enabled by default. `claude` (`~/.claude/CLAUDE.md`), `codex` (`~/.codex/AGENTS.md`), `gemini` (`~/.gemini/GEMINI.md`), `opencode` (`~/.config/opencode/AGENTS.md`), `github` (`~/.copilot/copilot-instructions.md`) are opt-in via `enabledProviders`.
- **Layering:** exactly **one** user-level context file survives across all providers (highest priority wins, so `~/.omp/agent/AGENTS.md` shadows `~/.agents/AGENTS.md`). One project file per directory depth, higher-priority provider shadowing the rest; multiple depths all load. Injection order: user file first, then project root → cwd (later = more prominent). Task subagents do not receive `AGENTS.md` files (but do get `RULES.md`).
- **Imports:** `@path` expanded inline; relative to the importing file, `~/`, absolute; skips code spans/fences; recursion up to 5 hops; cycles skipped; missing targets leave the literal token.
- **Override/extra:** `contextFiles.extra` (e.g. `AGENTS.local.md`); no `AGENTS.override.md` in omp docs. Size limits: none documented.

**Pi**
- **Global file:** `<agent-dir>/AGENTS.override.md`, `AGENTS.md`, `AGENTS.MD`, `CLAUDE.md`, or `CLAUDE.MD`; agent dir default `~/.pi/agent`, override with `PI_CODING_AGENT_DIR`. Context files also load from the working directory and its parents. `AGENTS.override.md` replaces `AGENTS.md`/`CLAUDE.md` in the same directory only. `SYSTEM.md` / `APPEND_SYSTEM.md` are separate.
- **Imports, size limits, symlinks:** not documented in the pages I read. `[UNVERIFIED]`.

## 3. Compatibility matrix

"Native" = documented as read without extra config. `n/d` = not documented in the sources above.

| Tool | Global path (documented) | Reads AGENTS.md natively | Reads CLAUDE.md | Imports (`@`) | Size limit |
|---|---|---|---|---|---|
| Claude Code | `~/.claude/CLAUDE.md`, `~/.claude/rules/*.md` | Project only, and only when no `CLAUDE.md` in/above cwd; no user-level `AGENTS.md` | Yes (native) | Yes; relative/abs/`~`, 4 hops | None hard; advise <200 lines |
| Codex CLI | `~/.codex/AGENTS.md` (`AGENTS.override.md` wins); `$CODEX_HOME` | Yes | No (only via `project_doc_fallback_filenames`, project scope) | n/d | 32 KiB combined (`project_doc_max_bytes`) |
| Gemini CLI | `~/.gemini/GEMINI.md` (name via `context.fileName`) | Only if `context.fileName` includes it | Only if configured | Yes; relative/abs, depth 5; `~` unverified | n/d |
| Cursor | None file-based documented (UI "User Rules") | Project/nested only; no global | CLI: project-root `CLAUDE.md` | No (mentions are lazy reads) | 500 lines advised |
| Copilot (CLI) | `~/.copilot/copilot-instructions.md`, `~/.copilot/instructions/**/*.instructions.md`; `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` | Project yes; global only through `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` | Yes (project) | Yes, but `~/` and absolute paths are not loaded | n/d |
| OpenCode V2 | `~/.config/opencode/AGENTS.md` | Yes | **No** (V1: fallback `~/.claude/CLAUDE.md`) | No in `AGENTS.md`; `instructions` array takes `~/`, globs, URLs | n/d |
| Amp | `~/.config/amp/AGENTS.md`, `~/.config/AGENTS.md` | Yes | Fallback per directory when no `AGENTS.md` (project); global n/d | Yes; `@~/`, globs, no absolute | n/d |
| Windsurf / Devin | Windsurf: `global_rules.md` (path n/d in vendor docs); Devin: `~/.config/devin/AGENTS.md`, `~/.devin/global_rules.md` | Devin yes; Windsurf project-scoped | Devin: `~/.claude/CLAUDE.md` + project `CLAUDE.md` | n/d | Windsurf 12000 chars per rule file |
| omp | `~/.omp/agent/AGENTS.md`; also `~/.agents/AGENTS.md` by default | Yes | Opt-in for user-level (`enabledProviders: [claude]`); project yes | Yes; relative/abs/`~`, 5 hops | n/d |
| Pi | `~/.pi/agent/AGENTS.md` (or `CLAUDE.md`, `AGENTS.override.md`) | Yes | Yes | n/d | n/d |

### Modular / imports / rules directories at user level (question 4)

- **User-level modular directory:** Claude Code `~/.claude/rules/` (recursive; `paths` frontmatter globs); Copilot CLI `~/.copilot/instructions/**/*.instructions.md` (`applyTo`); Devin CLI `~/.devin/rules/*.md` (`trigger` frontmatter). Windsurf global is a single `global_rules.md`; Cursor user rules are UI-only; Codex, Gemini, OpenCode, Amp, Pi, omp have no such directory (omp has `RULES.md` plus skills).
- **`@` imports usable from a user-level file:** Claude Code, Gemini CLI (with path-validation caveat), Amp, omp. Copilot CLI supports `@` but refuses `~/` and absolute paths. OpenCode uses config `instructions` instead. Codex, Cursor, Pi: none documented.
- **Dynamic globs through imports:** Amp (`globs:` frontmatter on `@`-mentioned files).

## 4. Implications for a single source-of-truth global file

Keep the two tiers separate. Everything under "Documented" cites sections 2-3. Everything under "Inference" is my reasoning.

### Documented facts

1. There is **no cross-tool global path**. The agents.md spec defines no user level, and each tool picks its own directory (`~/.claude`, `~/.codex`, `~/.gemini`, `~/.config/opencode`, `~/.config/amp`, `~/.copilot`, `~/.omp/agent`, `~/.pi/agent`).
2. `~/.agents/` is a real convention in exactly two places I could verify: omp reads `~/.agents/AGENTS.md` and `~/.agent/AGENTS.md` by default (<https://github.com/can1357/oh-my-pi/blob/main/docs/context-files.md>), and Devin reads `~/.agents/skills/` (skills only, not AGENTS.md) (<https://docs.devin.ai/desktop/devin-local/customizations.md>). Claude Code explicitly does **not** read anything under `.agents/` as instructions (<https://code.claude.com/docs/en/memory>).
3. Tools with a documented way to pull a file from an arbitrary location:
   - Claude Code: `@~/...` import inside `~/.claude/CLAUDE.md`, no approval dialog at user scope.
   - Amp: `@~/...` mention inside `~/.config/amp/AGENTS.md`.
   - omp: `@~/...` import inside any context file, or `~/.agents/AGENTS.md` directly.
   - OpenCode: `instructions: ["~/.agents/AGENTS.md"]` in `~/.config/opencode/opencode.jsonc`.
   - Copilot CLI: `COPILOT_CUSTOM_INSTRUCTIONS_DIRS=$HOME/.agents` makes it load `AGENTS.md` from that directory.
   - Gemini CLI: `context.fileName` makes `~/.gemini/AGENTS.md` the global file (name only, not location).
   - Codex: relocate with `CODEX_HOME`, which moves the entire Codex home, not just the instructions file.
4. Symlink facts: the Claude Code docs say a `CLAUDE.md` symlink to `AGENTS.md` works for reading but Edit/Write refuse to write through it, and in Cowork sessions a symlinked `~/.claude/CLAUDE.md` is skipped. The agents.md FAQ itself recommends a symlink for migration. No other tool documents symlink behavior (<https://code.claude.com/docs/en/memory>, <https://agents.md/>).
5. Collisions to avoid: Codex, omp and Pi keep only the first non-empty/highest-priority user file, so pointing several mechanisms at the same content is safe but one winner is chosen (Codex: override then AGENTS.md; omp: one user file across all providers, which shadows `~/.agents/AGENTS.md` if `~/.omp/agent/AGENTS.md` exists). Copilot CLI de-duplicates identical user-level files; Claude Code's `claude-md-and-agents-md` mode skips an `AGENTS.md` already loaded via import.
6. Size: the only hard cap documented is Codex's 32 KiB combined (`project_doc_max_bytes`). Claude advises <200 lines; Cursor <500 lines; Windsurf 12000 chars per rule file. Several vendors recommend small instruction files and skills for the rest (<https://docs.devin.ai/cli/extensibility/rules.md>, <https://code.claude.com/docs/en/memory>).
7. Tool-specific features that a shared file cannot carry: Claude `paths`-scoped rules, Copilot `applyTo`, Cursor `.mdc` frontmatter, Amp `globs`, Devin `trigger`, omp `RULES.md`. These are per-tool and stay outside the shared file.

### Inference `[INFERENCE]`

- **Canonical location:** `~/.agents/AGENTS.md` is the best-fit canonical file because omp reads it natively and every other tool can reach it with one small piece of per-tool glue. There is no vendor reason to prefer another path; the choice is about neutrality. A repo that version-controls the global file (this repo) can hold the source and have `~/.agents/AGENTS.md` be a symlink or copy of it.
- **Per-tool glue, cheapest first:**
  - Codex: `~/.codex/AGENTS.md` as a symlink to `~/.agents/AGENTS.md` (Codex documents no symlink handling, so test with the "Summarize the current instructions" check from the Codex docs). Alternative that avoids symlinks: none documented.
  - Claude Code: `~/.claude/CLAUDE.md` containing `@~/.agents/AGENTS.md` plus Claude-only notes. Prefer this over a symlink: reading and writing both behave, and the Cowork skip rule does not bite.
  - Gemini CLI: `context.fileName: ["AGENTS.md","GEMINI.md"]` in `~/.gemini/settings.json` and `~/.gemini/AGENTS.md` as symlink/copy; avoid depending on `@~` imports until verified.
  - OpenCode: `instructions` entry in the global config; no file duplication.
  - Amp: `~/.config/amp/AGENTS.md` as symlink, or a one-line `@~/.agents/AGENTS.md` file.
  - Copilot CLI: `COPILOT_CUSTOM_INSTRUCTIONS_DIRS=$HOME/.agents` in the shell profile. Copilot on github.com and JetBrains is UI-only, so it cannot be fed from a file.
  - Cursor: nothing file-based at user level is documented; UI paste only. Cursor will see only project-root `AGENTS.md`.
  - Windsurf/Devin: `~/.config/devin/AGENTS.md` as symlink. Windsurf Cascade's own global file is `global_rules.md` with a 12000-char cap; whether Devin Desktop's local agent honors `~/.config/devin/AGENTS.md` follows from "Devin Local shares its harness with Devin CLI" but I did not see it stated for global rules `[UNVERIFIED]`.
  - Pi: `~/.pi/agent/AGENTS.md` as symlink.
- **Why a copy-on-sync script may beat symlinks:** only Claude Code documents symlink edge cases, and they involve its write tools and Cowork. Untested tools could choke on symlinks (permissions, trust, realpath checks). A tiny sync script that copies or links, and a verification step per tool, is safer than trusting undocumented behavior. Windows users cannot rely on symlinks (documented for Claude Code; same OS limitation applies generally).
- **Content to keep in the shared file:** only guidance that is true for every tool and has no tool-specific syntax. Keep it small: the strictest documented limits are Codex 32 KiB and Claude's 200-line advice, and instructions are loaded into every session's context.
- **Tool names in text:** a shared file will be read by agents with different tool sets, so refer to behaviors, not tool names, or add a short per-tool section. omp injects `AGENTS.md` into a "MUST follow" block but task subagents drop it, so anything subagents must obey belongs in `RULES.md` there.
- **Precedence risk:** since most tools concatenate user and project guidance without defined precedence (Claude, Copilot CLI, OpenCode), the shared file should avoid hard rules that projects may legitimately contradict, or state "project files override these defaults" explicitly.
- **Verification step to add to the repo:** per tool, run a "list loaded instruction files" check (Claude `/context all`, Codex "Summarize the current instructions", Gemini `/memory show`, Copilot `/instructions`, Pi startup header, omp injected `<repo-rules>`), after every global change.

## 5. Gaps still open

- Windsurf global rules filesystem path: absent from vendor docs I could load. The old docs.windsurf.com pages redirect to Devin pages.
- Codex: imports, symlinks, trust gating for AGENTS.md, and whether `~/.agents/AGENTS.md` is ever read (the docs list only `CODEX_HOME`).
- Gemini: `~` support and allowed-directory rules for `@` imports from a global file; any size cap.
- Cursor: any on-disk representation of User Rules; global `AGENTS.md`.
- Pi: imports and size limits.
- Copilot: symlink behavior; any size cap for personal instructions.
