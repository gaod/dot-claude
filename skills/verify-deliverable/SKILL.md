---
name: verify-deliverable
description: Run before claiming any deliverable is done, closing a task, or accepting a subagent's report — checks the Definition of Done, selects the verification tier (mechanical self-acceptance vs fresh-context verifier), and applies per-type minimum verification. Trigger whenever about to say "done", "complete", "finished", or report results to the user.
---

# Verify Deliverable — Definition of Done, verification tiers, per-type minimums

> Purpose: turn experience-dependent judgment into mechanically executable criteria. Rubrics carry a positive example (✅ do this) and a negative one (❌ not this).
> Applies to every model tier. When a rubric conflicts with your intuition, follow the rubric first, then log the disagreement in `~/.claude/lessons.md`.
> The tech stacks in the examples are illustrative — substitute your project's equivalents.

## 1. When it actually counts as done (Definition of Done)

Check every item before claiming completion; one missing item means "not done," and the report must say which item is missing:

- [ ] The deliverable exists and is complete (files: read back; code: re-read your own diff)
- [ ] Acceptance criteria checked one by one (they should have been written at kickoff; if not, write them now, then check)
- [ ] Verification matches the tier (see §2): mechanical evidence for low-risk work; fresh-context verifier when a trigger fires. **Projects this machine can't run (check `~/.claude/hosts.md` first) may only claim "locally verified" — or "pending CI" if pushing was authorized this session**
- [ ] Tests and docs updated along with the change (changed behavior without touching tests = not done)
- [ ] No "should," "probably," or "in theory" qualifying the core conclusions of the report

✅ Positive: "Changed 3 files (paths listed), added 2 test cases, pushed to PR #42, CI green: <link>."
❌ Negative: "Changes complete, tests should pass." — "should pass" means unverified; that sentence equals "not done."

## 2. Verification tiers (check triggers, not vibes)

**Self-acceptance on mechanical evidence** is allowed only when ALL hold:

- ≤2 files and ≤30 changed lines
- No behavior change, and the files aren't referenced by other rules/skills/routing files
- No external side effects or persistent state changes (nothing left the repo: no pushes, no messages, no config or data writes)
- Every changed hunk is attributable to this task, so reverting it cannot touch pre-existing user work

Evidence must be mechanical — diff read-back, `rg` counts, test/lint output. The producer's *opinion* ("looks fine," "should be right") is never evidence; the producer's *mechanical output* is.

**Fresh-context verifier is REQUIRED** if ANY trigger fires:

- New rules/skill/agent/routing file, or edits to a file other files route to
- Doc/rules changes >30 lines or ≥3 files
- Security-sensitive change, data migration, release artifact, cross-module refactor
- The producer cannot name a mechanical check that would catch its own most likely error

Authorization is a separate axis: irreversible or externally visible actions follow iron rule 2 (ask the user) regardless of tier — a verifier PASS is not authorization.

✅ Positive: fixed two typos in a README (1 file, 4 lines, referenced by nothing) — re-read the diff, done. No verifier needed.
❌ Negative: rewrote the `delegate-work` skill's model table and self-approved it because "it's just docs" — that file is routed to by CLAUDE.md and read by every future session; trigger 1 fires, send it to `verifier`.

## 3. Per-type minimums (apply on top of the tier)

| Deliverable | Minimum verification |
|------|----------|
| Docs / rules / skill files | Per the tiers above; when a trigger fires, fresh-agent read-back judging each acceptance criterion (use the `verifier` agent; template 5 in the `delegate-work` skill's `templates.md`) |
| Code (repo this machine can run locally) | Run the module's tests + lint, paste the output |
| Code (repo this machine can't run, see `~/.claude/hosts.md`) | Static consistency (`rg` against existing conventions) + push and watch `gh pr checks` — pushing requires session authorization (iron rule 2); without it, stop at "locally verified" and provide the CI-trigger commands |
| Batch file edits | Sample read-back ≥ 3 files + `rg` to confirm no stragglers (count check: expected N changes, actual N) |
| Research conclusions | Every key fact cites a source; no source → mark it "speculation" |

✅ Positive: after batch-editing 20 files, `rg -l 'old_pattern'` should return 0 and `rg -l 'new_pattern' | wc -l` should return 20 — paste both numbers into the report.
❌ Negative: "all 20 files updated" with no counts as evidence.

## 4. Limits of the system (honesty clause)

The following **cannot be fixed by decomposition and verification** — don't pretend the system solves them:

- **Taste judgments** (UX copy tone, API naming aesthetics, long-term architecture trade-offs): the system can only guarantee "doesn't violate written rules," not "good." Approach: (a) copy the repo's existing conventions; (b) produce 2–3 candidates for the user to pick; (c) say plainly "this is a taste question — I'm giving a compliant answer, not necessarily the best one."
- **Vague requirements**: a task whose acceptance criteria can't be written down should first be turned into one whose criteria can (ask the user or run a small experiment) — don't just start building.
- **Unknown unknowns**: fresh-agent acceptance only verifies quality *within* the criteria; whatever the criteria missed, nobody catches. For high-risk tasks, add one open question to the criteria: "Beyond the above, what do you consider this deliverable's biggest risk?"
