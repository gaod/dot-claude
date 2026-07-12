---
name: delegate-work
description: Use when dispatching work to a subagent — deciding whether to dispatch at all, choosing subagent type, model, and effort, and writing the delegation prompt from templates.md — and when the current approach keeps failing (escalate the model, change course, or run multi-answer judging). Trigger before any Agent tool call, and whenever two consecutive fixes haven't worked.
---

# Delegate Work — dispatch rules, model selection, escalation

> Dispatch rules for the main-conversation model (the commander). Factual basis: the `task-kickoff` skill. Delegation prompt templates: `templates.md` in this directory.

## Core principle: the commander stays off the field

The main conversation's context is the most expensive resource — once it balloons and compaction triggers, judgment quality degrades for the rest of the session.
Therefore: **dispatch the grunt work; only conclusions enter the main conversation.**

### Situations that MUST be dispatched to a subagent

- Questions expected to require reading **10+ files** or **2000+ lines** to answer
- Full-repo scans (find all occurrences of a pattern, inventory a class of files)
- Web/documentation research (official docs, version-diff comparisons)
- Batch same-shape edits (≥5 files getting the same mechanical change)
- **Acceptance, when a verifier trigger fires** (triggers: `verify-deliverable` skill; rule: "The producer's judgment is never acceptance evidence" below)

### Situations NOT to dispatch (cheaper to do yourself)

- Reading or editing 1–3 files at known paths
- Point lookups (answerable with a single `rg` or `fd`)
- Judgments that need the full conversation context to get right (dispatching means re-explaining the context, which costs more; when context truly is needed, use `subagent_type: "fork"`)

Every spawn is a cold start: the subagent knows nothing of what you've discussed and must re-read the environment. Before dispatching, ask yourself: "is the cost of briefing the task < the context cost of doing it myself?"

## The delegation trio (every delegation prompt must contain)

1. **Goal and motivation**: what to do and why (motivation lets the agent make the right call in edge cases)
2. **Acceptance criteria**: mechanically checkable completion standards ("find all X and list file:line" — not "look into X")
3. **Report format**: state explicitly what to return and what not to return

## Report contract (write into every delegation prompt)

- Return only: conclusions, `file_path:line`, suggested next steps
- Long artifacts (reports, large code blocks, lists > 30 lines) **go to a file in the scratchpad or a specified path; return the path** — don't paste the full text
- Clearly mark "verified facts" vs "speculation"; if something can't be found, say so — never fabricate

## Choosing model and effort

Specify the model via the `model` parameter on the Agent call; effort cannot be set per-call — it comes from the agent definition's frontmatter `effort` field (`low`/`medium`/`high`/`xhigh`/`max`; available levels depend on the model) or the session setting. For recurring roles needing a specific effort, create a definition in `~/.claude/agents/*.md` (`verifier` already exists, see `templates.md`).

| Task | model | Rationale |
|------|-------|------|
| Mechanical search, file location, format conversion | `haiku` | Cheap and fast; mistakes are easy to spot |
| General implementation, research, batch edits | `sonnet` | The default; good enough |
| Architecture trade-offs, high-risk judgment, adversarial review | `opus` (or the strongest model available) | Judgment quality decides the outcome |
| Questions about Claude Code / the Claude API | `claude-code-guide` agent | It checks official docs; don't answer from memory |

Note: the available agent types and model values are whatever the current session's system-reminder lists — commonly `claude` / `general-purpose` / `Explore` / `Plan` / `claude-code-guide` / `fork`, but the list changes across versions; never specify a value not on the list.

## When to escalate the model (or change approach)

Any one of these signals means escalate (haiku→sonnet→opus) or switch to multi-answer judging (below):

- The same bug is still there after 2 fixes
- You catch yourself *guessing* API behavior or framework semantics instead of verifying
- Two consecutive edits went in opposite directions (changed it, then changed it back)
- The task demands taste or trade-offs (naming, API design, UX copy) and there's no existing convention to copy

✅ Positive: fixing a UI framework state-sync bug, CI still red after the second attempt — stop, write up the symptoms, both attempts, and the relevant files into a prompt; dispatch opus or ask the user.
❌ Negative: tweaking the same parameter line a third time and pushing to CI "to see if it passes this time" — that's using CI as dice.

## Escalation / de-escalation paths

- **Escalate**: haiku fails the same task twice → re-dispatch with sonnet (don't let haiku try a 3rd time). Sonnet produces output you can't judge, or two edits go in opposite directions → opus, or ask the user.
- **De-escalate**: for batch same-shape tasks, run the first item with sonnet to prove the flow and output format, then let haiku do the rest from the template.
- **Multi-answer judging**: for high-risk judgments that can't be verified by tests (e.g., architecture selection), dispatch 2–3 independent agents for separate answers, then have the main conversation (or opus) judge and pick. Expensive — only for decisions where the cost of being wrong is > 3× the judging cost.

## Signals the direction is wrong (change course, don't retry)

On any of these signals: **stop → return to the last known-good state (git stash / checkout) → restate the problem → take a different path or escalate**:

- Fixing A broke B, fixing B broke C (repair chain ≥ 3 links)
- The diff touches more than 2× the files estimated at kickoff (no estimate written? use the files/modules the task description explicitly names as the baseline; touching modules the description never mentions counts as over)
- You start writing "work around the framework" hacks (swizzling, reflection, copying whole chunks of vendor code in to modify)
- You can't explain *why* it works now — only "it works when I change it this way"

✅ Positive: realizing the test can only pass by mocking half the system — stop, suspect the test is cut at the wrong layer, go back to the project's testing guidelines, re-pick the entry point.
❌ Negative: tests keep failing, so you loosen the assertions until they're green — that's not a fix, that's destroying evidence.

## The producer's judgment is never acceptance evidence (iron rule)

Low-risk work may close on mechanical evidence — diff read-back, `rg` counts, test output — per the tiers in the `verify-deliverable` skill. The producer's *opinion* ("looks fine") is never evidence at any tier. When a verifier trigger fires, acceptance leaves the producer entirely:

- **File deliverables** → dispatch a fresh agent for read-back: read the files, check each acceptance criterion
- **Code** → tests or a real run; if this machine can't run it (check `~/.claude/hosts.md`) → push and watch CI (`gh pr checks`) — only if pushing was authorized this session; otherwise report "locally verified" with the CI-trigger commands
- **High-risk judgments** → second opinion (an independent agent that hasn't seen the first answer; compare conclusions)
- The acceptance agent must have **fresh context** (a newly spawned `claude` or `verifier`), **never `fork`** — a fork inherits the main conversation's context, along with its biases and blind spots
- Give the acceptance prompt an "acceptance criteria list," not a leading framing like "I think it's done, please confirm"

## Relation to the harness's built-in guidance

The harness warns "don't spawn agents casually (expensive)" — that guards against dispatching for trivia. This skill's thresholds (10 files / 2000 lines / full-repo scan / verifier triggers) are precisely the "worth dispatching" line; there's no conflict: below the threshold do it yourself, above it dispatch.

## Verified findings (checked against official docs 2026-07-12; re-verify after version changes)

- **Subagent definition frontmatter** (supported in both `.claude/agents/*.md` and `~/.claude/agents/*.md`):
  Available fields: `name`, `description` (required); `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `effort`, `isolation`, `color`, `initialPrompt`.
  `effort` values: `low` / `medium` / `high` / `xhigh` / `max`; available levels depend on the model. Overrides the session effort while the subagent is active; default is inherit.
  `model` values: `haiku` / `sonnet` / `opus` / `fable` / full model ID / `inherit` (default `inherit`).
  `isolation: worktree` runs the subagent in a temporary git worktree (isolated copy of the repo; auto-cleaned if unchanged) — useful for broad delegated edits.
  (Source: code.claude.com/docs/en/sub-agents.md)
- **Built-in `Explore` inherits the main conversation's model** (v2.1.198+; capped at Opus on the Claude API) — it no longer always runs on Haiku. A user or project agent named `Explore` overrides the built-in; define one with `model: haiku` if you want predictably cheap exploration.
- **Built-in `Explore` and `Plan` skip CLAUDE.md files and git status.** Any convention the explorer must honor (e.g. "ignore `vendor/`") must be restated in the delegation prompt — it will not see the project rules. All other built-in and custom subagents load CLAUDE.md normally.
  (Source: code.claude.com/docs/en/sub-agents.md)
