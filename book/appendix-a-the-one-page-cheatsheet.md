# The one-page cheatsheet

_Appendix A_

### When it’s going wrong

- **Esc** — interrupt, keep work
- **Esc Esc** — rewind menu
- **Ctrl+C** — stop / clear / exit
- **/rewind** — restore code or thread
- **/clear** — reset to zero tokens
- **git revert** — anything multi-file

### Every session

- **/context** — what’s filling the window
- **/compact …** — summarise, with instructions
- **/plan** — before anything uncertain
- **/code-review** — before every commit
- **/ship** — PR → review → watch CI (v2.1.330+)
- **Shift+Tab** — cycle permission modes
- **Ctrl+O** — transcript viewer
- **Ctrl+G** — edit prompt / plan in $EDITOR

### Prefixes

- **/cmd** — command or skill
- **!cmd** — shell; output enters context
- **@path** — file mention, tab-completes
- **\ + Enter** — newline, works everywhere

### Starting a session

```
claude --permission-mode plan
claude -c              # continue
claude --resume        # pick
claude --worktree      # isolated
claude --safe-mode     # no config
claude -p "…" --output-format json
```

### Defaults worth knowing

- **Permission mode** — auto (v2.1.284); `claude -p` starts Manual
- **Model** — Opus 5.5 on every plan (v2.1.280)
- **Effort** — medium on Opus 5.5, Sonnet 5.5, Haiku 5.5, Fable 5.2
- **Window** — 1M tokens; auto-compact ~967K; idle compact after ~1 h
- **Search** — Glob/Grep absent on macOS/Linux/WSL; Bash `find`/`grep`

### The scoped prompt

```
Goal: <one sentence>
Touch only: <paths>
Do not touch: <paths>
Done when: <command exits 0>
Plan first.
```

### Survives compaction?

- **System prompt** — yes
- **Root CLAUDE.md** — re-injected
- **Auto memory** — re-injected
- **paths: rules** — no
- **Nested CLAUDE.md** — no
- **Skill bodies** — 5K each / 25K total
- **Skill descriptions** — rebuilt (v2.1.304+)
- **Plan-mode plan** — re-injected
- **Chat instructions** — no — put them in a file

### Which mechanism?

- **Every task, always** — CLAUDE.md
- **One area of the code** — paths: rule
- **A procedure** — skill
- **Noisy reading** — subagent
- **Must happen, always** — hook
- **External system** — MCP

### Unattended checklist

- **1** — verifiable /goal condition
- **2** — --max-turns
- **3** — --max-budget-usd
- **4** — --allowedTools allowlist
- **5** — sandbox / container tier
- **6** — commit before you start

### Stop rules

- **2 failed corrections** — /clear, better prompt
- **3 failed prompts** — abandon the session
- **8 Stop-hook blocks in a row** — Claude Code overrides
- **3 / 20 classifier blocks** — auto mode falls back
