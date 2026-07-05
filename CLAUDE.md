# CLAUDE.md (global)

## Working system (read the routing table first, load full files on demand)

`~/.claude/rules/` is the working system for this environment. Before starting any multi-step task, read the matching file:

| Situation | Read this |
|------|--------|
| Before starting work: confirm what this machine can run (toolchain, verification path) | `rules/05-hosts.md` (machine not listed → probe per its checklist and add a section yourself) |
| Understand the environment's structural risks and their fixes | `rules/00-diagnosis.md` |
| Dispatching subagents, choosing models, verifying deliverables | `rules/10-dispatch.md` |
| Judgment calls: escalate model / is it done / ask the user / change approach | `rules/20-judgment.md` |
| Writing a delegation prompt | `rules/30-delegation-templates.md` (use the `verifier` agent for acceptance) |
| Modifying the rules files or CLAUDE.md | `rules/40-maintenance.md` (read first — it has ownership rules) |
| Hit a pitfall | Add one line to `rules/50-lessons.md` |

Three iron rules, always in effect:
1. **Never claim completion without evidence (test output / CI link / read-back).** Grade every report: verified / pending CI / unverified
2. **External actions (messages, email, issue tracker, merging PRs, pushing shared branches) require explicit authorization in the current session**
3. **Never self-verify**: acceptance goes to a fresh-context agent; `fork` is only for execution tasks that need the full conversation context — never for acceptance (see `rules/10-dispatch.md`)

## Shell tools

Use dedicated tools instead of traditional commands: find files with `fd` (named `fdfind` on Debian/Ubuntu; don't use find/ls -R) | search text with `rg` (don't use grep) | analyze code structure with `ast-grep` | interactive selection with `fzf` | JSON with `jq` | YAML/XML with `yq`.
Install what's missing (brew on macOS, winget/scoop on Windows, apt on Linux, pkg on FreeBSD); fall back to built-in commands only if installation fails, and note it in your report. See `rules/05-hosts.md` for what's actually available per machine.
