# Tools and the permission split

_I · Mechanism · Chapter 04_

Claude Code ships **more than 50 built-in tools** — the tools reference lists 54, several of them conditional on platform, plan, or opt-in — and only about **a dozen** ask for permission in Manual mode. The split is the design: tools that only read run freely; tools that mutate state or leave the machine are gated. Every tool name doubles as the exact string you use in permission rules, hook matchers, and subagent tool lists — so knowing the inventory _is_ knowing the control surface.

## The five categories

| Category | Tools |
| --- | --- |
| File operations | `Read` · `Edit` · `Write` · `NotebookEdit` |
| Search | `Glob` (file names) · `Grep` (file contents) — **absent by default on macOS, Linux, and WSL**; there Claude runs `find` and `grep` through `Bash` |
| Execution | `Bash` · `PowerShell` · `Monitor` |
| Web | `WebFetch` · `WebSearch` |
| Code intelligence | `LSP` — definitions, references, type errors; needs a language plugin |

The rest are orchestration: `Agent` spawns subagents, `Skill` runs skills, `Workflow` runs dynamic workflows, `EnterPlanMode`/`ExitPlanMode` gate planning, plus session utilities — scheduled prompts, worktrees, MCP resource readers, `ToolSearch`. The task checklist (`TaskCreate`/`TaskUpdate`/`TaskList`) is offered by default only to older models since v2.1.268; `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` opts newer ones in.

## Which ones ask

| Tool | What it does | Asks |
| --- | --- | --- |
| `Read` | File contents with line numbers; also images, PDFs, notebooks | No |
| `Glob` | Find files by name pattern | No |
| `Grep` | Search file contents (ripgrep) | No |
| `LSP` | Type errors, go-to-definition, references | No |
| `Agent` | Spawn a subagent in its own context window | No |
| `Edit` | Exact string replacement in a file | **Yes** |
| `Write` | Create or overwrite a whole file | **Yes** |
| `Bash` | Run shell commands | **Yes** |
| `Monitor` | Watch logs or commands in the background | **Yes** |
| `WebFetch` | Fetch a URL, extract via prompt | **Yes** |
| `WebSearch` | Web search — titles and URLs only | **Yes** |
| `Skill` | Execute a skill | **Yes** |
| `Workflow` | Run a dynamic workflow — a script orchestrating many subagents | **Yes** |

`Agent` deserves a note: launching a subagent never prompts, but the subagent’s own tool calls are checked against your permission rules as they happen. A subagent with `Bash` in its list still cannot run an unapproved command silently. And the Asks column is Manual mode: in `auto`, the starting mode since v2.1.284, a classifier answers instead of you (→ Ch. 05).

## Behaviours that bite

The inventory tells you what exists. These four mechanics change how sessions actually go.

### Edit enforces read-before-edit

Three checks must pass:

1. Claude read the file _in the current conversation_. Opus 4.6, Haiku 4.5, and older models always need the read; newer ones may edit an unread file when reading it would not prompt. A file that changed on disk since can still be edited when `old_string` matches its current content exactly and once.
2. `old_string` matches exactly. One whitespace character off is a miss.
3. It appears exactly once — unless `replace_all` is set.

Viewing a file with plain `cat`, `head`, `tail`, `sed -n`, or `grep` on a single file also counts as the read; piped output does not. `Write` has the same rule for overwrites — new files are exempt.

### Glob and Grep: absent by default, and they disagree about `.gitignore`

On macOS, Linux, and WSL, Claude Code leaves both tools out of the default set: Claude searches with `find` and `grep` through `Bash` (embedded `bfs` and `ugrep`), so searches reach your hooks and permission rules as `Bash` calls. They come back when you name `Glob` or `Grep` in `--tools` or `--allowedTools`, or when `Bash` is removed from the session.

Where they exist, `Grep` respects `.gitignore`: gitignored files are invisible to content search unless Claude passes the path directly. `Glob` does not by default (`CLAUDE_CODE_GLOB_NO_IGNORE=false` changes that), so a name-pattern search happily returns files — sorted by modification time, capped at 100 — from `node_modules` equivalents that `Grep` would skip. `Grep` also uses ripgrep regex syntax, not POSIX grep.

### Bash is stateless between commands

Each command runs in a separate process. `export` does not survive to the next call, and `cd` carries over only from a foreground command that exits 0 and ends inside the project or an added directory — otherwise the shell resets with a `Shell cwd was reset` notice. Two limits bound every command: a **2-minute default timeout** (Claude can request up to 10) and **30,000 characters** of output, beyond which the full output lands in a file and Claude gets the path plus a preview. A failing command gets only about 10,000 characters inline, head and tail, with no file path; `bashOutputMaxChars` (v2.1.261+, up to 128,000) raises the inline ceiling.

### WebFetch is lossy by design

It does not hand Claude the page. A small, fast model reads the fetched content and answers Claude’s extraction prompt; _that answer_ is all Claude sees. A result claiming a page “doesn’t mention” something may only mean the prompt didn’t ask — `Ctrl+O` shows the question Claude asked and the answer it got. Responses cache for 15 minutes, and cross-host redirects are returned rather than followed. `WebSearch` is the complement: titles and URLs only — reading any of them takes a follow-up fetch.

## Restrict and disable: `ToolName(specifier)`

Tool names are the permission vocabulary. The same rule format works in `settings.json`, the `--allowedTools`/`--disallowedTools` CLI flags, subagent frontmatter, skill `allowed-tools`, and hook conditions.

| Rule | Applies to | Matches |
| --- | --- | --- |
| `Bash(npm run *)` | Bash, Monitor | Command patterns |
| `Read(~/secrets/**)` | Read, Grep, Glob, LSP | Path patterns |
| `Edit(/src/**)` | Edit, Write, NotebookEdit | Path patterns |
| `WebFetch(domain:example.com)` | WebFetch | Domains |
| `Agent(Explore)` | Agent | Subagent types |
| `mcp__server__tool` | MCP tools | Server and tool name |
| `WebSearch` | WebSearch | Whole tool only — no specifier |

```
{
  "permissions": {
    "allow": ["Bash(pnpm test *)", "WebFetch(domain:docs.astro.build)"],
    "deny":  ["Read(./.env*)", "Read(~/.ssh/**)", "WebSearch"]
  }
}
```

Three rules worth memorising: an `Edit(...)` allow **also grants `Read`** on the same path, so you don’t need both; a **bare tool name in `deny`** removes the tool entirely — the official way to switch one off; and a `Write(...)`, `Glob(...)`, or `Grep(...)` rule **with a path never matches** — write `Edit(...)` or `Read(...)` instead. A `Read` deny also blocks `Edit` and `Write` on the same path, new files included (v2.1.228+).

> **Warning — Where deny rules stop**
>
> `Read`/`Edit` deny rules also cover file commands Claude Code recognises inside Bash — `cat`, `sed`, `grep` — but not arbitrary subprocesses. A Python script that opens the file itself walks straight past them. OS-level enforcement takes the sandbox. → Ch. 05

## MCP tools: names now, schemas on demand

Custom tools arrive via MCP servers, and Claude Code defers their definitions by default: only tool names and server instructions load at startup, and the full schema loads through the built-in `ToolSearch` tool when Claude actually needs one. That default exists because definitions are expensive — Anthropic measured the same deferred-loading pattern on the API side cutting upfront tool tokens from ~77K to ~8.7K. `ENABLE_TOOL_SEARCH=auto:N` loads tools upfront while their definitions stay under N% of the window; `false` loads everything. Run `/context` for the breakdown and `/mcp` for per-server costs.
