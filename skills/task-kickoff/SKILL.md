---
name: task-kickoff
description: Run at the start of any multi-step task — look up or probe this machine's capabilities (~/.claude/hosts.md), write mechanically checkable acceptance criteria, pick the verification tier, and create a task-state file that survives compaction. Trigger before implementation work, refactors, research tasks, or anything with more than one step.
---

# Task Kickoff — probing, acceptance criteria, task state

> Single-machine facts always go in `~/.claude/hosts.md`, never in this file.
> When facts here go stale, update per the `maintain-claude-config` skill — don't create a new file.

## Kickoff checklist (run in order)

1. **Probe or look up the machine** (protocol below) — this determines what "verified" can mean on this machine
2. **Write acceptance criteria** — mechanically checkable ("`rg X` returns 0", "tests in module Y green"), not "looks good"
3. **Pick the verification tier** — see the `verify-deliverable` skill; decide *now* whether this work will need a fresh-context verifier, and write that into the criteria
4. **Create a task-state file** in the scratchpad (goal / acceptance criteria / done / todo) and update it after each step — after compaction, use it to recover state

## Environment probing protocol (every machine, before every multi-step task)

The user's environment may span multiple machines, and the same `~/.claude` system is synced to all of them. Therefore:

1. Probe identity: `hostname` + `uname -s` (Darwin=macOS; Linux; FreeBSD; MINGW*/MSYS*=Windows Git Bash)
2. Find this machine's section in `~/.claude/hosts.md` and reuse its facts (for stale-looking sections, spot-check one or two items)
3. No section found → run the probing checklist at the top of `hosts.md` and add the results as a new section (that file may be written to directly)
4. **Never assume a toolchain exists**: before running any command a spec or doc requires, verify with `which` (or `command -v`); if absent, check `hosts.md` for the alternative verification path — do not install heavy toolchains on your own (see Risk 1 below)

## The three structural risks and their fixes (hold across machines, by severity)

### 1. Verification gap: the verification a spec demands may not run on the current machine (the top source of "fake done")

**Symptom**: a project spec says "run the tests and make sure they pass," but the machine may lack the required toolchain (e.g., iOS projects can't compile on Linux/Windows/FreeBSD). Models then head toward three bad endings: (a) try to install the toolchain and fail after burning many tokens; (b) claim "tests pass" without running them (most dangerous); (c) get stuck.

**Fix**:
- Probe first per the protocol above; on machines that can't run it, do static verification only (`rg` / `ast-grep` against conventions) and route dynamic verification through CI — push branch → PR → `gh pr checks` until green. See `hosts.md` for each machine's concrete path.
- Iron rule: **never claim "tests pass" without test output or a CI run URL as evidence.** Always grade reports: verified (with evidence) / pending CI / unverified.
- Detailed criteria: see "Definition of Done" in the `verify-deliverable` skill.

### 2. Bloated fixed context: thousands of tokens leak at the start of every session

**Symptom**:
- A project CLAUDE.md stuffed with directory trees, templates, and tables (tens of KB is common) gets fully loaded every session.
- Files under `.claude/rules/` (project or user level) without `paths` frontmatter are loaded unconditionally at launch — a rules directory is not an on-demand store.
- Connected MCP connectors and plugins inject large numbers of tool names into every session — most irrelevant to the project at hand.

**Fix**:
- Keep project CLAUDE.md and any `rules/` files to "hard rules only"; multi-step procedures belong in skills (which load on demand), and reference data belongs in plain files outside `rules/` read via file tools.
- [Requires user action] Disconnect unused connectors; CLI plugins can be disabled per project in `.claude/settings.json` (see the verified findings at the bottom for syntax).
- Note to models: deferred MCP tools inject only their names — don't proactively ToolSearch schemas unrelated to the task.

### 3. The main thread doing grunt work + compaction amnesia: quality collapses mid-way through long tasks

**Symptom**: the main conversation does heavy file reading, repo scanning, and build-log reading itself; context balloons; after compaction triggers, early decisions are lost and the model starts redoing finished work or drifting from the original goal. This is structural, machine-independent.

**Fix**:
- Heavy reads (>10 files or >2000 lines), full-repo scans, web research, batch file edits, acceptance — dispatch a subagent; the main conversation takes conclusions only. See the `delegate-work` skill.
- At the start of a multi-step task, create a task-state file in the scratchpad (goal / acceptance criteria / done / todo) and update it after each step. After compaction, use it to recover state.
- Write facts worth keeping across sessions into the memory mechanism (if the harness provides one). Note that memory is local to each machine — machine-bound facts must say which machine they apply to.

## Verified findings (checked against official docs 2026-07; re-verify after version changes)

- **`.claude/rules/` loading behavior**: rules files without `paths` frontmatter load unconditionally at session start; user-level `~/.claude/rules/` applies to every project. Path-scoped rules (with `paths` globs) load when Claude works with matching files. Task-specific procedures belong in skills instead.
  (Source: code.claude.com/docs/en/memory.md)
- **Disabling a plugin per project**: in the project's `.claude/settings.json`, write
  `"enabledPlugins": { "<plugin>@<marketplace>": false }` to override the global setting.
- **`effortLevel`** (settings.json): values `low` / `medium` / `high` / `xhigh`; project level overrides global.
- **claude.ai-bound MCP connectors** (tool prefix `mcp__claude_ai_*`): **no official mechanism found** for project-level disabling — they can only be disconnected on the claude.ai side. When this matters, confirm with the user first; don't guess at settings.
