# claude-config — a rules system for long-term reliable AI coding agents

A working system for Claude Code that drops straight into `~/.claude/`: dispatch rules, judgment rubrics, delegation templates, a maintenance protocol, and an "automatic environment probing" mechanism. The goal: consistent, verifiable work quality from any model tier (Haiku / Sonnet / Opus) in your environment.

Design background: built in one pass by a stronger model, for long-term use by all subsequent model tiers. The principle: "write rules for the weakest executor — concrete, executable, with criteria and positive/negative examples."

## Why Skills (architecture note)

Claude Code loads every file under `.claude/rules/` **without `paths` frontmatter unconditionally at session start** — a rules directory is not an on-demand store ([memory docs](https://code.claude.com/docs/en/memory)). An earlier version of this system kept ~450 lines of procedures in `rules/` behind a routing table, which meant the whole system was fixed context in every session — exactly the failure mode it warned against.

This version uses the harness's native primitives:

- **Always loaded (small, by design)**: `CLAUDE.md` (skill map + shell tools) and `rules/safety-and-quality.md` (the three iron rules + ask-the-user triggers) — together well under 60 lines.
- **Loaded on demand**: four [Skills](https://code.claude.com/docs/en/skills) that auto-trigger by description at the moments that matter.
- **Never auto-loaded (data)**: `hosts.md` and `lessons.md`, read and written by the skills through file tools.

## Installation

```bash
# Back up your existing setup
cp -r ~/.claude ~/.claude.backup-$(date +%F) 2>/dev/null

# Drop in the system files (won't overwrite other runtime files under ~/.claude)
cp CLAUDE.md ~/.claude/CLAUDE.md      # if you already have a global CLAUDE.md, merge in the skill map and shell-tools sections
cp -r rules skills agents ~/.claude/
cp hosts.md lessons.md ~/.claude/
```

After installing, **no environment info needs to be filled in manually**: `hosts.md` ships empty, carrying only a probing checklist. The first time you run a multi-step task in Claude Code on each machine, the AI probes automatically per the protocol in the `task-kickoff` skill (hostname, OS, toolchain, CLI tools, CI channel) and writes the results as that machine's section. Later sessions just look it up.

## File structure

| File | Purpose | Loading |
|------|------|------|
| `CLAUDE.md` | Skill map + shell-tool rules (goes in `~/.claude/`) | Every session |
| `rules/safety-and-quality.md` | Three iron rules + when to stop and ask the user | Every session |
| `skills/task-kickoff/SKILL.md` | Environment probing protocol + three structural risks (fake completion / bloated context / compaction amnesia) + kickoff checklist | On demand |
| `skills/delegate-work/SKILL.md` | Dispatch rules: when to use subagents, the delegation trio, report contract, model/effort selection, escalation & change-course rubrics, acceptance rules | On demand |
| `skills/delegate-work/templates.md` | Five delegation prompt templates (search / implementation / refactoring / research / review) | On demand |
| `skills/verify-deliverable/SKILL.md` | Definition of Done, verification tiers and per-type minimums, honesty clause | On demand |
| `skills/maintain-claude-config/SKILL.md` | Ownership and safe-modification process for the system files | On demand |
| `hosts.md` | Per-machine facts (ships empty; the AI fills it in automatically) | Never (data) |
| `lessons.md` | Pitfall log (append-only, with format and threshold) | Never (data) |
| `agents/verifier.md` | Subagent definition for the fresh-context acceptance reviewer | On dispatch |

## Core ideas (the three iron rules)

The canonical wording lives in `rules/safety-and-quality.md` (auto-loaded every session); in brief:

1. **Never claim completion without evidence** — grade reports: verified / locally verified / pending CI / unverified
2. **External actions require explicit authorization in the current session** — messages, email, merging PRs, pushing shared branches; applies regardless of verification tier, and needing CI for verification does not by itself grant it
3. **The producer's judgment is never acceptance evidence** — low-risk work closes on mechanical evidence (diff read-back, `rg` counts, test output); anything hitting a verifier trigger (`verify-deliverable` skill) goes to a fresh-context subagent, never a context-inheriting fork

## Known decay modes and prevention (maintainers, read this)

1. **Ritualization**: templates copied but acceptance criteria written as empty phrases → criteria must be mechanically checkable; verifier FAILs vague criteria on sight
2. **Bloat**: every pitfall stuffed into CLAUDE.md → lessons go only to `lessons.md`; promotion to formal rubric goes through the `maintain-claude-config` process; line-count thresholds trigger slimming
3. **Broken routing**: files renamed, references point at nonexistent paths — and in this architecture, **a skill `description` that stops matching its trigger moments is the same defect** → `rg`-scan references before renames; keep descriptions accurate; broken links are P0
4. **Verification decaying into self-review**: catching yourself writing "looks fine" is the signal. The tier system has its own decay mode: the low-risk exemption creeping upward ("this 80-line rewrite *feels* small") — the tier conditions are numeric precisely so this creep is detectable
5. **Skills not firing**: the system's load points are CLAUDE.md, the safety rules, and skill descriptions; if a skill doesn't auto-trigger, the CLAUDE.md skill map is the fallback — invoke it explicitly. For a stronger guarantee, add your own SessionStart hook

## Honesty clause: what this system cannot fix

Decomposition, templates, and fresh-context acceptance raise **execution quality**; but **taste and vague questions** (long-term architecture trade-offs, copy tone, whether a feature should exist) can't be fixed by it. When you hit one, in order: copy the repo's existing conventions → use the strongest model available → produce multiple candidates for the user to choose → say plainly "this exceeds what the system can guarantee."

## License / usage

Take and modify freely. If you fork, consider clearing `lessons.md` and accumulating your own — lessons are only valid in the environment where they were learned.
