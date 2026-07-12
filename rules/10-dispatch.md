# 10 — Model Dispatch Rules

> Dispatch rules for the main-conversation model (the commander). Factual basis: `00-diagnosis.md`. Delegation prompt templates: `30-delegation-templates.md`.

## Core principle: the commander stays off the field

The main conversation's context is the most expensive resource — once it balloons and compaction triggers, judgment quality degrades for the rest of the session.
Therefore: **dispatch the grunt work; only conclusions enter the main conversation.**

### Situations that MUST be dispatched to a subagent

- Questions expected to require reading **10+ files** or **2000+ lines** to answer
- Full-repo scans (find all occurrences of a pattern, inventory a class of files)
- Web/documentation research (official docs, version-diff comparisons)
- Batch same-shape edits (≥5 files getting the same mechanical change)
- **Acceptance, when a `20-judgment.md` §5 trigger fires** (see "The producer's judgment is never acceptance evidence" below)

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

Specify the model via the `model` parameter on the Agent call; effort cannot be set per-call — it comes from the agent definition's frontmatter `effort` field (`low`/`medium`/`high`/`xhigh`) or the global setting. For recurring roles needing a specific effort, create a definition in `~/.claude/agents/*.md` (`verifier` already exists, see `30-delegation-templates.md`).

| Task | model | Rationale |
|------|-------|------|
| Mechanical search, file location, format conversion | `haiku` | Cheap and fast; mistakes are easy to spot |
| General implementation, research, batch edits | `sonnet` | The default; good enough |
| Architecture trade-offs, high-risk judgment, adversarial review | `opus` (or the strongest model available) | Judgment quality decides the outcome |
| Questions about Claude Code / the Claude API | `claude-code-guide` agent | It checks official docs; don't answer from memory |

Note: the available agent types and model values are whatever the current session's system-reminder lists — commonly `claude` / `general-purpose` / `Explore` / `Plan` / `claude-code-guide` / `fork`, but the list changes across versions; never specify a value not on the list.

## Escalation / de-escalation paths

- **Escalate**: haiku fails the same task twice → re-dispatch with sonnet (don't let haiku try a 3rd time). Sonnet produces output you can't judge, or two edits go in opposite directions → opus, or ask the user. Full escalation criteria: `20-judgment.md`.
- **De-escalate**: for batch same-shape tasks, run the first item with sonnet to prove the flow and output format, then let haiku do the rest from the template.
- **Multi-answer judging**: for high-risk judgments that can't be verified by tests (e.g., architecture selection), dispatch 2–3 independent agents for separate answers, then have the main conversation (or opus) judge and pick. Expensive — only for decisions where the cost of being wrong is > 3× the judging cost.

## The producer's judgment is never acceptance evidence (iron rule)

Low-risk work may close on mechanical evidence — diff read-back, `rg` counts, test output — per the tiers in `20-judgment.md` §5. The producer's *opinion* ("looks fine") is never evidence at any tier. When a §5 trigger fires, acceptance leaves the producer entirely:

- **File deliverables** → dispatch a fresh agent for read-back: read the files, check each acceptance criterion
- **Code** → tests or a real run; if this machine can't run it (check `05-hosts.md`) → push and watch CI (`gh pr checks`)
- **High-risk judgments** → second opinion (an independent agent that hasn't seen the first answer; compare conclusions)
- The acceptance agent must have **fresh context** (a newly spawned `claude` or `verifier`), **never `fork`** — a fork inherits the main conversation's context, along with its biases and blind spots
- Give the acceptance prompt an "acceptance criteria list," not a leading framing like "I think it's done, please confirm"

## Relation to the harness's built-in guidance

The harness warns "don't spawn agents casually (expensive)" — that guards against dispatching for trivia. This file's thresholds (10 files / 2000 lines / full-repo scan / §5 acceptance triggers) are precisely the "worth dispatching" line; there's no conflict: below the threshold do it yourself, above it dispatch.
