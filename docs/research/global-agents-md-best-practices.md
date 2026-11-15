# Best practices for a global (user-level) AGENTS.md

Research date: 2026-11-15. Scope: guidance and evidence for writing and maintaining a *user-level* instruction file shared by several coding agents (target: `~/.agents/AGENTS.md`).
Tool-specific loading mechanics (which agent reads which path) are out of scope here; see the sibling tool-mechanics research.

**Source rules.** Primary sources only: vendor docs, spec site, papers, vendor security disclosures. Every claim has a URL. Anything not verified is marked **[unverified]**. Anything I conclude myself is marked **[inference]**. Community or vendor-marketing material is labelled **[community]**.

**Headline caveat.** No vendor doc or paper I found studies *global/user-level* files. All empirical evidence below is about *repository-level* files. Applying it to a global file is an extrapolation **[inference]**.

---

## 1. What official docs recommend

### 1.1 Cross-vendor consensus

| Theme | Guidance | Source |
|---|---|---|
| Write only what the agent can't discover | Claude: "Only write down what Claude can't find by reading the repo, or what it would otherwise spend time rediscovering every session." Test: could it find this by reading the code? would it actually go look? If both yes, leave it out. | [Claude Code best practices](https://code.claude.com/docs/en/best-practices.md) |
| | AAIF: "For each instruction, ask whether knowing it would change what the agent does." | [AAIF, Writing an effective AGENTS.md](https://aaif.io/blog/writing-an-effective-agents-md) |
| | Cursor "avoid": copying whole style guides (use a linter), documenting every command, rare edge cases, duplicating the codebase. | [Cursor rules](https://cursor.com/docs/rules) |
| Keep it short | Claude: "target under 200 lines per CLAUDE.md file. Longer files consume more context and reduce adherence." | [Claude memory](https://code.claude.com/docs/en/memory.md) |
| | Codex: "A short, accurate AGENTS.md is more useful than a long file full of vague rules." Reference task-specific files when it grows. | [Codex best practices](https://learn.chatgpt.com/guides/best-practices) |
| | Cursor: keep rules under 500 lines, split into composable rules. | [Cursor rules](https://cursor.com/docs/rules) |
| | Copilot: "short, self-contained statements"; they are "sent with every chat message", so should be "broadly applicable to most requests". | [Copilot custom instructions](https://docs.github.com/en/copilot/concepts/prompting-and-context/custom-instructions) |
| | Gemini: "Avoid adding excessive content to GEMINI.md. Keep instructions actionable." | [Gemini CLI memory tutorial](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/cli/tutorials/memory-management.md) |
| Be specific and verifiable | Claude: "Use 2-space indentation" not "Format code properly"; "Run `npm test` before committing" not "Test your changes". | [Claude memory](https://code.claude.com/docs/en/memory.md) |
| | AAIF table of vague → usable rewrites (which tests, where, when; name generated paths and what to edit instead). | [AAIF](https://aaif.io/blog/writing-an-effective-agents-md) |
| Add rules reactively | Claude: add when Claude makes the same mistake a second time, review catches something, or you retype the same correction. | [Claude memory](https://code.claude.com/docs/en/memory.md) |
| | Codex: "add new rules only after you notice repeated mistakes"; after a repeated mistake ask for a retrospective and update the file. | [Codex best practices](https://learn.chatgpt.com/guides/best-practices) |
| | Cursor: "Start simple. Add rules only when you notice Agent making the same mistake repeatedly." | [Cursor rules](https://cursor.com/docs/rules) |
| | Anthropic: start with a minimal prompt, then add instructions "based on failure modes found during initial testing". "Minimal does not necessarily mean short". | [Effective context engineering](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents) |
| No contradictions | Claude: if two instructions contradict, "Claude may pick one arbitrarily"; user and project rules that conflict: "Claude may follow either one, so keep the two consistent." | [Claude memory](https://code.claude.com/docs/en/memory.md) |
| | Copilot: "avoid providing conflicting sets of instructions"; all levels are sent. | [Copilot custom instructions](https://docs.github.com/en/copilot/concepts/prompting-and-context/custom-instructions) |
| Instructions are advisory, not enforcement | Claude: CLAUDE.md is "context, not enforced configuration"; delivered "as a user message after the system prompt"; for must-never/must-always use hooks or deny rules. | [Claude memory](https://code.claude.com/docs/en/memory.md), [best practices](https://code.claude.com/docs/en/best-practices.md) |
| | Copilot: "custom instructions do not guarantee that generated content will comply with a policy". | [Copilot custom instructions](https://docs.github.com/en/copilot/concepts/prompting-and-context/custom-instructions) |
| | AAIF: don't let AGENTS.md "be the only thing preventing an invalid change"; move formatter/static-analysis-enforceable rules out of prose. | [AAIF](https://aaif.io/blog/writing-an-effective-agents-md) |

### 1.2 What to include (per vendor)

- **Claude Code:** exact test/typecheck/lint commands and how to run one test; places where code is misleading; decisions not in the repo; anything corrected twice. Multi-step procedures or part-of-codebase-only rules → skills or path-scoped rules, not CLAUDE.md. ([best practices](https://code.claude.com/docs/en/best-practices.md), [memory](https://code.claude.com/docs/en/memory.md))
- **Codex:** repo layout, how to run, build/test/lint commands, conventions and PR expectations, do-not rules, "what done means and how to verify work". ([Codex best practices](https://learn.chatgpt.com/guides/best-practices)) The Codex guide's own *global* example is three "working agreements" (run tests after JS edits, prefer pnpm, ask before adding production dependencies). ([Codex AGENTS.md guide](https://learn.chatgpt.com/docs/agent-configuration/agents-md.md))
- **agents.md spec:** no required fields; plain Markdown; suggested sections: project overview, build/test commands, code style, testing, security. Closest file wins on conflict; explicit user chat prompts override everything. ([agents.md](https://agents.md/))
- **Copilot:** personal instructions examples are *response preferences* (language, tone, "Keep explanations brief"). ([Copilot personal instructions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-personal-instructions)) Warns that style-of-answer, always-this-detail-level, and "refer to external resource" instructions "may cause problems" in large/diverse repos. ([custom instructions](https://docs.github.com/en/copilot/concepts/prompting-and-context/custom-instructions))
- **Gemini CLI:** global `~/.gemini/GEMINI.md` = "default instructions for all your projects"; tutorial example of a global rule is "Always use strict typing". ([GEMINI.md docs](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/cli/gemini-md.md), [tutorial](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/cli/tutorials/memory-management.md)) Also recommends negative constraints ("Do not use class components") as "often more effective than vague positive instructions" (vendor claim, no data given).
- **Cursor:** "User Rules" are "Global to your Cursor environment" and used by Agent (Chat); the doc gives no user-rule-specific writing advice. ([Cursor rules](https://cursor.com/docs/rules)) Whether Cursor user rules live in a file that could be synced from `~/.agents` **[unverified]**.

### 1.3 Global vs project (what each scope is *for*)

- Claude: user file = "Personal preferences for all projects. Code styling preferences, personal tooling shortcuts"; project file = team-shared architecture/standards/workflows; local file = personal per-project. Loaded broadest → most specific so project text appears later in context. ([Claude memory](https://code.claude.com/docs/en/memory.md))
- Claude: "Neither set overrides the other" between user and project rules. ([Claude memory](https://code.claude.com/docs/en/memory.md)) Codex instead says files closer to cwd override earlier guidance because they appear later in the merged prompt. ([Codex AGENTS.md guide](https://learn.chatgpt.com/docs/agent-configuration/agents-md.md)) Copilot Chat on github.com ranks personal > repository > organization. ([Copilot](https://docs.github.com/en/copilot/concepts/prompting-and-context/custom-instructions))  
  → The three vendors define precedence differently; a global file should not depend on winning conflicts **[inference]**.
- Codex: global file is "persistent defaults ... so every repository inherits your working agreements"; combined size cap `project_doc_max_bytes` 32 KiB by default, then silent truncation. ([Codex AGENTS.md guide](https://learn.chatgpt.com/docs/agent-configuration/agents-md.md)) Global + project + nested share that budget.
- Gemini memory tool: routes "cross-project personal preferences" to global `~/.gemini/GEMINI.md`, repo-shared facts to repo `GEMINI.md`. ([Gemini memory tool](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/tools/memory.md))
- Copilot: "Consider the scope over which you want an instruction to apply before deciding whether to add it at the personal, repository, or organization level." ([Copilot](https://docs.github.com/en/copilot/concepts/prompting-and-context/custom-instructions))

### 1.4 Structure and formatting

- Claude: group under headers and bullets; "Organized sections are easier for Claude to follow than dense paragraphs." ([memory](https://code.claude.com/docs/en/memory.md))
- Anthropic: delineate sections with Markdown headers or XML tags, "although the exact formatting of prompts is likely becoming less important as models become more capable". ([context engineering](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents))
- Claude: block-level HTML comments in CLAUDE.md are stripped before injection, so maintainer notes cost no tokens (comments inside code blocks are preserved). ([memory](https://code.claude.com/docs/en/memory.md)) Whether other agents strip them **[unverified]**.
- Imports (`@path`) help organisation "but don't reduce its context cost". ([memory](https://code.claude.com/docs/en/memory.md)) Cursor mentions in rules are *not* inlined; the agent reads them on demand. ([Cursor rules](https://cursor.com/docs/rules))
- GitHub blog analysis of 2,500+ agents.md files: commands early, one code example over three paragraphs, explicit boundaries (always / ask first / never), six areas (commands, testing, structure, style, git workflow, boundaries). **[community-ish: vendor-authored, descriptive observation, no outcome data]** ([GitHub blog](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)). Note this is about Copilot custom-agent personas; its scope is narrower than a global file.

### 1.5 Tone and emphasis

- Anthropic: "If you keep adding emphasis to a CLAUDE.md line and Claude still misses it, that's a sign the line belongs in a hook." Adding IMPORTANT "probably won't fix it": either the file is too long or the rule can't have exceptions. ([best practices](https://code.claude.com/docs/en/best-practices.md))
- Anthropic (prompting guide, Opus 4.5/4.6 era): newer models are more responsive to system prompts; aggressive language that fixed undertriggering "might now overtrigger". "Where you might have said 'CRITICAL: You MUST use this tool when...,' you can use more normal prompting such as 'Use this tool when...'". "Tune anti-laziness prompting" downward for 4.6. ([Claude prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices.md)) This is API-prompt guidance applied by analogy to CLAUDE.md **[inference]**.
- Anthropic: give the *reason* for a rule ("Claude can generalize from the explanation"); example: say the text will be read by a TTS engine instead of "NEVER use ellipses". Test: show it to a colleague with no context. ([prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices.md))
- Anthropic: aim for "the right altitude": specific enough to guide, flexible enough to give strong heuristics; avoid hardcoded brittle logic and avoid vague guidance; curate "diverse, canonical examples" instead of a "laundry list of edge cases". ([context engineering](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents))
- Cursor: "Avoid vague guidance. Write rules like clear internal docs"; "Provide concrete examples or referenced files". ([Cursor rules](https://cursor.com/docs/rules))
- Claude: `/doctor prompt-audit` flags "instructions written for older models". ([memory](https://code.claude.com/docs/en/memory.md)) Instruction wording tuned to one model generation can rot as models change **[inference]**.

### 1.6 What *not* to put in the file (consolidated)

- Facts the agent finds by reading the repo: directory trees, dependency lists, architecture overviews (Claude `/doctor` trims these: [memory](https://code.claude.com/docs/en/memory.md)). 
- Generic advice with no repo-specific consequence ([AAIF](https://aaif.io/blog/writing-an-effective-agents-md)).
- Formatter/linter-enforceable style ([Cursor](https://cursor.com/docs/rules), [AAIF](https://aaif.io/blog/writing-an-effective-agents-md)).
- Long task-specific procedures → skills (loaded on demand) ([Claude best practices](https://code.claude.com/docs/en/best-practices.md)).
- Credentials, secrets, private operational details ([AAIF](https://aaif.io/blog/writing-an-effective-agents-md)).
- Hard rules that must never be violated → hooks/permissions, not prose ([Claude best practices](https://code.claude.com/docs/en/best-practices.md)).
- Instructions that duplicate or conflict with the harness's own built-ins (e.g. git/PR rules vs Claude Code's defaults; Claude has `includeGitInstructions` to turn off its own) ([memory troubleshooting](https://code.claude.com/docs/en/memory.md)).

---

## 2. Empirical evidence: do context files help or hurt?

### 2.1 Gloaguen et al., "Evaluating AGENTS.md" (ETH Zurich / LogicStar; arXiv 2602.11988 v3, 29 Sep 2026)

Source: [arXiv HTML v3](https://arxiv.org/html/2602.11988v3) · [abs](https://arxiv.org/abs/2602.11988). Venue as listed on the SRI Lab page: MemAgents @ ICLR 2026 (oral) ([SRI Lab](https://www.sri.inf.ethz.ch/publications/gloaguen2026agentsmd)); peer-review status beyond workshop **[unverified]**.

**Design.**
- Benchmarks: SWE-bench Lite (300 tasks, 11 popular Python repos, none with developer context files) and new CTXbench (138 instances from 12 niche Python repos, all with developer-committed context files; tasks standardised by an LLM, 25/138 tests hand-edited).
- Agents: Claude Code + Sonnet-4.5, Codex + GPT-5.2, Codex + GPT-5.1 mini, Qwen Code + Qwen3-30B-Coder. One sample per agent per instance; temperature 0 (0.7 for Qwen).
- Conditions: None; LLM-generated (agent's own `/init`-style recommended procedure); Dev (committed file, CTXbench only).
- Mean developer context file: 641 words, 9.7 sections (min 24, max 2003 words).

**Results.**
- Resolution rate: LLM-generated files changed it by −0.5 pts (SWE-bench) and −2 pts (CTXbench) on average; dropped in 5 of 8 settings. p = 0.87 and 0.37 (CMH test) → no significant effect. Dev files: +2.4 pts avg, p = 0.21 (not significant), but significantly better than LLM-generated (p = 0.038). Dev files improved all agents except Claude Code.
- Cost/steps: every setting got more steps; LLM-generated files raised cost +20% (SWE-bench) / +23% (CTXbench), p < 0.001. Dev files: +3.34 steps and up to +19% cost (CTXbench, p = 0.0126). Raw success rates, per-agent (Table 5): e.g. Sonnet-4.5 on SWE-bench 59.9% none vs 58.7% LLM.
- Behaviour: instructions are followed. `uv` used 1.6×/instance when mentioned vs <0.01× when not; repo-specific tools 2.5× vs <0.05×. Agents ran more tests, searched/read/wrote more files. Reasoning tokens +22% (GPT-5.2, SWE-bench) with LLM files. The authors conclude no lack of instruction-following explains the null result; they hypothesise extra instructions make the task harder.
- Repo overviews did not help: 8 of 12 dev files had one; 95–100% of LLM files (except GPT-5.1 mini, 36%) did; no meaningful reduction in steps-to-first-relevant-file.
- Ablations: stronger model generating the file did not help (+2% SWE-bench, −3% CTXbench); generation prompt choice no consistent effect; **file length not correlated with success or cost**; removing overview/tooling/testing sections did not significantly change accuracy (removing testing/tooling sections reduced cost significantly); **when all other docs (`*.md`, docs/, examples) were removed, LLM-generated files improved results by +2.7% and beat dev files** → context files are largely redundant with existing docs.
- Authors' recommendations: omit LLM-generated context files for now; human-written files should hold only instructions not already in the README (specific conventions, non-functional requirements) and be "rigorously evaluated before adoption".

**Caveats (mine unless stated).**
- Repository-level files only; no global/user-level condition.
- Python only (authors: training-data coverage may mask benefit for niche languages).
- Task = issue resolution scored by tests; does not measure style adherence, security, review burden, or user-preference satisfaction (authors flag security/efficiency as future work) — exactly what a personal global file mostly targets **[inference]**.
- n = 138 / 300, one run each, mostly non-significant differences; "no significant effect" ≠ "no effect".
- Models are Sonnet-4.5/GPT-5.2-class; current models (see Claude docs above) differ **[inference]**.
- The paper (and AAIF's summary of it) say context files increased cost; part of that is the agent *doing what the file says* (more testing), which may be desired.

### 2.2 Lulla et al., "On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents" (arXiv 2601.20404, 28 Jan 2026)

Source: [arXiv](https://arxiv.org/abs/2601.20404). 10 repos, 124 PRs, agents run with vs without AGENTS.md (abstract names Codex and Claude Code as examples of such agents, not necessarily the ones tested **[unverified]**). With the file: median runtime −28.64%, output tokens −16.58%, "comparable task completion behavior". Measured efficiency only, not patch correctness (AAIF's reading: [AAIF](https://aaif.io/blog/writing-an-effective-agents-md)). Which agents/model versions were used and how "comparable" completion was assessed **[unverified; I read only the abstract]**.

*Tension with 2.1:* Lulla finds efficiency gains, Gloaguen finds cost increases. Different repos, agents, metrics (wall-clock/output tokens vs total inference cost/steps), and file provenance **[inference]**; neither study isolates content type except Gloaguen's ablations.

### 2.3 AAIF five-run experiment (one repo, one agent)

Per AAIF: on the more complex task, 2 of 5 runs without a short AGENTS.md reoriented or ran an unrequested production build; 0 of 5 with it. AAIF itself calls this "useful evidence rather than a general benchmark". ([AAIF](https://aaif.io/blog/writing-an-effective-agents-md)) I did not read the underlying post **[unverified]**.

### 2.4 Chatlatanagulchai et al., "Agent READMEs" (arXiv 2511.12884)

Descriptive, not outcome-based. 2,303 context files from 1,925 repos. Files "evolve like configuration code through frequent, small additions", are "complex, difficult-to-read artifacts". Content: test procedures 75.9%, implementation details 70.8%, architecture 68.1%; security 14.8%, performance 14.5%. ([arXiv](https://arxiv.org/abs/2511.12884)) Useful as a baseline of what people write, not of what works.

### 2.5 Instruction count / context length degradation

- **IFScale** (Jaroslawicz et al., Distyl AI, arXiv 2507.11538): 20 models, 7 providers; 500 keyword-inclusion instructions in a report-writing task; 5 seeds per density (10–500, step 10). Best frontier model 68% accuracy at 500. Three patterns: threshold decay (reasoning models like o3, gemini-2.5-pro near-perfect to ~100–250 then decline), linear decay (gpt-4.1, claude-sonnet-4: 42.9% at 500), exponential decay (small models, floor 7–15%). Primacy bias (earlier instructions obeyed more) peaks around 150–200 instructions. Errors are mostly omissions. Variance rises for top models as density grows. **Caveat:** synthetic keyword-inclusion in English business reports; authors say it may not generalise; models are mid-2025. ([arXiv HTML](https://arxiv.org/html/2507.11538))
- **Context rot** (Chroma technical report, July 2025; not peer-reviewed **[community-ish: vendor technical report]**): 18 LLMs; performance becomes less reliable as input length grows even on simple tasks; distractors hurt more at longer lengths; lower needle-question similarity degrades faster. ([Chroma](https://research.trychroma.com/context-rot)) Anthropic cites this and concludes context is "a finite resource with diminishing marginal returns" and that good context engineering finds "the smallest possible set of high-signal tokens". ([Anthropic](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents))
- **Bridge to a global file [inference]:** a global file is paid on *every* session of *every* project, on top of project files, harness system prompt, tools/MCP, and skills metadata; Anthropic notes MCP servers add tool names and instructions to context even when unused ([best practices](https://code.claude.com/docs/en/best-practices.md)). IFScale and context-rot measure degradation with instruction/token count, but at far higher counts (tens to hundreds) than a 30–80-line file; no study I found pins a threshold for agent instruction files. Gloaguen found no length effect within the lengths of committed files (24–2003 words).

### 2.6 Synthesis of the evidence (bounded)

1. Specific operational instructions are followed (Gloaguen, tool-usage counts).
2. Following them changes behaviour and raises cost; benefits to task success are unproven (Gloaguen: not significant) or are efficiency-only (Lulla).
3. Generic overviews and LLM-generated boilerplate are the weakest content (Gloaguen).
4. Redundancy with existing docs is why files often don't help; when docs are absent, files help (+2.7%).
5. Vendor docs and the empirical paper agree on a *direction* (short, non-discoverable, specific) but the paper shows vendors' own recommendation to include overviews/auto-generate files is not supported by its data.

---

## 3. Maintaining the file over time

**Vendor docs say little about versioning/changelogs/ADRs for instruction files.** I found no vendor doc that prescribes a changelog or ADR process for AGENTS.md/CLAUDE.md **[absence of evidence from the docs listed above; not exhaustively verified]**. What exists:

### 3.1 Vendor-documented maintenance mechanics

- **Treat as living docs.** agents.md: "Treat AGENTS.md as living documentation." ([agents.md](https://agents.md/)) Cursor: "Check your rules into git ... When you see Agent make a mistake, update the rule." ([Cursor](https://cursor.com/docs/rules))
- **Prune.** Claude: review "periodically to remove outdated or conflicting instructions"; `/doctor prompt-audit` (v2.1.283+) finds "instructions written for older models, references to files or commands that don't exist, and files that contradict each other", proposes edits, changes nothing until asked. ([memory](https://code.claude.com/docs/en/memory.md)) Gemini: "Periodically check your GEMINI.md files to remove outdated rules." ([tutorial](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/cli/tutorials/memory-management.md)) AAIF: delete instructions that "no longer affect decisions"; a stale warning "is worse than no warning". ([AAIF](https://aaif.io/blog/writing-an-effective-agents-md))
- **Inspect what the agent actually loaded.** Claude `/context all`, `/memory`, `InstructionsLoaded` hook for logging which files load and why ([memory](https://code.claude.com/docs/en/memory.md)); Gemini `/memory show` prints "the raw text the model receives" ([tutorial](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/cli/tutorials/memory-management.md)); Codex: ask it to "Summarize the current instructions", or log with `-c log_dir=...` ([Codex guide](https://learn.chatgpt.com/docs/agent-configuration/agents-md.md)).
- **Test instruction changes in a PR (Copilot).** Copilot code review reads instructions from the PR *head* branch, "so you can test changes to them in the same pull request without merging them first". ([Copilot repo instructions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions)) Security flip side in §4.
- **Don't assume tools behave the same.** AAIF: how nested files are discovered/combined "can differ between agent implementations", so test each agent you rely on. ([AAIF](https://aaif.io/blog/writing-an-effective-agents-md))
- **Hooks for what must hold; delete hooks for mere preferences** ([Claude best practices](https://code.claude.com/docs/en/best-practices.md)): an instruction audit includes deciding whether each line is a preference or an invariant.
- **Maintainer notes without token cost** via HTML comments (Claude only verified) ([memory](https://code.claude.com/docs/en/memory.md)). Useful for "why this line exists" **[inference]**.

### 3.2 Evaluating changes

- Gloaguen et al. conclude any attempt to improve agents through context files "should be rigorously evaluated before deployment". ([arXiv](https://arxiv.org/html/2602.11988v3)) Their method (with/without file, paired tasks, steps/cost/success, trace analysis of tool calls) is a template; a personal-scale version would be far smaller and noisier **[inference]**.
- Anthropic's skill-authoring guide is the most explicit vendor eval workflow for agent instructions: "Create evaluations BEFORE writing extensive documentation"; identify gaps by running without the skill, build ~3 scenarios, establish a baseline, write minimal instructions, "Run evaluations, compare against baseline, and refine"; test across each model you plan to use. ([Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices.md)) It is about skills, not global memory files; applying it here is an extension **[inference]**.
- Anthropic's context-engineering post: "start by testing a minimal prompt with the best model available ... then add clear instructions ... based on failure modes". ([Anthropic](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents))

### 3.3 Recording decisions (ADR)

- Primary source for the ADR format: Michael Nygard, "Documenting Architecture Decisions" (2011): short numbered records (Title, Context, Decision, Status, Consequences); numbers never reused; reversed decisions kept and marked superseded; "one or two pages"; motivated by avoiding "blind acceptance" or "blind reversal" of past decisions. ([Cognitect blog](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions))
- Nygard's own rationale ("Large documents are never kept up to date. Small, modular documents have at least a chance") applies equally to keeping the *rationale* out of the loaded file and in a separate record **[inference]**.
- No vendor doc says to record instruction-change decisions; applying ADRs to an instruction file is my proposal, not a documented practice. Anything about lightweight vs full ADRs for personal use is **[community opinion/inference]**.

---

## 4. Security and privacy of a global file synced to git

### 4.1 Secrets

- AAIF: instructions "should never contain credentials, secrets or private operational details simply to save an agent from asking for them". ([AAIF](https://aaif.io/blog/writing-an-effective-agents-md))
- GitHub secret scanning scans full Git history on all branches (public repos free/automatic); when a secret is found, "rotate the affected credential immediately"; removing it from history is "time-intensive and often unnecessary if you've already revoked the credential". ([GitHub docs](https://docs.github.com/en/code-security/concepts/secret-security/secret-scanning)) Implication: once committed, assume leaked; history rewriting is not a remedy **[inference]**. Scanning coverage for a private user-owned repo on a personal plan is **[unverified]**: docs list private-repo scanning as needing Secret Protection on Team/Enterprise plans.
- "Never commit secrets" is an *agent* instruction in many files (GitHub blog: the most common helpful constraint) ([GitHub blog](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)); it constrains the agent, not your own commits **[inference]**.
- **Agent-written memory can land in a synced file [inference from cited docs].** Gemini's memory tool writes cross-project preferences into `~/.gemini/GEMINI.md` and its tutorial example saves "the staging server IP is 10.0.0.5" ([memory tool](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/tools/memory.md), [tutorial](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/cli/tutorials/memory-management.md)). If that path is symlinked into the repo, such facts would be committed. Claude's auto memory is separate and machine-local (`~/.claude/projects/<project>/memory/`, not shared across machines) ([memory](https://code.claude.com/docs/en/memory.md)). Whether Codex or others write to their global file automatically **[unverified]**.
- Claude Code docs' own split for private content: project-local instructions in a gitignored `CLAUDE.local.md`; user content in `~/.claude`. ([memory](https://code.claude.com/docs/en/memory.md))

### 4.2 Public vs private repo (privacy, not secrets)

- Public repo exposes your workflow, tools, internal hostnames/org names/paths, and any "ask before touching X" lists. No source states this directly; it follows from the file being plain text in a public repo **[inference]**.
- OWASP lists "Revealing sensitive information about AI system infrastructure or system prompts" as a prompt-injection impact and has an LLM07 System Prompt Leakage entry ([OWASP LLM01](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)). A public instruction file hands attackers your agent's "system prompt"-equivalent and can make targeted injection easier **[inference]**.

### 4.3 Prompt injection via committed instruction files

- **Mechanism.** OWASP: prompt injection need not be human-readable "as long as the content is parsed by the model"; indirect injection comes from "external sources, such as websites or files"; no foolproof prevention known; mitigations include least privilege, human approval for high-risk actions, segregating untrusted content. ([OWASP LLM01](https://genai.owasp.org/llmrisk/llm01-prompt-injection/))
- **Rules File Backdoor** (Pillar Security, 18 Mar 2025; **vendor security research**): invisible Unicode (zero-width joiners, bidi markers) in Cursor/Copilot rule files carried hidden instructions that made the AI insert a malicious script and not mention it; they report hidden characters also invisible in the GitHub PR review UI, and that poisoned rule files survive forking. Tested on Cursor and Copilot. ([Pillar](https://www.pillar.security/blog/new-vulnerability-in-github-copilot-and-cursor-how-hackers-can-weaponize-code-agents)) Vendor responses/patches **[unverified]**. Independent reproduction **[unverified]**.
- **Repo-controlled config is an attack surface.** Claude Code advisory GHSA-mmgp-wc2j-qcv7 / CVE-2026-33068 (High, 7.7): a repo's committed `.claude/settings.json` could set `permissions.defaultMode` to `bypassPermissions` and silently skip the trust dialog; fixed in 2.1.53. ([GitHub advisory](https://github.com/anthropics/claude-code/security/advisories/GHSA-mmgp-wc2j-qcv7)) Relevant lesson: settings/instructions that travel with a checkout are trusted by design in some paths.
- **Trust model for user-scope files.** Claude Code treats `~/.claude/CLAUDE.md` and `~/.claude/rules/` as "files you wrote yourself": it loads their imports without the approval dialog it shows for project-level external imports, which exists "to protect you from files other people commit to a shared project". ([memory](https://code.claude.com/docs/en/memory.md)) So whatever lands in your global file runs with full trust in every project **[inference]**: a compromised global repo (stolen credentials, malicious merged PR, poisoned dependency of your sync script) affects all your projects and machines.
- **Copilot reads instructions from the PR head branch** for code review ([docs](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions)): in a repo where others submit PRs, the PR can alter the instructions used to review itself **[inference]**. Lower relevance for a single-owner repo.
- **Vendor best practices for untrusted content:** review proposed commands, avoid piping untrusted content, VMs for risky tool calls; "no system is completely immune". ([Claude Code security](https://code.claude.com/docs/en/security.md)) Claude Code's `ConfigChange` hooks can audit/block settings changes during sessions (same page).
- **What reduces risk for a personal repo [inference]:** private repo; single owner; branch protection/2FA/signed commits on it; no third-party contributions to instruction text; scan diffs for non-printing Unicode (Pillar offers a scanner, [rule-scan.pillar.security](https://rule-scan.pillar.security/), not tested); copy-in rather than auto-pull of others' rule files; no executable hooks or scripts bundled with the instruction repo.
- Don't use instructions as the security boundary: use permissions/sandbox/hooks ([Claude best practices](https://code.claude.com/docs/en/best-practices.md), [AAIF](https://aaif.io/blog/writing-an-effective-agents-md)).

---

## 5. Synthesis: candidate principles (my inference)

Each is my conclusion from the cited evidence, not a vendor or paper recommendation.

1. **Earn every line.** Admit an instruction only if (a) the agent can't derive it, and (b) it changes behaviour. *Evidence:* Claude best-practices test; AAIF question; Gloaguen overviews don't help and redundant content adds cost. Sources in §1.1, §2.1.
2. **Global file = personal, cross-project, non-discoverable preferences** (communication style, workflow habits, tool preferences, approval boundaries); repo-specific facts (commands, layout, architecture) stay in project files. *Evidence:* Claude scope table; Gemini memory-tool routing; Codex "personal defaults"; Copilot scope advice. §1.3
3. **No repo overviews, no tech-stack summaries, no generic best practices** in the global file. *Evidence:* Gloaguen §4.3; Cursor/AAIF "what to avoid". §1.6, §2.1
4. **Keep it short; treat ~200 lines (Claude) as a ceiling and aim far lower**, since it's loaded alongside project files and shares a combined budget (Codex 32 KiB). *Evidence:* Claude's 200-line target; Codex cap; IFScale/context rot show degradation with load though not at this scale; Gloaguen found no length effect. The evidence for any specific number is weak. §1.1, §1.3, §2.5
5. **Write verifiable, concrete rules with the reason attached; avoid shouty emphasis.** *Evidence:* Claude/AAIF specificity examples; "reasons let Claude generalise"; emphasis softening for newer models; "adding IMPORTANT probably won't fix it". §1.1, §1.5
6. **Put invariants in enforcement, not prose.** Anything that must never/always happen → hooks, permissions, sandbox, CI. Keep the file for judgment-level preferences. *Evidence:* Claude best practices ("Claude's call"), Copilot non-guarantee, AAIF. §1.1
7. **Add on observed failure; remove on obsolescence.** Add a rule after the second repeat of a mistake; record the triggering incident; prune lines that no longer change behaviour or were written for older models. *Evidence:* Claude/Codex/Cursor "repeat mistake" triggers; `/doctor prompt-audit`; AAIF. §1.1, §3.1
8. **Avoid cross-agent conflicts and tool-specific precedence reliance.** Keep one shared body; put agent-specific instructions in clearly separated sections or tool-specific files. *Evidence:* three different precedence models; conflict warnings (Claude, Copilot); AAIF "test each agent". §1.3, §3.1
9. **Keep rationale and history out of the loaded file.** One short ADR per non-obvious decision (including "removed X because Y"), in a separate directory not loaded by agents; keep the loaded file a minimal current state. *Evidence:* Nygard's ADR rationale; HTML-comment stripping (Claude only); loaded-file size cost. §3.3. The "why not inside the loaded file" part is my inference.
10. **Evaluate changes against a baseline, at least informally.** For each change that is meant to alter behaviour, name the failure observed, define a check, compare behaviour with/without the line on a few representative tasks, per agent/model you use. *Evidence:* Gloaguen's conclusion; Anthropic eval-first guidance for skills; context-engineering "start minimal". Effect sizes at personal scale will be below noise; use for catching regressions/conflicts, not proving gains. §3.2
11. **Private repo; no secrets, internal hostnames, or customer/employer names; no auto-writing agent memory into the tracked file; non-printing-Unicode check on every diff; treat the repo as trusted-config supply chain.** *Evidence:* AAIF, GitHub secret-scanning docs, Gemini memory-tool behaviour, OWASP, Pillar, Claude user-scope trust model. §4
12. **Version with git; record meaningful changes (what/why/evidence) in a changelog or ADR; tag known-good states.** *Evidence:* weak: Nygard (ADRs), agents.md ("living documentation"), Agent-READMEs observation that files evolve by frequent small additions. No vendor mandates this. §3

---

## 6. Open questions the evidence does not settle

1. **Do global files help at all?** No study evaluates user-level files. Gloaguen's null/negative result is for repo files on bug-fix/feature tasks; personal-preference rules (style, communication) may behave differently and are not measured by test-pass benchmarks.
2. **Is there a length threshold for agent instruction files?** Claude says 200 lines; Cursor 500; Codex 32 KiB; Copilot "2 pages" appears only in a generated-prompt template inside its docs (not a stated limit). Gloaguen found no length effect (24–2003 words); IFScale/context-rot tested far larger loads on synthetic tasks. No agent-specific curve exists.
3. **Does emphasis wording matter for CLAUDE.md/AGENTS.md in current models?** Anthropic advises dialing it back in API prompts and says IMPORTANT "probably won't fix" misses, but no data for memory files. Equivalent guidance for Codex/Gemini models was not found.
4. **Negative vs positive phrasing.** Gemini docs claim negative constraints work better; no data. Anthropic's TTS example prefers reasons over bare prohibitions. No head-to-head evidence.
5. **Instruction position.** IFScale shows primacy bias at moderate density; whether first lines of a global file get more adherence in real agent loops is untested. Claude orders broad→specific so project text is read after user text; Codex likewise. Effect of that ordering is untested.
6. **Do instructions decay over long sessions / after compaction?** Claude says project-root CLAUDE.md is re-injected after `/compact`; behaviour of others and adherence over long contexts not found.
7. **Model drift.** Which global-file lines become unnecessary or harmful as models improve? Claude's `/doctor prompt-audit` targets "older models" but publishes no criteria or data.
8. **Does a duplicated rule across global and project files help or hurt?** Gemini's memory tool says to avoid duplicating facts across tiers; no data. Claude/Copilot warn on conflicts, not on duplicates.
9. **Multi-agent sharing.** Whether one shared file works equally well for Claude/Codex/Gemini/Cursor (different models, different harness system prompts) is unmeasured; Gloaguen noted Dev files (not agent-specific) helped all agents except Claude Code, n small.
10. **Security evidence is thin and vendor-sourced.** Pillar's Rules File Backdoor is one vendor's disclosure, tested on Cursor and Copilot (2025); no independent reproduction located, no data for Codex/Claude/Gemini global-file poisoning, and no data on whether hidden Unicode is stripped by current harnesses.
11. **Whether changelog/ADR overhead pays off for a single-author instruction file.** Nygard's experience report (6–10 developers, a few months, 2011) is the only primary source and concerns architecture, not prompts.
12. **Whether Cursor user rules, Copilot CLI personal instructions, and other tools can load from a shared `~/.agents/AGENTS.md`** **[unverified here]**; see the tool-mechanics research. One related fact: Claude Code does not read `AGENTS.md` under a `.agents/` directory by default, though user-scope files may `@import` from the home directory ([memory](https://code.claude.com/docs/en/memory.md)).
