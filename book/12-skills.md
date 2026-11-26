# Skills

_III · Harness · Chapter 12_

A skill is a markdown file whose body loads _only when used_. That single property is the whole point: long reference material and multi-step procedures cost almost nothing until the moment they are needed. Create one when you keep pasting the same instructions into chat, or when a section of `CLAUDE.md` has grown from a fact into a procedure.

> **Note — Commands and skills have merged**
>
> `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both create `/deploy` and behave the same way. Existing `commands/` files keep working. Skills add a directory for supporting files, frontmatter for invocation control, and autonomous loading.

## Where skills live

| Scope | Path | Applies to |
| --- | --- | --- |
| Enterprise | Via managed settings | All users in the organisation |
| Personal | `~/.claude/skills/<name>/SKILL.md` | All your projects |
| Project | `.claude/skills/<name>/SKILL.md` | This project only |
| Plugin | `<plugin>/skills/<name>/SKILL.md` | Where the plugin is enabled |
| MCP server | Served by the server — appears as `/<server>:<skill>` | Where the server is connected. Beta, v2.1.332 — see below |

On a name clash, enterprise overrides personal overrides project, and any of them overrides a bundled skill. Nested `.claude/skills/` directories below your working directory load when Claude touches files there; a clashing nested skill appears under a directory-qualified name like `/apps/web:deploy`, and Claude picks the variant matching the files it is working on. Claude Code watches these directories and picks up edits within seconds — **no restart needed.**

## The file

```
---
name: fix-issue
description: Fix a GitHub issue end to end, from `gh issue view` to an open PR
disable-model-invocation: true
argument-hint: [issue-number]
allowed-tools: Bash(gh *) Read Edit Grep Glob
---

Analyze and fix GitHub issue $ARGUMENTS.

1. `gh issue view $0` to get the details
2. Search the codebase for relevant files
3. Implement the fix
4. Write and run tests that prove it
5. Run lint and typecheck
6. Commit with a descriptive message, push, open a PR
```

## Frontmatter reference

All fields are optional; only `description` is genuinely recommended. v2.1.218+ booleans accept `yes/no/on/off/1/0` in any case.

| Field | What it does |
| --- | --- |
| `name` | Display name in listings. Defaults to the directory name. For **plugin** skills it also sets the command’s final segment. |
| `description` | What the skill does **and when to use it**. This is how Claude decides to load it. Put the key use case first; third person. |
| `when_to_use` | Extra trigger phrases or example requests, appended to the description. Counts toward the 1,536-character listing budget. |
| `argument-hint` | Autocomplete hint, e.g. `[filename] [format]`. |
| `arguments` | Named positional arguments for `$name` substitution. |
| `disable-model-invocation` | `true` keeps Claude from loading it autonomously **and keeps it out of the startup index**. It also stops the skill being preloaded into subagents and running when a session-scoped scheduled task fires with it as the prompt. Use for anything with side effects. |
| `user-invocable` | `false` hides it from the `/` menu — background knowledge you don’t invoke by hand. |
| `allowed-tools` | Tools usable without a permission prompt during the invoking turn. The grant clears on your next message. |
| `disallowed-tools` | Tools removed while the skill is active — e.g. `AskUserQuestion` for a background loop. |
| `model` · `effort` | Override for the rest of the current turn; not saved to settings. `model` applies when **you** run the skill with `/name`; when Claude loads it on its own the session keeps its model v2.1.330. With `context: fork`, `effort` sets the forked subagent’s level. |
| `context: fork` + `agent` | Run the skill’s content **as a subagent’s prompt**, in an isolated window. → Ch. 13 |
| `background` | With `context: fork`, `false` makes the invoking turn wait for the result. v2.1.218+ forked skills run in the background by default. |
| `paths` | Globs limiting when the skill auto-activates. |
| `hooks` | Hooks scoped to this skill’s lifetime. |
| `shell` | `bash` (default) or `powershell` for inline command execution. |

## String substitutions

| Variable | Expands to |
| --- | --- |
| `$ARGUMENTS` | Everything passed. If absent from the body, arguments are appended as `ARGUMENTS: <value>`. |
| `$0` `$1` · `$ARGUMENTS[N]` | Positional, 0-based. Shell-style quoting, so `/skill "hello world" x` gives `$0 = hello world`. |
| `$name` | Named argument declared in `arguments:`, mapped by position |
| `${CLAUDE_SKILL_DIR}` | The skill’s own directory. v2.1.129+ also substituted inside `allowed-tools`. |
| `${CLAUDE_PROJECT_DIR}` | Project root v2.1.196+ |
| `${CLAUDE_SESSION_ID}` · `${CLAUDE_EFFORT}` | Session ID; current effort level |

The `${CLAUDE_SKILL_DIR}` pairing is the neat trick — use it in both places and the permission rule matches the exact command the body tells Claude to run, so a bundled script executes without prompting:

```
---
name: render-chart
allowed-tools: Bash(${CLAUDE_SKILL_DIR}/scripts/render.sh *)
---
Run `${CLAUDE_SKILL_DIR}/scripts/render.sh <csv-file>` to render the chart.
```

## Progressive disclosure inside a skill

Keep `SKILL.md` **under 500 lines**. Past that, split into supporting files and reference them so Claude knows what each contains and when to load it. Files that are executed rather than read — scripts — cost nothing at all.

```
my-skill/
├── SKILL.md      # required — overview and navigation
├── reference.md  # detailed API docs — loaded when needed
├── examples.md   # usage examples — loaded when needed
└── scripts/
    └── helper.py # executed, never loaded into context
```

If two paths through a skill are mutually exclusive or rarely used together, keep them in separate files. That is where the token saving actually comes from.

## Naming and description craft

- Use the **gerund form** for names — `reviewing-prs`, `writing-migrations` — because it describes the activity the skill provides.
- Write descriptions in **third person**. They are injected into the system prompt, and an inconsistent point of view measurably hurts discovery.
- State both **what it does and when to use it**. “Reviews a diff” is half a description; “Reviews a diff for correctness bugs before commit or PR” is a whole one.
- Name constraints: max 64 characters, lowercase letters, numbers, and hyphens only. Description max 1,024 characters, no XML tags.

## Skill composition — how skills call skills

There is **no separate sub-skill primitive.** A `SKILL.md` body is just instructions loaded into the conversation; if those instructions tell Claude to invoke another skill by name, Claude calls the same `Skill` tool a human would. Each hop loads more text into the window, so compose only the stages the current task needs.

> **Warning — The hard gate**
>
> `disable-model-invocation: true` hides a skill from Claude’s context _entirely_, not just from autonomous triggering. A skill carrying the flag cannot be invoked via the `Skill` tool under any circumstance — **even when another loaded skill’s instructions explicitly tell Claude to call it.** Removing the flag is the only way to let one skill orchestrate another.

| Pattern | Who writes the task | Runs where |
| --- | --- | --- |
| Skill with `context: fork` + `agent:` | The skill’s own content becomes the subagent prompt | Isolated subagent — no access to the calling conversation |
| Subagent with a `skills:` field | The delegating message | The subagent’s context, with those skills preloaded as reference |

Reach for orchestration when a workflow has clean reusable stages (fetch → draft → polish → review) that are each useful standalone. Pick a **subagent** when a stage benefits from a fresh context — a review pass gains from not being close to the draft. Pick an **inline Skill call** when it benefits from continuity — a rewrite pass gains from the context already built up.

## Skills from MCP servers (beta)

An MCP server can now ship skills alongside its tools — the instructions for using the tools travel with them. A skill’s name is `<server>:<skill>`, so a `pdf` skill on a server named `docs` is `/docs:pdf report.pdf`; connectors and plugin servers add a prefix (`claude_ai_Docs:pdf`, `plugin_my-plugin_docs:pdf`). Find them with `/` or `/skills`. v2.1.332; the docs state v2.1.331+.

- **Availability.** On with `/login` on Pro, Max, and Ultra; off on Team and Enterprise until the organisation enables Skills over MCP (connectors) or sets `disableDirectMcpSkills: false` in managed settings (other servers). Terminal and local Desktop sessions only — not cloud sessions, `claude -p`, GitHub Actions, the Agent SDK, cloud providers, or a custom `ANTHROPIC_BASE_URL`. The model must be Fable, or Opus 4.8, Sonnet 5, Haiku 5.5 or later.
- **What does not apply.** The server supplies the content, so Claude Code ignores `allowed-tools`, `disallowed-tools`, `hooks`, `model`, `effort`, `context`, `agent`, `background`, `shell`, and `paths`; `!` commands do not run, `@` references are not attached, and `${CLAUDE_PROJECT_DIR}` is not substituted. Supporting files arrive as MCP resources.
- **Approval.** In auto mode the classifier decides whether a skill loads, without reading it. In Manual or `acceptEdits` you get a prompt; **Allow this version** skips it until the server changes the skill. Rules are per server: `Skill(mcp__docs__*)` in `deny`, `ask`, or `allow` (`allow` only in user or managed settings, for servers you added yourself), or `Skill(docs:pdf)` for one skill.

> **Warning — A server can edit your instructions**
>
> A skill can direct Claude to run code, and the server can change it at any time. Anthropic scans skills only from some connectors, not from servers you add with `claude mcp add`. Add only servers you would trust with a `CLAUDE.md`.

## Gating, hiding, and pruning skills

- **`skillOverrides`** (settings, or the `/skills` menu — `Space` cycles, `Esc` saves to `settings.local.json`) sets a skill per name without editing its `SKILL.md`: `"on"`, `"name-only"` (name listed, description hidden), `"user-invocable-only"` (hidden from Claude, still in `/`), `"off"`. It does not apply to plugin skills — use `/plugin`.
- **`/skill-doctor`** v2.1.261+ shows what each skill costs in context and how often it is used, and flags the ones never invoked. Every listed skill taxes every turn; prune the expensive, unused ones first.
- **`/reload-skills`** re-scans the skill directories if a new skill does not show up; after compaction it re-sends the full skill list to Claude.
- **`!` commands bypass `PreToolUse`.** A skill’s inline `` !`command` `` runs while the skill renders, so a guard you wrote as a `PreToolUse` hook never sees it. Add the same script to the **`PreSkillShellCommand`** hook (v2.1.332; matcher `Bash|PowerShell`, `onFailure: "block"`). It can only refuse — never approve or rewrite. A failed injected command aborts the whole invocation. `disableSkillShellExecution: true` turns inline shell off for user, project, plugin, and added-directory skills. → Ch. 14

## Troubleshooting

| Symptom | Cause and fix |
| --- | --- |
| Skill never triggers | The description doesn’t name the trigger. Add `when_to_use` with the phrases you actually type. |
| Skill triggers too often | The description is too broad. Narrow it, or set `disable-model-invocation: true` and invoke by hand. |
| Description appears cut short | You are over the 1,536-character listing budget across `description` + `when_to_use`. `/doctor` estimates the listing’s cost; `skillListingBudgetFraction` raises the budget, `skillOverrides` frees it. |
| Skill missing in a cloud session | Cloud and Cowork sessions do not read `~/.claude/skills/`. Commit it to the repo’s `.claude/skills/` or ship it in a plugin. |
| A skill named `computer-use` stops loading | On a Mac with computer use enabled the name is reserved v2.1.332; a startup notice asks you to rename yours. |
