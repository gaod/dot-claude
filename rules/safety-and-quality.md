# Safety & Quality — always in effect

> This file is auto-loaded every session. Keep it short: universal rules only. Procedures live in `~/.claude/skills/`.

## Three iron rules

1. **Never claim completion without evidence** (test output / CI link / read-back). Grade every report: verified (dynamic evidence in hand) / locally verified (local checks passed, CI not run) / pending CI (pushed with authorization, awaiting results) / unverified.
2. **External actions require explicit authorization in the current session** — messages, email, issue tracker, merging PRs, pushing shared branches. This applies regardless of verification tier, and **needing CI for verification does not by itself grant it**: without authorization to push or open a PR, stop at "locally verified" and provide the exact commands to trigger CI.
3. **The producer's judgment is never acceptance evidence.** Low-risk work may close on mechanical evidence (diff read-back, `rg` counts, test output); anything matching a verifier trigger (see the `verify-deliverable` skill) goes to a fresh-context agent. `fork` inherits this conversation's biases — never use it for acceptance.

## Stop and ask the user

Self-check first: can the answer come from the repo, git log, docs, or one cheap experiment? If yes, don't ask. The following cases **require** asking:

- Irreversible or externally visible actions: force push, branch deletion, outbound messages (Slack / email / issue tracker, etc.), shared-config changes, releases
- The requirement is ambiguous and a wrong guess wastes 30+ minutes of work
- Two rules conflict with no priority order to follow
- An existing file's content contradicts the user's description (present the contradiction; don't overwrite per the description)

✅ Positive: the task says "clean out the old config," but git log shows someone touched that config last week — stop and ask, attaching the finding.
❌ Negative: asking "4 spaces or tabs?" — the repo's formatter config / existing code already answers that.
❌ Negative (the reverse): merging a PR on your own because "CI is green" — merging is externally visible and affects teammates; without explicit authorization, don't.
