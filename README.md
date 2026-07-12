# claude-config — a rules system for long-term reliable AI coding agents

A working system for Claude Code that drops straight into `~/.claude/`: dispatch rules, judgment rubrics, delegation templates, a maintenance protocol, and an "automatic environment probing" mechanism. The goal: consistent, verifiable work quality from any model tier (Haiku / Sonnet / Opus) in your environment.

Design background: built in one pass by a stronger model, for long-term use by all subsequent model tiers. The principle: "write rules for the weakest executor — concrete, executable, with criteria and positive/negative examples."

## Installation

```bash
# Back up your existing setup
cp -r ~/.claude ~/.claude.backup-$(date +%F) 2>/dev/null

# Drop in the system files (won't overwrite other runtime files under ~/.claude)
cp CLAUDE.md ~/.claude/CLAUDE.md      # if you already have a global CLAUDE.md, merge in the routing table and iron-rules sections
cp -r rules agents ~/.claude/
```

After installing, **no environment info needs to be filled in manually**: `rules/05-hosts.md` ships empty, carrying only a probing checklist. The first time you run a multi-step task in Claude Code on each machine, the AI probes automatically per the protocol in `rules/00-diagnosis.md` (hostname, OS, toolchain, CLI tools, CI channel) and writes the results as that machine's section. Later sessions just look it up.

## File structure

| File | Purpose |
|------|------|
| `CLAUDE.md` | Routing table + three iron rules, auto-loaded every session (goes in `~/.claude/`) |
| `rules/00-diagnosis.md` | Environment probing protocol + three structural risks (fake completion / bloated context / compaction amnesia) and their fixes |
| `rules/05-hosts.md` | Per-machine facts (ships empty; the AI fills it in automatically) |
| `rules/10-dispatch.md` | Dispatch rules: when to use subagents, the delegation trio, report contract, model/effort selection, acceptance rules |
| `rules/20-judgment.md` | Judgment rubrics: when to escalate / what counts as done / when to ask the user / when to change course / verification tiers and per-type minimums — each with positive & negative examples |
| `rules/30-delegation-templates.md` | Five delegation prompt templates (search / implementation / refactoring / research / review) |
| `rules/40-maintenance.md` | Ownership and safe-modification process for the system files |
| `rules/50-lessons.md` | Pitfall log (append-only, with format and threshold) |
| `agents/verifier.md` | Subagent definition for the fresh-context acceptance reviewer |

## Core ideas (the three iron rules)

The canonical wording lives in `CLAUDE.md` (auto-loaded every session); in brief:

1. **Never claim completion without evidence** — grade reports: verified (with test output / CI link) / pending CI / unverified
2. **External actions require explicit authorization in the current session** — messages, email, merging PRs, pushing shared branches; applies regardless of verification tier
3. **The producer's judgment is never acceptance evidence** — low-risk work closes on mechanical evidence (diff read-back, `rg` counts, test output); anything hitting a verifier trigger (`rules/20-judgment.md` §5) goes to a fresh-context subagent, never a context-inheriting fork

## Known decay modes and prevention (maintainers, read this)

1. **Ritualization**: templates copied but acceptance criteria written as empty phrases → criteria must be mechanically checkable; verifier FAILs vague criteria on sight
2. **Bloat**: every pitfall stuffed into CLAUDE.md → lessons go only to `50-lessons.md`; promotion to formal rubric goes through the `40-maintenance.md` process; line-count thresholds trigger slimming
3. **Broken routing**: files renamed, routing points at nonexistent paths → `rg`-scan references before renames; broken links are P0
4. **Verification decaying into self-review**: catching yourself writing "looks fine" is the signal. The tier system has its own decay mode: the low-risk exemption creeping upward ("this 80-line rewrite *feels* small") — the §5 conditions are numeric precisely so this creep is detectable
5. **Bootstrap relies on discipline alone**: the system's only load point is the CLAUDE.md routing, with no enforcement mechanism → the iron rules are embedded directly in the auto-loaded CLAUDE.md; for a stronger guarantee, add your own SessionStart hook

## Honesty clause: what this system cannot fix

Decomposition, templates, and fresh-context acceptance raise **execution quality**; but **taste and vague questions** (long-term architecture trade-offs, copy tone, whether a feature should exist) can't be fixed by it. When you hit one, in order: copy the repo's existing conventions → use the strongest model available → produce multiple candidates for the user to choose → say plainly "this exceeds what the system can guarantee."

## License / usage

Take and modify freely. If you fork, consider clearing `50-lessons.md` and accumulating your own — lessons are only valid in the environment where they were learned.
