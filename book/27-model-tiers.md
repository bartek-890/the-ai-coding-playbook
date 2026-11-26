# Model tiers

_V · Economics · Chapter 27_

The choice is an economics problem, not a benchmark one. Most teams default to the top tier for everything because it wins the leaderboard, or stay on the middle tier out of habit — and pay the wrong rate for most steps of an agent workflow either way. The spread runs as high as **80:1** between cheapest and most expensive.

## The stack at a glance

|  | Fable 5.2 | Opus 5.5 | Sonnet 5.5 | Haiku 5.5 |
| --- | --- | --- | --- | --- |
| API ID | `claude-fable-5-2` | `claude-opus-5-5` | `claude-sonnet-5-5` | `claude-haiku-5-5` |
| In / out per M tokens | $8 / $40 | $4 / $20 | $2 / $10 | $0.10 / $0.50 ($0.50 / $2.50 for prompts over 100K) |
| Cache read per M tokens | $0.20 (0.025×) | $0.20 (0.05×) | $0.10 (0.05×) | $0.01 (0.1×) |
| Context window | 1M | 1M | 1M | 1M |
| Max output | 128K | 128K | 128K | 128K |
| Latency | Moderate | Moderate | Fast | Fastest |
| Thinking | Adaptive, **always on** | Adaptive, **always on** | Adaptive | Adaptive |
| Default effort — API / Claude Code | `high` / `medium` | `medium` / `medium` | `high` / `medium` | `medium` / `medium` |
| Knowledge cutoff | Jun 2026 | Jun 2026 | Jun 2026 | Jun 2026 |
| Safety-classifier refusals | **Yes** — `stop_reason: "refusal"` on HTTP 200 | **Yes** | **Yes** | **Yes** — no server-side fallback |

From Anthropic’s model and pricing documentation, current as of 2026-11-26. The previous generation is still listed: Fable 5.1 and 5 at $10 / $50, Opus 5, 4.8, 4.7, 4.6 and 4.5 at $5 / $25, Sonnet 5 at $2 / $10 (the introductory price became permanent; the planned rise to $3 / $15 never happened), Sonnet 4.6 and 4.5 at $3 / $15, and Haiku 4.5 at $1 / $5 on a 200K window. Opus 4.1 and Opus 4 are retired except on some clouds. Verify before you commit a budget — this is the fastest-ageing table in the book.

> **Note — What Claude Code puts you on**
>
> `default` resolves to **Opus 5.5** on Pro, Max, Ultra, Team, Enterprise, and the API (v2.1.280; before that Pro and Team Standard defaulted to Sonnet 5). On the Anthropic API the aliases `opus`, `sonnet`, and `haiku` mean 5.5; some other providers lag (Bedrock and Agent Platform still resolve `sonnet` to 4.5, Foundry resolves `opus` to 4.6). `fable` means Fable 5.2 and is **never** the default anywhere — pick it with `/model fable` or `claude --model fable`, and expect “Requires usage credits” on that row on some plans. `opusplan` plans on Opus and executes on Sonnet. Unexpectedly high spend on a per-token plan usually traces to Opus left as the default.

> **Note — One tier most people will never see**
>
> **Claude Mythos 5.2** shares Fable 5.2’s pricing and is limited-availability, offered to organizations verified through Anthropic’s verification programs such as the Cyber Verification Program. Mentioned here so that a reference to it in someone else’s config is not a mystery.

## Four roles, four tiers

| Role in the workflow | Tier | Why |
| --- | --- | --- |
| **Frontier escalation** — the single hardest step, called directly | Fable 5.2 | Anthropic’s most capable models for demanding reasoning and long-horizon agentic work, at double Opus’s token rate and with a 30-day data-retention requirement |
| **Orchestrator** — plans, delegates, holds the long thread | Opus 5.5 | Built for long-running agentic coding and knowledge work at **half the cost of Fable 5.2**. In Anthropic’s testing, at its default `medium` effort it matches or beats Opus 5 at `high` on coding and knowledge-work evaluations |
| **Default worker** — most coding and knowledge tasks, solo | Sonnet 5.5 | Anthropic’s best speed-to-intelligence combination, at half Opus’s rate; same 1M window and 128K output |
| **Parallel worker** — fast, scoped, many at once | Haiku 5.5 | The cheapest rate and the fastest model, aimed at high-volume, latency-sensitive work: classification, extraction, routing. Built to run as a subagent. |

Encoded so scripts and subagent launchers can read it:

```
FRONTIER=claude-fable-5-2      # rare, hardest-step escalation only
ORCHESTRATOR=claude-opus-5-5   # long-horizon, high autonomy
DEFAULT=claude-sonnet-5-5      # daily driver
WORKER=claude-haiku-5-5        # parallel sub-agents
```

> **Note — The ranking daily users give you is not the leaderboard**
>
> Ask people who spend six hours a day inside these tools to rank the tiers and the order that comes back does not track SWE-bench, and often does not track code quality either. What it tracks is **stability, price, rate limits, latency, and trust that the thing will behave tomorrow the way it behaved today**. Neither ranking is wrong; they answer different questions. Benchmarks measure capability in a controlled run. Practitioners are reporting the cost of depending on something all day.
>
> The practical use of that: when a colleague says a model “is better”, ask which of the five they mean. Half the model-choice arguments on a team are two people ranking on different axes, and the answer is usually a routing rule rather than a winner.

## The decision rules

1. **Pick the default on purpose, then move off it with a reason**

   Claude Code starts you on Opus 5.5, but the best-practices guidance is that Sonnet costs less on routine work, and at half the rate the saving is real: `/model sonnet` for clear-scope work, `opusplan` to plan on Opus and execute on Sonnet. Run the default role at **medium effort or lower**. On Sonnet 5, the cost-performance curve overlapped Opus’s range at every effort level, but the advantage narrowed or reversed at high and xhigh — so escalate to Opus, not to Sonnet at xhigh, when you need frontier accuracy. Switching with `/model` also switches every subagent that inherits your model, so move to Opus _before_ Claude delegates a research pass, not after.

2. **Reach for Opus 5.5 when the task is long-horizon and expensive to get wrong**

   A multi-day refactor, a codebase-scale migration, an orchestrator keeping a plan straight across hundreds of tool calls. Two operational notes: **give it the whole spec up front** — a complete first turn produces more efficient _and_ more accurate runs than drip-feeding. And fast mode is an Opus-and-Fable lever Sonnet and Haiku do not have. → Ch. 28

   Do not carry over the effort level you used on Opus 5. Opus 5.5 starts at `medium` (Opus 5 started at `high`), and at a given level it tends to think _more_ per turn than Opus 5, especially at `xhigh` and `max`. Start at the default, move on the evidence of your own evals, and keep `xhigh` and `max` for work where you measured a gain. At those levels, set a large `max_tokens` — 128,000, the model’s maximum, worked well in Anthropic’s testing — so it has room to think _and_ act.

3. **Reserve Fable 5.2 for one step, not one session**

   Select it explicitly. Describe the outcome rather than the steps, hand it the ambiguous problems — a root-cause investigation, outage debugging, an architecture decision — and skip the verification reminders; it verifies its own work with less prompting. At double Opus’s rate, with a 30-day data-retention requirement and credit billing on some plans, it is an escalation tier, not a default.

   Refusals are no longer a Fable-only integration cost: Fable, Opus 5.5, Sonnet 5.5, and Haiku 5.5 all carry safety classifiers that decline as a successful HTTP 200 with `stop_reason: "refusal"`, so every API integration should branch on it. In Claude Code the retry is built in: a flagged Fable or Opus 5.5 request re-runs on Opus 5 (biology) or Opus 4.8 (cybersecurity), a Sonnet 5.5 cybersecurity flag re-runs on Sonnet 5, and other cases end in a refusal. A repository with security or biology material can trip the classifier on workspace context alone — `claude --safe-mode` tells you whether your customizations are the trigger.

4. **Fan Haiku out, and keep each job small**

   Haiku 5.5 now has the 1M window and the same five effort levels, so the old constraints are gone. The one to design around is price: a prompt **over 100K tokens costs five times the rate** ($0.50 / $2.50). Give each worker a small self-contained job rather than a sprawling history, and the cheapest rate holds.

## Opus 5 and 5.5 change what you should write in your prompts

Most model upgrades are transparent — same prompts, better answers. These are not, and the differences are behavioural rather than API-level. Five you will notice without changing a line of code:

| What changed | What to do about it |
| --- | --- |
| **It verifies its own work unprompted** | **Remove verification instructions carried over from earlier models.** “Include a final verification step”, “use a subagent to verify” — these now cause _over_-verification. Anthropic’s Fable guidance says the same. |
| It delegates to subagents more readily | Expect more parallel work by default. Multi-agent runs show effective writer–verifier behaviour and few cases of agents overwriting each other. |
| It narrates progress more often | Fine when you are watching; noise in a piped `-p` run. Ask for terse output where that matters. |
| Responses and deliverables run longer | An output budget matters more than it used to. → Ch. 29 |
| A turn can end on a status report while work is still owed | Opus 5.5 can close a long turn with text — a summary that announces the next step — instead of taking it. Treat that as a report, not as done: state the completion condition and let `/goal` check it. → Ch. 22 |

## Moving to 5.5: the requests that now return 400

The 5.5 generation is the opposite kind of upgrade: the API changes, and old request code fails loudly. Anthropic lists four breaking changes for Opus 5.5 and five for Sonnet 5.5; each returns an HTTP 400 from any script, SDK call, or agent that still sends the old shape.

| Old request | On 5.5 | Fix |
| --- | --- | --- |
| `thinking: {"type": "disabled"}` | 400 on both | Opus: omit `thinking`, lower effort. Sonnet: send `{"type": "between_tools"}` at `high` effort or below |
| `"budget_tokens": N` | 400 on both | Adaptive thinking plus an effort level |
| `tool_choice` `any` or `tool` | 400 on both | `auto` plus `strict: true` on the tool, or structured outputs; say in the prompt when the tool applies |
| `computer_20251124` tool | 400 on Claude API and Google Cloud | `computer_toolset_20260801`. Amazon Bedrock still accepts the old tool |
| Thinking block replayed after editing earlier history | 400 on accounts created 2026-08-31 or later | Keep history append-only; to change instructions, append a new system message |
| Advisor tool with an Opus 4.8, 4.7 or Sonnet 5 advisor | 400 on Sonnet 5.5 | Use Opus 5.5, Opus 5, Fable, or Sonnet 5.5 itself |

`between_tools` is Sonnet 5.5’s lowest setting: no thinking up front, short progress notes between tool calls. It takes no other field, returns a 400 at `xhigh` and `max`, and prevents changing effort on individual messages.

The same direction holds beyond the API shapes above: **thinking cannot be turned off** on Opus 5.5, Fable, or Haiku 5.5, and forced tool use returns a 400 on Fable 5.2 as well. On Opus 5 the switch was still accepted at `high` or below. Lower the effort level where you used to disable thinking, and since `max_tokens` is a hard limit on _total_ output — thinking plus response text — revisit it for any workload sized for a thinking-free model.

> **Warning — The one that fails silently**
>
> On both 5.5 models, the notes written between tool calls arrive as `thinking` blocks, not `text` blocks. At the default `display: "omitted"` their text is empty, so a UI that streams them to users **goes quiet mid-turn with no error**. Set `thinking.display` to `"updates"` (beta), or on Sonnet use `between_tools`, which returns them with summary text.
>
> Related: a response can now _start_ with a `thinking` block. Code that reads `content[0]` as the answer breaks. Select blocks by `type`.

Refusals widen too. Opus 5.5 adds `bio` and `reasoning_extraction` to `cyber`; Sonnet 5.5 has five categories. The practical risk is `reasoning_extraction`: a prompt that asks the model to write its reasoning into the response can now be declined, and server-side fallback does not retry that category. Delete “show your reasoning” lines from prompts carried over from older models.

Two smaller wins are worth knowing. First, the **minimum cacheable prompt is 512 tokens on all four current models** (and on Opus 5), down from 1,024 on Opus 4.8 and Sonnet 5. Prompts that used to be silently too short to cache now cache, with no change to your code (→ Ch. 29). Second, mid-conversation changes keep the prompt cache: **tool changes** (beta, `mid-conversation-tool-changes-2026-07-01`) and **per-message effort** (beta) both avoid the penalty of editing the prefix, whereas changing the top-level `effort` between requests invalidates the cached messages. One tool list no longer has to be frozen for the life of a session.

## The orchestrator–worker pattern

The tiers compose. An Opus orchestrator delegates scoped sub-tasks to Haiku workers, with Sonnet as the solo default when no orchestration is needed and Fable called directly for the rare step that needs it.

|  |  |
| --- | --- |
| **Symptom** | A single big model reads a 30K-token API reference or a pile of source files, and that clutter stays in context for the rest of the run, dragging quality down. |
| **Cause** | Heavy reads done in the main loop persist on every subsequent turn. |
| **Solution** | Push the read into a subagent that runs in its own window and returns only the conclusion — spawned on Haiku, so you get the isolation _and_ the cheapest token rate for the grunt work. |

> **Key — Why the per-token spread matters more than the benchmark spread**
>
> In a fan-out architecture the **worker tier runs far more tokens than the orchestrator**. A cheaper worker is therefore a cheaper bill on the dominant cost line — and that spread runs up to 80:1 across the full stack (Fable 5.2 against Haiku 5.5 on prompts under 100K), 40:1 between Opus and Haiku, not just the 2:1 between the workhorse tiers. Tier selection is the only lever that changes your unit price without changing anything you do.

In Claude Code, the practical expression of all of this is one line of subagent frontmatter: `model: haiku`. Define an `Explore` override with it and every codebase survey in every session runs at the cheapest tier that handles the job. → Ch. 13 To set it once instead, `CLAUDE_CODE_SUBAGENT_MODEL=haiku` and `CLAUDE_CODE_SUBAGENT_EFFORT=low` (v2.1.316) become the default for every subagent and workflow agent that is not assigned one another way.
