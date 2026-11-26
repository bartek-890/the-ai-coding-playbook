# Effort, thinking, and fast mode

_V · Economics · Chapter 28_

Three separate controls, constantly mistaken for one setting. **Effort** scales every token Claude spends. **Thinking budget** caps reasoning tokens specifically, and only on older fixed-budget models. **Fast mode** changes neither — it pays a flat premium for the same model to answer up to 2.5× faster. Anthropic’s own documentation recommends stacking them, not picking one.

## The levers at a glance

| Lever | What it scales | Cost effect | How to set it |
| --- | --- | --- | --- |
| Effort level | All response tokens: text, tool calls, thinking | Lower level = fewer tokens = lower cost | `/effort`, `--effort`, `CLAUDE_CODE_EFFORT_LEVEL`, `effortLevel` |
| Thinking budget | Reasoning tokens only, fixed-budget models only | Thinking bills as output tokens | `MAX_THINKING_TOKENS`, `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` |
| Fast mode | **Nothing** — same tokens, same quality | Flat premium per token | `/fast`, `fastMode`, `Option+O` |

## Effort: the token-spend dial

Effort is a _behavioural signal, not a hard cap_. At low effort Claude still thinks on a genuinely hard step — just less than it would at high effort on the same step. It affects every token in the response, which is what makes it different from a thinking-only control: lower effort also means **fewer tool calls and terser explanations**, not just less reasoning. And the scale is calibrated per model — the same level name is not the same amount of thinking on two models.

| Level | What it trades | Typical use |
| --- | --- | --- |
| `max` | No constraint on token spend | Hard problems you want worked through without you, such as a vulnerability hunt. Prone to overthinking; test before adopting it broadly |
| `xhigh` | Extended capability for long-horizon work | Agentic tasks over 30 minutes, million-token budgets |
| `high` | Balanced depth and cost | Work where verification matters or edge cases are likely, such as a bug in an existing codebase |
| `medium` | Moderate savings | Day-to-day engineering with a clear scope, such as implementing a feature |
| `low` | Biggest savings, some capability loss | Quick exchanges where you review every result, and **subagents** — Anthropic names them explicitly |

Availability varies: Fable models, Opus 5.5, Sonnet 5.5, Haiku 5.5, Opus 5, Sonnet 5, Opus 4.8, and Opus 4.7 support all five levels; Opus 4.6 and Sonnet 4.6 skip `xhigh`; **Haiku 4.5 has no `effort` parameter at all** and returns an error if you send one.

**The default is per model, and it moved.** In Claude Code, Fable 5.2, Opus 5.5, Sonnet 5.5, and Haiku 5.5 all default to `medium`; Opus 4.7 defaults to `xhigh`; the rest (Opus 5, Opus 4.8, Sonnet 5) to `high`. On the Claude API the defaults differ — Fable 5.2 and Sonnet 5.5 default to `high` — so a Claude Code session and your own API call can run the same model at different levels. In Anthropic’s testing Opus 5.5 at `medium` matches or beats Opus 5 at `high`, but at a given level it tends to think more per turn, so do not carry an Opus 5 level over; test your own work at each. Levels you saved before v2.1.280 (the old top-level `effortLevel`) do not apply to Opus 5.5 — it starts at `medium` until you pick a level for it.

> **Note — Precedence, and two names that aren’t levels**
>
> `CLAUDE_CODE_EFFORT_LEVEL` wins over everything. Below it: a level picked in the session (`/effort`, the `/model` picker), then `--effort`, then your saved settings (per model, or `effortLevel`), then the model’s default. A skill’s or subagent’s `effort:` frontmatter applies while it runs; `CLAUDE_CODE_SUBAGENT_EFFORT` (v2.1.316) sets the default for subagents and workflow agents that name none. Confirm what you got with `/effort status`. `Enter` in the slider saves a level as your default for that model; `s` applies it to this session only (v2.1.257); `max` is session-only unless set through the environment variable.
>
> **`ultracode`** is not a level but a Claude Code setting: with it on, Claude plans a dynamic workflow for each substantive task at whichever effort the session runs. Toggle it in the `/effort` slider (`Tab`) or with `/effort ultracode on|off` (v2.1.284); `--effort ultracode` also starts the session at `xhigh`. Every request then costs more tokens and time. **`ultrathink`** dropped anywhere in a prompt requests deeper reasoning for that one turn without touching the session setting — and on Opus, Sonnet and Haiku 5.5 and Fable 5.2/5.1 it also raises that turn to `xhigh` (v2.1.311). “Think hard” and “think more” are ordinary prompt text, not keywords. `CLAUDE_CODE_DISABLE_ULTRATHINK=1` turns the keyword off.

## Extended thinking: adaptive versus fixed budget

Fable models, Sonnet 5 and later, Haiku 5.5, and Opus 4.7+ always use **adaptive reasoning** — the model decides per step whether to think and how much, guided by effort. On these, a nonzero `MAX_THINKING_TOKENS` is ignored; effort is the only real dial. Opus 4.6 and Sonnet 4.6 default to adaptive too but can revert with `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1`, at which point `MAX_THINKING_TOKENS` sets an actual ceiling.

```
# session-wide spend dial — on adaptive models, the only thinking control
export CLAUDE_CODE_EFFORT_LEVEL=medium

# fixed-budget models only: revert from adaptive, then set a ceiling
export CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1
export MAX_THINKING_TOKENS=8000
```

`MAX_THINKING_TOKENS=0` turns thinking off entirely on the Anthropic API on older models, but not on the current ones. **Opus 5.5, Sonnet 5.5, Haiku 5.5, and the Fable models cannot disable thinking under any setting** — the session toggle reads “Thinking can’t be turned off”, and `alwaysThinkingEnabled: false` and `MAX_THINKING_TOKENS=0` have no effect. (On the API, Sonnet 5.5’s nearest equivalent is `between_tools`, accepted at `high` effort or below.) On **Opus 5**, disabling thinking is only accepted at effort `high` or below; at `xhigh` or `max` the request returns a 400, and Claude Code sends `high` instead when thinking is off. Anthropic’s own guidance is to leave thinking on and control cost with a lower effort level instead. There is a correctness reason as well as a cost one: with thinking disabled, Opus 5 occasionally writes a tool call into its text output instead of emitting a proper tool-use block. → Ch. 27

Thinking tokens **bill as output tokens whether or not you ever look at them** — collapsed by default, expandable with `Ctrl+O` (interactive sessions on the Anthropic API receive redacted thinking blocks unless `showThinkingSummaries` is on). That is the direct link to cost: a model at `high` effort with adaptive reasoning can spend tens of thousands of thinking tokens before writing a single word of visible output.

## Fast mode: paying for latency

A Claude Code setting that runs the same model through a different API configuration prioritising speed. **Not a cheaper or smaller model** — output quality is identical; only arrival speed and per-token price change. Available on Fable 5.2 (v2.1.321) and Opus 5.5; Opus 5 and Opus 4.8 remain research preview.

| Model | Fast-mode in / out per MTok | Notes |
| --- | --- | --- |
| **Opus 5.5** | $8 / $40 | Up to 2.5× faster. The fast-mode default in v2.1.280+ |
| Fable 5.2 | $16 / $80 | Needs v2.1.321+. `"fastMode": true` alone does nothing here — run `/fast` while on Fable 5.2 |
| Opus 5 | $10 / $50 | Was the default in v2.1.219–2.1.279 |
| Opus 4.8 | $10 / $50 | **Removed 2026-11-30** — see below |
| Sonnet · Haiku | — | **Not supported.** Enabling fast mode switches you to Opus. |

Each rate is a flat 2× the model’s standard price, and it is flat across the full 1M-token window. Toggle with `/fast` (then `Space`, `Enter`), `/fast on|off`, `Option+O`, or `"fastMode": true`. The VS Code extension has a **Toggle fast mode** command when the model supports it. Turning it off leaves you on the same model — it does not revert; use `/model` for that.

> **Warning — Opus 4.8 fast mode ends on 2026-11-30**
>
> Fast mode for Opus 4.8 was deprecated 2026-10-30 and is removed 2026-11-30. After that date a session still on Opus 4.8 with fast mode on gets its requests **rejected by the API, not served at standard speed**. Before then: `/model` to Opus 5.5 (v2.1.280+; fast mode stays on), or `/fast off` to stay on 4.8 at standard speed. If a new session still starts on 4.8, something with higher priority than your `/model` choice is setting it. Opus 4.7 has no fast mode at all (removed 2026-07-24); since v2.1.221, switching to it turns fast mode off instead of failing.

Three operational details that surprise people. On subscription plans, fast mode **draws from usage credits even when you have plan usage remaining** — those tokens do not count against your included usage and bill at the fast rate from the first token; credits must be on (`/usage-credits`), and Team and Enterprise need an Owner to enable the feature first. It has **separate rate limits** from standard speed; when you hit them, fast mode falls back to standard speed on its own, the `↯` icon greys out, and it re-enables after the cooldown — nothing breaks, you just quietly stop paying the premium. And if credits run out, Claude Code retries at standard speed and pricing and turns fast mode off.

> **Warning — Turn it on at the start of a session, not the middle**
>
> The **first fast-mode message in a conversation pays the full, uncached fast-mode input price for the entire existing context** — not just the new message. Fast mode and standard speed keep separate caches, so each later switch re-reads part or all of the conversation uncached again. Switching on midway through a long session means paying the premium on everything you have already accumulated.
>
> Fast mode also carries over into future sessions by default (on Fable 5.2, only if you turned it on while using Fable 5.2). Set `fastModePerSessionOptIn: true` to make every new session start with it off.

## How to combine them

- **SIMPLE TASK, WANT IT NOW** — Low effort + fast mode — fewer tokens, each arriving faster
- **QUALITY MATTERS MOST** — Drop fast mode, raise effort
- **SUBAGENT WORK** — Low effort, cheap tier, no fast mode
- **LONG-HORIZON MIGRATION** — Opus at xhigh, full spec up front, no fast mode

The reasoning behind the second row is worth stating plainly: **paying more per token for identical output makes no sense next to a setting that changes what the model actually produces.** Fast mode buys wall-clock time and nothing else. Spend on it only when your own time is the constraint.
