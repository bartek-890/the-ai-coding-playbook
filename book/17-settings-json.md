# settings.json

_III · Harness · Chapter 17_

One file format, five scopes, and a precedence order that explains most “why isn’t my setting taking effect?” confusion. Everything configurable in Claude Code eventually lands here.

## Scopes, highest precedence first

| Scope | Location | Shared |
| --- | --- | --- |
| **Managed** | MDM / plist / registry / `managed-settings.json` | IT-deployed. **Cannot be overridden.** |
| **CLI arguments** | `--settings`, individual flags | This invocation only |
| **Local** | `.claude/settings.local.json` | No — gitignored |
| **Project** | `.claude/settings.json` | Yes — committed |
| **User** | `~/.claude/settings.json` | No — all your projects |

Permission rules **merge** across scopes rather than overriding; the precedence order applies to scalar values. A lower scope can never loosen a managed deny. Run `/status` to see which sources actually loaded.

## The keys worth knowing

| Key | Effect |
| --- | --- |
| `model` · `fallbackModel` | Active model; chain of fallbacks when the primary is overloaded |
| `effortLevel` | `low` · `medium` · `high` · `xhigh` (not `max`) — the default for models with no saved level of their own. Since v2.1.251 `/effort` saves a level per model under `modelSettings` instead, and a model’s saved level beats this key |
| `fastMode` | Faster, costlier output on supported models (Opus 5.5, Fable 5.2…). → Ch. 28 |
| `permissions.allow` / `.deny` / `.ask` | The `ToolName(specifier)` rules. → Ch. 04 |
| `permissions.defaultMode` | Starting permission mode. With none set, terminal and VS Code sessions start in **auto**; `claude -p` and the SDK start in Manual. An `auto` or `bypassPermissions` written in a _project_ file is ignored. → Ch. 04 |
| `permissions.blockWritesOutsideWorkingDirectories` v2.1.314 | `Write`, `Edit`, and `NotebookEdit` refuse paths outside the working directories in **every** mode, `bypassPermissions` included. Any file can turn it on and none can lift it. Does not limit shell commands, MCP tools, or hook scripts — that is `sandbox.*` |
| `permissions.denyInsteadOfPrompting` v2.1.319 | A call that would wait on a permission prompt is denied and the turn continues — for sessions nobody is watching. User, managed, or `--settings` only; a repo cannot set it. Calls an allow rule, the mode, or the auto classifier approves still go ahead |
| `autoMode` (user or managed only) | Your own prose rules for the auto-mode classifier: `environment`, `allow`, `soft_deny`, `hard_deny`, plus `classifyAllShell`. Include `"$defaults"` in every array or you replace the built-ins. Ignored in project settings |
| `disableAutoMode` | `"disable"` removes auto from the mode cycle and the default |
| `sandbox.*` | OS isolation, credentials, network allowlists. → Ch. 05 |
| `hooks` | Lifecycle hooks. → Ch. 14 |
| `env` | Environment variables applied to every session |
| `autoMemoryEnabled` · `autoMemoryDirectory` | Auto-memory control. → Ch. 11 |
| `claudeMd` (managed only) | Org-wide instructions inline, without deploying a separate file |
| `claudeMdExcludes` | Glob patterns of memory files to skip. Arrays merge across layers. |
| `cleanupPeriodDays` | Session, checkpoint, and subagent-transcript retention. Default 30. |
| `idleCompaction` | Compact a 200K+ conversation you have stepped away from, about an hour in, before the 1-hour prompt cache expires. Default `true`; `false` turns it off |
| `skillOverrides` · `disableSkillShellExecution` | Hide or collapse a skill without editing it; stop skills and commands running inline shell. → Ch. 12 |
| `worktree.symlinkDirectories` | Symlink large directories (`node_modules`, `.cache`) into each worktree instead of copying them |
| `pluginConfigs["cc-plugin-agents-md@builtin"].options.instructionFiles` | The **Project instructions** setting: which of `CLAUDE.md` and `AGENTS.md` load. → Ch. 10 |
| `includeCoAuthoredBy` · `attribution` | Commit and PR message customisation |
| `enableAllProjectMcpServers` | Auto-approve project-scoped MCP servers |
| `disabledMcpjsonServers` · `enabledMcpjsonServers` | Block or allowlist specific servers |
| `statusLine` | Custom status line — the place to surface live context usage |
| `outputStyle` | Output formatting mode. Read at startup; needs a restart. |
| `editorMode` | `normal` or `vim` |
| `disableAllHooks` | Kill switch for hooks, custom status line — and `/goal` |
| `disableBundledSkills` | Turn off the built-in skills |
| `autoUpdatesChannel` | `latest` or `stable` |

## Environment variables

| Variable | Effect |
| --- | --- |
| `CLAUDE_CODE_EFFORT_LEVEL` | Effort level — **highest precedence, overrides everything** |
| `MAX_THINKING_TOKENS` | Cap reasoning tokens; `0` disables thinking on the API |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | Revert fixed-budget models to the older thinking mode |
| `DISABLE_AUTO_COMPACT` | Turn off automatic compaction |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | Turn off auto memory |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | Turn off rewind checkpoints |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | Load memory files from `--add-dir` directories |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | Scrub provider credentials from **every** subprocess environment |
| `CLAUDE_CODE_NEW_INIT` | Interactive multi-phase `/init` |
| `DISABLE_AUTOUPDATER` · `CLAUDE_CODE_ENABLE_TELEMETRY` | Updates and telemetry |
| `CLAUDE_CODE_SUBAGENT_EFFORT` | Effort subagents run at when nothing else names one (v2.1.316). → Ch. 13 |
| `CLAUDE_CODE_DISABLE_ULTRATHINK` | Turn the `ultrathink` keyword off (v2.1.316) |
| `ENABLE_TOOL_SEARCH` | MCP tool deferral: unset or `true` defers all, `auto:N` defers past N% of the window, `false` loads everything. → Ch. 15 |

## A working project settings file

```
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": ["Bash(pnpm test *)", "Bash(pnpm lint)", "Bash(git status)"],
    "ask":   ["Bash(git push *)"],
    "deny":  ["Read(./.env*)", "Read(~/.ssh/**)", "Read(~/.aws/**)",
              "Edit(migrations/**)"],
    "blockWritesOutsideWorkingDirectories": true
  },
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{ "type": "command",
                   "command": "pnpm prettier --write \"$CLAUDE_FILE\"" }]
    }]
  },
  "cleanupPeriodDays": 30
}
```

## The status line: put context usage where you can see it

Chapter 3 argues that the context window is the resource that decides output quality. The status line is how you stop having to run `/context` to find out where you are. It runs any shell script, receives JSON session data on stdin, and prints whatever you want.

```
{
  "statusLine": { "type": "command", "command": "~/.claude/statusline.sh", "padding": 2 }
}
```

The one-liner version, if you do not want a script:

```
{
  "statusLine": {
    "type": "command",
    "command": "jq -r '\"[\\(.model.display_name)] \\(.context_window.used_percentage // 0)% ctx\"'"
  }
}
```

The fields worth reaching for:

| Field | What it holds |
| --- | --- |
| `context_window.used_percentage` | Pre-calculated percentage used — the one number that matters |
| `context_window.context_window_size` | 200,000 by default, 1,000,000 on extended-context models |
| `context_window.total_input_tokens` | Tokens currently in the window, from the last API response. Includes cache reads and writes. v2.1.132+ — before that these were cumulative session totals. |
| `cost.total_cost_usd` | Estimated session cost, computed client-side. v2.1.211+ resets to $0 on `/clear`. |
| `model.display_name` | Which model is actually running — useful after a fallback |
| `exceeds_200k_tokens` | Fixed 200K threshold flag, regardless of the real window size |

Two `null` cases to guard: `current_usage` is null before the first API call and again after `/compact` until the next one, and the percentage fields can be null early in a session. Hence `// 0` in the example. `/statusline` asks Claude to write or remove one for you. The `subagentStatusLine` setting is gone (v2.1.323): subagent rows always use the default look, and a settings file that still holds the key loads without error.

> **Key — The three settings that change behaviour with zero discipline**
>
> 1. **Credential deny rules.** There is no built-in list; by default the agent can read `~/.ssh` and `~/.aws/credentials`. These close it from the first session.
> 2. **An `Edit(...)` deny on migrations, auth, or billing.** Off-limits zones are a security control, not a hygiene habit.
> 3. **A format-on-edit hook.** Deletes an entire category of instruction from `CLAUDE.md`, permanently.

> **Note — When a setting seems to do nothing**
>
> Run `/status` for active setting sources, then `/doctor` — or `claude doctor` in your shell for the entries Claude Code rejected — then start with `--safe-mode` to confirm your configuration is the cause. Most settings reload live; `model` and `outputStyle` require a restart. `/permissions` now refuses a rule that would be skipped at the next start, such as `rm -rf` with no tool name (v2.1.332).
