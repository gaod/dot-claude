# CLAUDE.md (global)

## Working system (skills — loaded on demand)

The working system for this environment lives in `~/.claude/skills/`. Skills auto-trigger by their descriptions; if one doesn't fire when it should, invoke it explicitly. The moments that matter:

| Moment | Skill |
|------|--------|
| Starting any multi-step task: probe/look up the machine, write acceptance criteria, pick the verification tier, create task state | `task-kickoff` |
| Dispatching a subagent: choosing type/model/effort, writing the delegation prompt — or the current approach keeps failing | `delegate-work` |
| Before claiming anything is done, closing a task, or accepting a subagent's report | `verify-deliverable` |
| Creating or modifying anything under `~/.claude`; logging a pitfall | `maintain-claude-config` |

Data files (not auto-loaded; read and written via the skills): `~/.claude/hosts.md` (per-machine facts — machine not listed → `task-kickoff` probes and adds it) and `~/.claude/lessons.md` (pitfall log, append-only).

The three iron rules live in `rules/safety-and-quality.md` — auto-loaded every session alongside this file, always in effect.

## Shell tools

Prefer dedicated tools over traditional commands: find files with `fd` (`fdfind` on Debian/Ubuntu; not find/ls -R) | search text with `rg` (not grep) | code structure with `ast-grep` | interactive selection with `fzf` | JSON with `jq` | YAML/XML with `yq`.
If one is missing, you may install **lightweight CLI utilities from this list only**, and only via package managers that run without elevated privileges (brew / scoop / winget) — never compilers, SDKs, or heavy toolchains (see the `task-kickoff` skill, Risk 1). Managers that need sudo/root (apt / pkg): don't run them — report the missing tool, provide the exact install command for the user, and use built-in alternatives meanwhile. If installation fails, fall back to built-ins and note it in your report. Per-machine availability: `~/.claude/hosts.md`.
