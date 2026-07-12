# 20 — Judgment Rubrics

> Purpose: turn experience-dependent judgment into mechanically executable criteria. Each rubric carries a positive example (✅ do this) and a negative one (❌ not this).
> Applies to every model tier. When a rubric conflicts with your intuition, follow the rubric first, then log the disagreement in `50-lessons.md`.
> The tech stacks in the examples are illustrative — substitute your project's equivalents.

## 1. When to escalate the model (or change approach)

Any one of these signals means escalate (haiku→sonnet→opus) or switch to multi-answer judging (see `10-dispatch.md`):

- The same bug is still there after 2 fixes
- You catch yourself *guessing* API behavior or framework semantics instead of verifying
- Two consecutive edits went in opposite directions (changed it, then changed it back)
- The task demands taste or trade-offs (naming, API design, UX copy) and there's no existing convention to copy

✅ Positive: fixing a UI framework state-sync bug, CI still red after the second attempt — stop, write up the symptoms, both attempts, and the relevant files into a prompt; dispatch opus or ask the user.
❌ Negative: tweaking the same parameter line a third time and pushing to CI "to see if it passes this time" — that's using CI as dice.

## 2. When it actually counts as done (Definition of Done)

Check every item before claiming completion; one missing item means "not done," and the report must say which item is missing:

- [ ] The deliverable exists and is complete (files: read back; code: re-read your own diff)
- [ ] Acceptance criteria checked one by one (they should have been written at kickoff; if not, write them now, then check)
- [ ] Verification matches the tier (see §5): mechanical evidence for low-risk work; fresh-context verifier when a trigger fires. **Projects this machine can't run (check `05-hosts.md` first) may only claim "static checks done, pending CI"**
- [ ] Tests and docs updated along with the change (changed behavior without touching tests = not done)
- [ ] No "should," "probably," or "in theory" qualifying the core conclusions of the report

✅ Positive: "Changed 3 files (paths listed), added 2 test cases, pushed to PR #42, CI green: <link>."
❌ Negative: "Changes complete, tests should pass." — "should pass" means unverified; that sentence equals "not done."

## 3. When to stop and ask the user

Self-check first: can the answer come from the repo, git log, docs, or one cheap experiment? If yes, don't ask. The following cases **require** asking:

- Irreversible or externally visible actions: force push, branch deletion, outbound messages (Slack / email / issue tracker, etc.), shared-config changes, releases (iron rule 2 — applies regardless of verification tier)
- The requirement is ambiguous and a wrong guess wastes 30+ minutes of work
- Two rules files conflict with no priority order to follow
- An existing file's content contradicts the user's description (present the contradiction; don't overwrite per the description)

✅ Positive: the task says "clean out the old config," but git log shows someone touched that config last week — stop and ask, attaching the finding.
❌ Negative: asking "4 spaces or tabs?" — the repo's formatter config / existing code already answers that.
❌ Negative (the reverse): merging a PR on your own because "CI is green" — merging is externally visible and affects teammates; without explicit authorization, don't.

## 4. Signals the direction is wrong (change course, don't retry)

On any of these signals: **stop → return to the last known-good state (git stash / checkout) → restate the problem → take a different path or escalate**:

- Fixing A broke B, fixing B broke C (repair chain ≥ 3 links)
- The diff touches more than 2× the files estimated at kickoff (no estimate written? use the files/modules the task description explicitly names as the baseline; touching modules the description never mentions counts as over)
- You start writing "work around the framework" hacks (swizzling, reflection, copying whole chunks of vendor code in to modify)
- You can't explain *why* it works now — only "it works when I change it this way"

✅ Positive: realizing the test can only pass by mocking half the system — stop, suspect the test is cut at the wrong layer, go back to the project's testing guidelines, re-pick the entry point.
❌ Negative: tests keep failing, so you loosen the assertions until they're green — that's not a fix, that's destroying evidence.

## 5. Verification: tiers first, then per-type minimums

### Verification tiers (check triggers, not vibes)

**Self-acceptance on mechanical evidence** is allowed only when ALL hold:

- ≤2 files and ≤30 changed lines
- No behavior change, and the files aren't referenced by other rules/routing files
- Fully reversible (`git revert` suffices)

Evidence must be mechanical — diff read-back, `rg` counts, test/lint output. The producer's *opinion* ("looks fine," "should be right") is never evidence; the producer's *mechanical output* is.

**Fresh-context verifier is REQUIRED** if ANY trigger fires:

- New rules/agent/routing file, or edits to a file other files route to
- Doc/rules changes >30 lines or ≥3 files
- Security-sensitive change, data migration, release artifact, cross-module refactor
- The producer cannot name a mechanical check that would catch its own most likely error

Authorization is a separate axis: irreversible or externally visible actions follow iron rule 2 (ask the user) regardless of tier — a verifier PASS is not authorization.

✅ Positive: fixed two typos in a README (1 file, 4 lines, referenced by nothing) — re-read the diff, done. No verifier needed.
❌ Negative: rewrote `10-dispatch.md`'s model table and self-approved it because "it's just docs" — that file is routed to by CLAUDE.md and read by every future session; trigger 1 fires, send it to `verifier`.

### Per-type minimums (apply on top of the tier)

| Deliverable | Minimum verification |
|------|----------|
| Docs / rules files | Per the tiers above; when a trigger fires, fresh-agent read-back judging each acceptance criterion (use the `verifier` agent) |
| Code (repo this machine can run locally) | Run the module's tests + lint, paste the output |
| Code (repo this machine can't run, see `05-hosts.md`) | Static consistency (`rg` against existing conventions) + push and watch `gh pr checks` |
| Batch file edits | Sample read-back ≥ 3 files + `rg` to confirm no stragglers (count check: expected N changes, actual N) |
| Research conclusions | Every key fact cites a source; no source → mark it "speculation" |

✅ Positive: after batch-editing 20 files, `rg -l 'old_pattern'` should return 0 and `rg -l 'new_pattern' | wc -l` should return 20 — paste both numbers into the report.
❌ Negative: "all 20 files updated" with no counts as evidence.

## 6. Limits of the system (honesty clause)

The following **cannot be fixed by decomposition and verification** — don't pretend the system solves them:

- **Taste judgments** (UX copy tone, API naming aesthetics, long-term architecture trade-offs): the system can only guarantee "doesn't violate written rules," not "good." Approach: (a) copy the repo's existing conventions; (b) produce 2–3 candidates for the user to pick; (c) say plainly "this is a taste question — I'm giving a compliant answer, not necessarily the best one."
- **Vague requirements**: a task whose acceptance criteria can't be written down should first be turned into one whose criteria can (ask the user or run a small experiment) — don't just start building.
- **Unknown unknowns**: fresh-agent acceptance only verifies quality *within* the criteria; whatever the criteria missed, nobody catches. For high-risk tasks, add one open question to the criteria: "Beyond the above, what do you consider this deliverable's biggest risk?"
