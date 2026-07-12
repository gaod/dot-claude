# Lessons Log (append-only)

> Format: `- [YYYY-MM-DD][project or global] situation → lesson → applied to: {file or "not yet"}`
> Writing threshold and slimming rules: see the `maintain-claude-config` skill.
> The entries below are generic seed lessons (from the fieldwork that produced this system); if you fork this repo, consider clearing them and accumulating your own.

- [seed][global] Project spec required running tests locally, but the current machine lacked the toolchain → specs must state *which environment* verification runs in, or models fake completion or install tools at random → applied to: task-kickoff Risk 1
- [seed][global] Agent calls can't specify effort; effort only lives in the agent definition's frontmatter (current values: see delegate-work verified findings) → roles needing a specific effort need a ~/.claude/agents/ definition first → applied to: delegate-work
- [seed][global] Directory paths written in project docs didn't match the repo's actual structure (the docs were never checked against the repo) → `ls`-verify any path before citing it from a doc; docs go stale, the repo doesn't → applied to: verify-deliverable §3
- [seed][global] The system's first draft hard-coded one machine's limits as universal rules, but the user has multiple machines with different OSes → rules must be environment-neutral, machine facts isolated in hosts.md, probe first (uname/which) then branch → applied to: task-kickoff, hosts.md
- [2026-07-12][global] The system stored its working files in `~/.claude/rules/`, assuming on-demand loading — but Claude Code loads every rules file without `paths` frontmatter at session start, so the whole system was fixed context → before designing around a harness mechanism, verify its loading behavior in official docs → applied to: this restructure (rules → skills)
