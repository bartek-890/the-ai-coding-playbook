# Glossary

_Appendix E_

- **Agent view** — `claude agents`: one screen listing every background session, what each is doing, and which need you. Research preview.
- **Agentic harness** — The runtime around a language model providing tools, context management, and an execution environment. The model plans; the harness acts.
- **Agentic loop** — Gather → act → verify, cycled until the task is done.
- **Auto memory** — Notes Claude writes for itself across sessions, indexed by `MEMORY.md` (first 200 lines or 25 KB load at startup).
- **Auto mode** — The permission mode where a classifier model reviews each action instead of you. The built-in starting mode in the terminal and VS Code since v2.1.284 — a model’s judgement, not a rule, so hard limits belong in `deny` rules or hooks.
- **Cache-stable prefix** — The byte-identical front of a request later calls reuse at ~0.1× input price. Anything dynamic must sit after the cache marker.
- **Channel** — An MCP server that pushes external events into a running local session.
- **Checkpoint** — A snapshot of tool-made file edits taken before each user prompt; restorable with `/rewind`. Not a replacement for git.
- **Code Mode** — Experimental opt-in (v2.1.323, `CLAUDE_CODE_EXPERIMENTAL_CODE_MODE=1`): Claude writes JavaScript that calls several tools in one step. Terminal only; auto mode’s classifier reviews the code, not each call inside it.
- **Completion condition** — The plain-text end state a `/goal` run works toward, judged by a separate evaluator model.
- **Context engineering** — Curating what enters the context window each turn rather than relying on a bigger window.
- **Effort** — A parameter scaling all response tokens — text, tool calls, and thinking together.
- **Evaluator** — The small fast model that judges a `/goal` condition after every turn. It does not call tools.
- **Fast mode** — An Opus- and Fable-tier setting buying up to 2.5× speed at a higher per-token price, billed through usage credits on subscriptions. Identical output quality.
- **Fork** — A subagent that inherits the parent conversation and system prompt — the exception to subagent isolation.
- **Goal** — `/goal <condition>` (v2.1.139): Claude keeps working turn after turn until a separate evaluator confirms the condition holds, judges it impossible, or an error you must fix clears it.
- **Hook** — Code that runs at a fixed lifecycle event regardless of what the model decides. The only deterministic layer in the harness.
- **Idle compaction** — Summarising a 200K+ token conversation left idle about an hour, just before the one-hour prompt cache expires, so the next message sends a summary instead of the whole history (v2.1.299). `"idleCompaction": false` turns it off.
- **Lost in the middle** — The observed failure to reliably use information placed mid-context; recall is strongest at the start and end.
- **MCP** — Model Context Protocol — the open standard for connecting external tools and data to an agent.
- **Mod** — A plugin that runs JavaScript inside Claude Code itself (v2.1.287): it can draw panes, add commands, and guard or rewrite tool calls — more than a settings hook can reach. Runs with your permissions; install only from sources you trust.
- **One-way door** — A decision that is irreversible or nearly so; a two-way door is one you can reopen and walk back through. Reversal cost is knowable before you prompt, unlike speed, which makes it the axis worth routing a delegation on.
- **Permission mode** — The session setting governing which actions run without asking: `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions`.
- **Plan mode** — A permission mode where Claude reads and plans but makes no edits until you approve. Prompt-enforced, not sandboxed.
- **Progressive disclosure** — Keeping the always-loaded surface small and pushing detail into files that load on demand.
- **Skill** — A markdown file whose body loads only when used, packaging a workflow or reference material.
- **Startup tax** — Tokens loaded into the window before your first message, paid on every new session.
- **Subagent** — A delegated agent running in its own context window, returning only a summary.
- **Verifiable completion condition** — A stop rule with one measurable end state the model can prove from its own output — the difference between a loop that terminates and one that spins.
