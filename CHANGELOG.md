# Changelog

What changed between editions of The AI Coding Playbook. Written for readers: if you already own an
edition, this tells you which chapters to re-read. The same link always serves the current edition.

## Edition 2026.11 — 26 November 2026

Verified against **Claude Code v2.1.332** (25 November 2026), 179 pages. This list is the delta
from **Edition 2026.10**, the PDF most subscribers hold (audited against Claude Code 2.1.288 on
3 October 2026). It is not the delta from the public Markdown repository, which had stayed at
2026.08 (v2.1.210–220); readers of that copy are two editions behind, so read the 2026.10 entry
below as well. Content that already shipped in 2026.10 (the Opus 5.5 and Sonnet 5.5 rows, per-model
`/effort`, the `ultracode` toggle, the `AGENTS.md` fallback in outline, `/review` as an alias of
`/code-review`) is not repeated here.

### Deadlines — check these first

- **30 Nov 2026 — fast mode for Opus 4.8 is removed.** A session still on Opus 4.8 with fast mode
  on gets its requests rejected, with no fallback. `/model` to Opus 5.5 or run `/fast off`. This
  replaces the 2026.10 warning about Opus 4.7. → Ch. 28, Appendix D
- **30 Nov 2026 or later — managed marketplaces update themselves** by default. Opt one out with
  `"autoUpdate": false` on its entry. → Ch. 16
- **14 Dec 2026 — standalone Claude Design closes.** → Ch. 08

### The five changes that matter most

1. **Model table moved on.** Fable 5.1 is now Fable 5.2 ($8 / $40, down from $10 / $50). Haiku 4.5
   is replaced by Haiku 5.5 (1M window, five effort levels, $0.10 / $0.50, five times that over
   100K tokens). Cache reads are now 0.05× on Opus 5.5 and Sonnet 5.5, 0.025× on Fable 5.2. The
   price spread top to bottom is 80:1 (was 10:1). → Ch. 27, 29
2. **Auto mode is the starting mode.** Interactive terminal and VS Code sessions start in auto
   since v2.1.284; `claude -p` starts in Manual since v2.1.302. New: who reviews, the supported
   models, `autoMode` rules and `/auto-mode-setup`, broad allow rules dropped on entry. A `-p` run
   no longer aborts on repeated classifier blocks; the action just does not run. → Ch. 05, 09, 17
3. **Defaults are per model.** Fable 5.2, Opus 5.5, Sonnet 5.5, and Haiku 5.5 all default to
   `medium` effort in Claude Code; thinking cannot be turned off on any of them. Fast mode now
   covers Fable 5.2 ($16 / $80, v2.1.321). → Ch. 27, 28
4. **Unattended and parallel work has more to it.** `/goal` pauses and ends spelled out
   (errors, retries, check-ins), `/loop` expiry now 30 days, agent view, dynamic workflows,
   Projects (beta), `/ship` and `/steward` for the PR loop. `/code-review` is narrower: bugs and
   `CLAUDE.md` violations only, cleanup moved to `/simplify`. → Ch. 08, 18, 22, 23
5. **Compaction you did not ask for.** Idle compaction summarises a 200K+ token conversation that
   sat idle about an hour; `/autocompact` moves the auto-compact trigger. `Glob` and `Grep` are
   absent by default on macOS, Linux and WSL. → Ch. 03, 04, 29

### Chapter by chapter

**Front matter**
- **00, manifest, Appendix F** — Scope line now v2.1.332, November 2026; sources for the new
  material added.

**Part I · Mechanism**
- **01** — Fable models also skip verification reminders. Stop-hook ceiling restated as eight
  blocks in a row with no tool call between; `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` raises it.
- **02** — `AGENTS.md` row added (read in place of `CLAUDE.md` when none exists, v2.1.277+). MCP
  startup cost now includes server instructions (cut at 4,096 chars). `/skill-doctor` added.
- **03** — Native 1M window on current models, auto-compact near 967K. Compaction table updated
  (skill index rebuilt v2.1.304+, plan re-injected, up to five files re-read). New section on
  `/autocompact` and idle compaction. New note that "manage context aggressively" is less of a
  hard wall than it was; `/clear` between tasks still pays.
- **04** — "More than 50" built-in tools (was 42; 13 gated, now about a dozen in Manual). New
  `Workflow` tool. `Glob`/`Grep` absent by default on macOS/Linux/WSL. Read-before-edit rule
  differs by model. `cd` carry-over corrected. Path-scoped `Write`/`Glob`/`Grep` rules never
  match. Server instructions load at startup.
- **05** — `manual` alias, the mode cycle from auto, which mode you start in, auto mode
  requirements, reviewer, and `autoMode` tuning. `-p` blocked-action behaviour corrected.
  Sandbox: write access to some home folders removed by default (v2.1.326); credential
  `"mode": "mask"`.

**Part II · Controls**
- **06** — `Up` restores a cleared draft (v2.1.288+; last ten kept, v2.1.292+). `Ctrl+Enter`
  sends queued messages; `←` on an empty prompt opens agent view; `Ctrl+R` search scope;
  `Ctrl+W` / `Alt+D` word deletion; thinking toggle note covers Sonnet 5.5 and Haiku 5.5;
  Apple Terminal removed from the native `Shift+Enter` list.
- **07** — Rewritten intro to `/` (commands, bundled skills, your skills, plugin skills in one
  menu).
- **08** — New rows: `/import`, `/autocompact`, `/ship`, `/steward`, `/skill-doctor`, `/tui`,
  `/advisor`, `/design-sync`. Changed: `/resume`, `/model`, `/effort`, `/fast`, `/permissions`,
  `/goal`, `/loop`, `/tasks`, `/code-review`, `/review`, `/doctor`. **Removed:** `/ultraplan`
  joins `/vim` and `/pr-comments` under "commands that no longer exist".
- **09** — `--permission-prompts none`, `--effort ultracode`, `--fallback-model` chains,
  `--tools` note on Glob/Grep, `--max-budget-usd` counts subagent spend, `-w '#PR'`, `claude
  doctor` API check, `claude agents --json`, `attach`/`logs`/`stop`/`respawn`/`rm`, `claude
  purge`, `claude ultrareview --trust-workspace`, `claude auth status`, `claude auto-mode`.

**Part III · Harness**
- **10** — `AGENTS.md` load table, import and symlink options, `/init` input sources, and a new
  section on auditing the file with `/doctor` and `/doctor prompt-audit`.
- **11** — Symlinked rules pointing outside the repo need external-import approval. Audit
  commands now `/memory` and `/context all`. **Correction:** the `/memory trim` line is removed;
  that command appears in neither the commands reference nor the Claude Code CHANGELOG. Prune
  stale entries from `/memory` by hand. (Also fixed in Appendix D.)
- **12** — New sections: skills from MCP servers (beta, `/<server>:<skill>`, v2.1.332) and
  gating, hiding and pruning (`skillOverrides`, `/skill-doctor`, `/reload-skills`,
  `PreSkillShellCommand`). `disable-model-invocation` and `model`/`effort` frontmatter details;
  reserved `computer-use` name.
- **13** — Explore model resolution on Fable; Explore and Plan skip `CLAUDE.md`. Subagents start
  in the background by default, forks on by default, new "Nesting and limits" section (three
  layers). Effort precedence with `CLAUDE_CODE_SUBAGENT_EFFORT`.
- **14** — `PreSkillShellCommand` event; "over thirty events". New section "When the hook itself
  fails" with `onFailure: "block"`. Trust dialog shows where hooks are declared (v2.1.329).
  `InstructionsLoaded` note.
- **15** — `claude mcp add-json <name> -` reads stdin (v2.1.327). `SKILLS` capability. New
  section on MCP tasks (v2.1.319) and elicitation; MCP SDK 2.0 client. `ENABLE_TOOL_SEARCH`
  modes.
- **16** — Manifest fields `engines["claude-code"]` and `optionalDependencies`; Marketplaces tab
  shows context-cost estimate; new section on mods (v2.1.287) with a hook/skill/MCP/mod chooser;
  managed marketplace auto-update from 2026-11-30.
- **17** — New keys: `permissions.defaultMode`, `blockWritesOutsideWorkingDirectories`,
  `denyInsteadOfPrompting`, `autoMode`, `disableAutoMode`, `idleCompaction`, `skillOverrides`,
  `disableSkillShellExecution`, `worktree.symlinkDirectories`, the Project instructions setting,
  `CLAUDE_CODE_SUBAGENT_EFFORT`, `CLAUDE_CODE_DISABLE_ULTRATHINK`, `ENABLE_TOOL_SEARCH`.
  `effortLevel` excludes `max`. Status-line null cases; `--safe-mode`.

**Part IV · Flows**
- **18** — `/plan [task]`; review step names `/code-review` and `/simplify`; `/ship`.
- **20** — Stop-hook ceiling and `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`.
- **21** — Raise effort on the debugging ladder; `/verify` as an observation rung.
- **22** — `/goal`: evaluator verdicts, "What pauses or ends a goal", `-p` starts in Manual.
  `/loop` expiry 30 days (was 7 before v2.1.330). Five-axis bound.
- **23** — New "Ways to run in parallel" table (agent view, agent teams, dynamic workflows,
  Projects); `claude -w`; `/code-review` flags; `--bg`; `claude agents`; `/fork`.
- **24** — Loop example now uses `--bare`, `dontAsk` and exit-status checks. `/batch` mechanics.
- **25** — A repo with `AGENTS.md` and no `CLAUDE.md` already works.
- **26** — Rewind table: a file Claude already edited does get later Bash changes undone.
  `/branch` and `/fork` for keeping a direction.

**Part V · Economics**
- **27** — Table rebuilt for Fable 5.2 and Haiku 5.5 with cache-read rates and a Claude Code
  default-effort row; refusals apply to all four models; Opus 5.5 behaviour notes (effort
  carry-over, turn can end on a status report); thinking cannot be disabled; 512-token cache
  minimum on all four; Haiku constraints removed. "Default to Sonnet" recommendation replaced
  with "pick the default on purpose" because Claude Code starts on Opus 5.5.
- **28** — Rewritten effort table, per-model defaults, precedence, `ultracode` as a setting,
  adaptive-reasoning list. Fast mode: Fable 5.2 row, flat 2× rates, Opus 5 and 4.8 rows, cache
  split between speeds, Opus 4.8 removal warning.
- **29** — Cache-read multipliers per model, minimum cacheable prompt per model, 80:1 tier
  spread, new section "Inside Claude Code: where the money goes" (`/usage`, cache lifetime, idle
  compaction, idle spend, agent teams about 7× tokens).
- **30** — `total_cost_usd` from `claude -p --output-format json`; `/goal` verdicts.

**Part VI · Prompts**
- **31** — "Think before coding" points at `/effort` and `ultrathink` rather than "think hard".

**Appendices**
- **A** — `/ship`; new "Defaults worth knowing" block; skill index and plan re-injection;
  Stop-hook "in a row".
- **B** — `/doctor prompt-audit` note.
- **C** — `AGENTS.md` row: Claude Code skips it when `CLAUDE.md` exists.
- **D** — Five new symptom rows (idle compaction, `-p` denials, auto-mode start, `/code-review`
  scope, Opus 4.8 fast mode) and one on Opus 5.5 `medium`; Grep/Glob row notes absence; memory
  row no longer cites `/memory trim`.
- **E** — New terms: agent view, auto mode, Code Mode, goal, idle compaction, mod; fast mode
  entry now Opus and Fable.
- **98** — Edition line.

### Known gaps

- Code Mode (v2.1.323, experimental) appears only in the glossary; it has no chapter.
- None of the book's own benchmarks were re-run for this edition.

## Edition 2026.10 — 3 October 2026

Verified against Claude Code v2.1.210–220, then audited against the changelog through v2.1.288.
Two commits: the Claude 5.5 lineup (3 Oct, 22:58), then a changelog audit (3 Oct, 23:07).

- **Ch. 27, 28, 29** — Model table moved to Fable 5.1, Opus 5.5, Sonnet 5.5, Haiku 4.5 ($4 / $20
  for Opus 5.5, $2 / $10 for Sonnet 5.5). Opus 5.5 defaults to `medium`; thinking behaviour and
  prompting changes after Opus 5.
- **Ch. 31** — Prompt library touched for the new lineup.
- **Audit to v2.1.288** — `AGENTS.md` read when no `CLAUDE.md` exists (v2.1.277+); `defaultMode`
  of `auto` or `bypassPermissions` ignored in project settings (v2.1.257+); `/effort` saved per
  model and the `ultracode` toggle (v2.1.251+, v2.1.284+); `/fork` copy gets its own worktree,
  `/review` alias, `--bare` behaviour (v2.1.286+).

## Edition 2026.08 — 5 August 2026

Verified against Claude Code v2.1.210–220. This is the edition the public Markdown repository had
stopped at.

- **Ch. 11** — Layer benchmark: a rule in auto memory and the same rule in `CLAUDE.md` both held
  3/3 when the request only tempted the violation and both fell to 0/3 when the prompt named it.
  Audit pass, separate subagent memory directories, a five-layer chooser.
- **Ch. 05** — Real resolution order: `PreToolUse` hook, deny, allow, ask, mode. Why a deny rule
  outlives `bypassPermissions`, with a deny rule observed firing.
- **Ch. 14** — Bare model alias that makes a prompt hook silently inert; a `Stop` check must match
  evidence, not the command name.
- **Ch. 22** — Classify the doors before an unattended run.
- **Ch. 31** — Reversibility gate at P-13 (later prompts shift by one).
- **Appendix C** — Shipped harness `settings.json` now carries destructive-command denies and ask
  rules, not read denies alone.

## Earlier editions

First Markdown edition. See the git history.
