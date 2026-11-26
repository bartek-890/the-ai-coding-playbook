# MCP

_III · Harness · Chapter 15_

The Model Context Protocol connects Claude Code to external tools and data — issue trackers, databases, design files, monitoring. Connect a server when you find yourself _copying data into chat from another tool_. That is the whole trigger. Everything else is configuration.

## Adding servers

```
# remote HTTP (most common for hosted services)
claude mcp add --transport http notion https://mcp.notion.com/mcp

# with a header
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer TOKEN"

# remote SSE
claude mcp add --transport sse asana https://mcp.asana.com/sse

# local stdio — note the `--` separator
claude mcp add --env AIRTABLE_API_KEY=KEY --transport stdio airtable \
  -- npx -y airtable-mcp-server
```

In JSON configuration the `type` field accepts `streamable-http` as an alias for `http`, so definitions copied from a server’s own docs work unmodified. WebSocket (`type: "ws"`) exists for servers that push events unprompted, but supports neither OAuth nor the `--transport` flag — header auth only.

Already have the server’s JSON? `claude mcp add-json <name> '<json>'` takes it as is. To keep a token out of your shell history and process list, pipe it instead — `cat server.json | claude mcp add-json demo -` (v2.1.327; run it outside a Claude Code session).

## Scopes and precedence

| Scope | Loads in | Shared | Stored in |
| --- | --- | --- | --- |
| **Local** (default) | Current project only | No | `~/.claude.json`, under the project entry |
| **Project** | Current project only | Yes, via git | `.mcp.json` in the project root |
| **User** | All your projects | No | `~/.claude.json` |

```
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

Project-scoped servers from `.mcp.json` require approval before use — a checked-in file is code someone else can add. `claude mcp reset-project-choices` clears those decisions. Some names are reserved for built-ins and will be skipped with a warning: `workspace`, `claude-in-chrome`, `computer-use`, `Claude Preview`, `Claude Browser`.

## Using what a server provides

- **TOOLS** — Named mcp__<server>__<tool> — the string for permission rules and hook matchers
- **RESOURCES** — Reference with @server:resource in a prompt
- **PROMPTS** — Exposed as slash commands: /mcp__server__prompt
- **CHANNELS** — Push events into a live session — see below
- **SKILLS** — Beta, v2.1.332: the server’s own instructions, as /<server>:<skill> → Ch. 12

`/mcp` manages connections and runs OAuth flows; `claude mcp login <name>` does it from the command line. An MCP tool call still running after two minutes moves to a background task rather than blocking the session; `CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS` changes the threshold, `0` turns it off.

## Tasks and elicitation: servers that run long and ask back

- **MCP tasks** (terminal, v2.1.319). A server can run a long job itself — a report export, a build — and hand back a task instead of blocking. Claude keeps working, the footer shows `1 MCP task`, and when the job ends the result goes to Claude, not the transcript; ask Claude for it. `/tasks` lists the task; select it and press `x` to stop it. A task on a remote server keeps running after you exit — `claude --resume` the same session to follow it again; a stdio server’s process dies with Claude Code, so stop its tasks first. Tasks need an HTTP or stdio server connected on protocol revision 2026-07-28 and are on for terminal, Agent SDK, and GitHub Actions sessions.
- **Elicitation.** A server can stop mid-task and ask you for structured input — **form** mode (fields the server defines) or **URL** mode (open a link for a sign-in that finishes in the browser). No configuration; the dialog just appears. To auto-answer without a dialog, use the `Elicitation` hook. → Ch. 14 In `claude -p` and SDK runs nobody can answer, so Claude Code asks the server to cancel the task.
- **The client moved.** Claude Code runs MCP on SDK 2.0 (the v1 client is gone; `MCP_SDK_GENERATION=v1` now means a compatibility mode, v2.1.323), and negotiates the newer protocol revision with servers that support it. A server that does shows a **Protocol** row on its `/mcp` page.

## The cost, and how to keep it down

Every connected server costs tool names at startup. Schemas are deferred by default and load through `ToolSearch` on demand — the pattern Anthropic measured cutting upfront tool tokens from ~77K to ~8.7K. `ENABLE_TOOL_SEARCH` tunes it: `auto:N` loads definitions upfront while they total under N% of the window and defers them beyond that, `false` loads everything. Three habits:

- **Disconnect servers you are not using this week** rather than denying their tools. Only connected servers cost anything.
- **Scope a noisy server to a subagent** with the `mcpServers` frontmatter field, so its names never reach your main window.
- **Check `/mcp` for per-server cost** before blaming the model.

> **Warning — MCP servers run outside every sandbox tier below the runtime**
>
> Each server is code you run alongside Claude Code. Anthropic does not security-audit third-party servers. Gate them with permission rules, prefer servers you wrote or trust, and remember that `/sandbox` does not cover them — only `@anthropic-ai/sandbox-runtime`, a container, or a VM does. → Ch. 05

## Channels: the outside world knocking

A **channel** is an MCP server that declares `claude/channel` and fires notifications into a _running local session_ — a CI failure, a Telegram message, a webhook. It is the inverse of ordinary MCP, where Claude queries on demand.

| Feature | What it does | Good for |
| --- | --- | --- |
| **Channels** | Push events into your running local session | Chat from your phone; CI into work in progress |
| Claude Code on the web | Fresh cloud sandbox from GitHub | Async tasks you check later |
| Standard MCP | Claude queries on demand during a task | Reading systems when needed |
| Remote Control | Drive a local session from claude.ai or mobile | Steering, not event forwarding |

Two gates must both pass: the server must be in MCP config **and** named on `--channels`, and the sender must be allowlisted. `.mcp.json` alone does not inject.

```
# in Claude Code
/plugin install telegram@claude-plugins-official
/telegram:configure <bot-token>

# restart with the channel active
claude --channels plugin:telegram@claude-plugins-official

# pair, then lock it down
/telegram:access pair <code>
/telegram:access policy allowlist
```

Requires v2.1.80+; permission relay — approving tool use from your phone — requires v2.1.81+. Research preview, Anthropic auth only. `fakechat` runs a localhost UI at `:8787` for testing the pipe without tokens. A channel server that negotiates MCP revision 2026-07-28 is not registered as a channel — that revision cannot carry channel messages — so keep stdio channel servers on the earlier handshake with `MCP_PROTOCOL_NEGOTIATION=legacy`.

> **Warning — Check the sender, not the room**
>
> In group chats `message.from.id` and `message.chat.id` are different fields. Allowlisting the _room_ means anyone in that group can prompt-inject your session. And permission relay means anyone who can reply through the channel can approve tool use — turn it on only when every allowlisted sender is someone you would hand the keyboard.
