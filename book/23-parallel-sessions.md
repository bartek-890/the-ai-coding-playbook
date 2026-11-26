# Parallel sessions

_IV · Flows · Chapter 23_

Everything so far assumes one human, one Claude, one conversation. Claude Code scales horizontally, and the payoff is not only throughput: **a fresh context reviews better than the context that wrote the code**, because it is not biased toward the reasoning that produced it.

## Ways to run in parallel

| Approach | What it gives you | Coordination cost |
| --- | --- | --- |
| **Git worktrees** | Separate CLI sessions in isolated checkouts so edits cannot collide | You coordinate |
| **Agent view** (`claude agents`, research preview) | One screen to dispatch background sessions, see which need you, and peek or attach; each session moves into its own worktree before it edits | You dispatch; it tracks |
| Desktop app sessions | Multiple local sessions managed visually, each in its own worktree | You coordinate, visually |
| Claude Code on the web | Anthropic-managed isolated VMs | You coordinate; no local setup |
| Agent teams | A team lead, shared tasks, and messaging between sessions. Experimental, off by default; each teammate runs its own context window | Claude coordinates |
| Dynamic workflows (`/workflows`) | A script Claude writes and a runtime executes: dozens to hundreds of subagents, results cross-checked, rerunnable. `/deep-research` is the bundled one | The script coordinates |
| Projects (claude.ai/code, public beta on Pro, Max, Ultra) | One conversation that starts parallel cloud threads, shares repo instructions and memory, and keeps running with your laptop closed | Claude coordinates; not on Team or Enterprise |

## Worktrees: the mechanical foundation

```
# Claude Code can do it for you
claude -w oauth

# or by hand, one checkout per workstream
git worktree add ../proj-oauth   -b feat/oauth
git worktree add ../proj-cleanup -b chore/cleanup

# then a session per directory
cd ../proj-oauth   && claude --name oauth
cd ../proj-cleanup && claude --name cleanup
```

Each session has its own working tree, its own branch (`worktree-<name>`), and its own context. They cannot overwrite each other’s edits, which is the failure that makes naive parallel sessions worse than serial ones. A fresh worktree has no gitignored files: list `.env` and the like in a `.worktreeinclude` at the repo root and Claude Code copies them into every worktree it creates (use `worktree.symlinkDirectories` for big directories, not the include file). A subagent can also take `isolation: worktree` in frontmatter to get the same guarantee automatically. → Ch. 13

> **Note — Name your sessions**
>
> `--name oauth-migration` or `/rename`. Treat sessions like branches: each workstream gets its own persistent context, and `claude --resume` becomes a list you can actually navigate instead of a wall of timestamps.

## The writer/reviewer pattern

1. **Session A implements**

   ```
   Implement a rate limiter for our API endpoints. Follow the
   middleware patterns in @src/middleware/. Tests included.
   ```

2. **Session B reviews — with no knowledge of how it was written**

   ```
   Review the rate limiter implementation in
   @src/middleware/rateLimiter.ts. Look for edge cases, race
   conditions, and consistency with our existing middleware patterns.
   Report only issues affecting correctness or the stated requirements.
   ```

3. **Session A addresses the findings**

   ```
   Here's the review feedback: [paste Session B output].
   Address these issues. Re-run the tests.
   ```

The same shape works with tests: one Claude writes the tests, another writes code to pass them. And it works _inside_ one session with a reviewer subagent, which is cheaper — the implementing session receives the gaps directly and can fix and re-review without you copying findings between windows.

**Prompt · In-session adversarial review · before you call it done**

```
Use a subagent to review the rate limiter diff against PLAN.md. Check that
every requirement is implemented, the listed edge cases have tests, and
nothing outside the task's scope changed.

Report gaps, not style preferences. Flag only what affects correctness
or the stated requirements.
```

Or just run the bundled skill: `/code-review` reviews your branch’s changes for correctness bugs in a background subagent that never saw your conversation, and returns findings; `--fix` applies them, `--comment` posts them on the PR. Cleanup is the separate `/simplify`. `/code-review ultra` runs the deeper cloud review.

## What the second pass actually returns

The reason to do this is not that it produces better code, though it does. It is that **it is what lets you trust work you did not watch get written**. The agent already handed you the time; a second pass is the obvious thing to spend it on.

Held to a sample worth quoting: across **83 completed tickets** put through an independent second-model review, **67** ended in a real change to the code — something the first pass got wrong or left exposed, caught before it shipped. Four reviews in five find something. Sorted by what kind of defect it was, across the 122 commits those reviews drove:

| What the review caught | Share of commits |
| --- | --- |
| Correctness / logic | **34%** |
| Concurrency / races | **22%** |
| Durability / data | **19%** |
| Security / injection | **9%** |

Read the middle three together: **half of every commit these reviews drove was the incident kind** — the defect that corrupts data, loses a write, or leaks a secret, not the one that misaligns a button. Those are also precisely the defects a human skim of a diff does not catch, because they are invisible in the change and only exist in the interaction between the change and everything around it.

Two rules keep the pattern honest. The reviewer is **read-only** — it raises findings and touches nothing, so it cannot quietly rewrite the thing it is judging. And the author decides: a finding is an argument, not an instruction, and a good author session pushes back on the ones that are wrong.

**Prompt · The reviewer brief · a session with none of your context**

```
You're reviewing the change in this working directory. You are read-only:
review and discuss only — do not edit files, commit, push, or reset the
tree.

Run `git rev-parse HEAD` and `git status` first, and anchor every finding
to that commit.

Write findings as a numbered list. For each: the file and line, what
breaks, and the input or interleaving that breaks it. Prioritise
correctness, concurrency, data durability, and injection over style.
```

> **Note — Use a file as the mailbox**
>
> Nothing here needs a live channel between the two sessions. Have the reviewer write its findings to `REVIEW.md` or a PR comment, then hand that file to the author session to work through. The review becomes a document that gets passed along, which means it survives a session ending, can be re-run against the fixes, and leaves a record of what was raised and what was rejected.

## Background sessions

| Command | Effect |
| --- | --- |
| `claude --bg "…"` · `/background` (`/bg`, or `←` on an empty prompt) | Start detached and return immediately, or detach the current session |
| `claude agents` | Agent view — every background session grouped by state (working, needs input, done), with a peek panel (`Space`) and PR status on the row |
| `/fork [prompt]` | Copy this conversation into a new background session and keep working here; the copy makes its edits in a worktree of its own |
| `claude attach <id>` | Attach to a background session in the terminal |
| `claude logs <id>` | Recent output without attaching |
| `claude stop` · `respawn` · `rm` | Stop, restart with conversation intact, or remove from the list |

Each background session draws on your plan independently, and a dispatched one keeps running with the terminal closed. Parallel runs multiply tokens: subagents, workflow agents, and agent teammates each send their own requests, and `/usage` shows the split. → Ch. 29

> **Warning — Parallelism multiplies mistakes as readily as output**
>
> Three sessions running unattended against the same repo without worktrees will produce a merge conflict at best and a silently lost edit at worst. Isolate first, then parallelise. And keep the review session’s context _clean_ — a reviewer that has been told how the code was written is no longer independent.
