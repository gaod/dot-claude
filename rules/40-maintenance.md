# 40 — System Maintenance Protocol

> Audience: any future model maintaining `~/.claude/rules/`, `~/.claude/agents/`, or any project CLAUDE.md.

## File inventory and ownership

| File | Nature | May modify on your own? |
|------|------|-----------|
| `00-diagnosis.md` | Cross-machine conclusions & verified findings | ✅ Update stale facts (with test evidence); don't remove the "three structural risks" framework |
| `05-hosts.md` | Per-machine facts | ✅ New machine sections may be added freely; stale facts may be updated (with verification date) |
| `10-dispatch.md` / `20-judgment.md` / `30-delegation-templates.md` | System core | ⚠️ Adding entries is fine; **modifying or deleting existing rubrics requires asking the user first** |
| `40-maintenance.md` (this file) | Constitution | ❌ Ask the user before touching |
| `50-lessons.md` | Lessons log | ✅ Append anytime (append-only) |
| `~/.claude/agents/*.md` | Agent definitions | ✅ New roles may be added; changing an existing role's duties — ask first |
| `~/.claude/CLAUDE.md` | Global routing | ⚠️ Only add/fix routing lines; never stuff long content back in |
| Each project's `CLAUDE.md` | **Team-shared, checked in** | ❌ Content changes require asking the user first (teammates use it too); same for its referenced files |

## Standard file-change procedure

1. Back up: `cp <file> ~/.claude/backups/<name>.$(date +%F).bak` (create the backups directory if it doesn't exist)
2. Modify (prefer adding new sections over touching existing text)
3. Verify: dispatch the `verifier` agent for read-back; acceptance criteria must at least include "no contradictions with the other rules files" and "referenced paths/commands actually exist"
4. If the change is inside a project: don't commit it yourself — leave it for the user to review

## How lessons get written back (after every pitfall)

Write to `50-lessons.md`, one line per lesson, format:

```
- [YYYY-MM-DD][project or global] one-line situation → one-line lesson → applied to: {file or "not yet"}
```

Threshold for "worth writing": this pitfall would cost the next session ≥10 minutes, and it's not visible from the repo/git log.
Hitting the same pitfall twice = it should graduate from lessons into a formal rubric (propose a change to `20-judgment.md` via the "ask the user first" process).

## Slimming protocol (anti-bloat)

- Trigger: `50-lessons.md` exceeds 40 entries, or any rules file exceeds 300 lines
- Action: delete lessons already promoted to formal rubrics; archive stale facts to `~/.claude/backups/`; merge duplicates
- Slimming means "move and merge," not "rewrite" — rewriting an entire rules file requires the user's consent

## Routing integrity (anti-broken-links)

- When moving or renaming any referenced file, in the same change first run `rg -l '<old filename>'` across `~/.claude/` and every project's CLAUDE.md (project root locations: see `05-hosts.md`) to find all references. References inside rules/ files → update together; references in a project CLAUDE.md → ask the user first per the ownership rules, then change everything together
- Periodically (or on user request) run a health check: existence-check every path and command referenced by each rules file (dispatching verifier is enough)

## Applying across projects

This system is environment-level (shared across the user's machines and all repos; machine differences are isolated in `05-hosts.md`). When starting work in a new project:
1. First establish verifiability of "this repo on this machine" (can it build/test locally? — this determines how items 2 and 5 of `20-judgment.md` are grounded; the answer can differ for the same repo on different machines)
2. If the repo has its own CLAUDE.md, that's the project spec; this system governs "how to work," the project CLAUDE.md governs "what kind of code to write" — on conflict, the project file wins
3. Project facts worth carrying forward go into that project's memory directory (if the harness provides one), not into the global rules
