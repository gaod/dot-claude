# CLAUDE.md (global)

## Working system (routing table — load full files on demand)

`~/.claude/rules/` is the working system for this environment. Before starting any multi-step task, read the matching file:

| Situation | Read this |
|------|--------|
| Before starting work: what this machine can run (toolchain, verification path) | `rules/05-hosts.md` (machine not listed → probe per its checklist and add a section yourself) |
| Understand the environment's structural risks and their fixes | `rules/00-diagnosis.md` |
| Dispatching subagents, choosing models, verifying deliverables | `rules/10-dispatch.md` |
| Judgment calls: escalate / is it done / ask the user / change approach / verification tier | `rules/20-judgment.md` |
| Writing a delegation prompt | `rules/30-delegation-templates.md` (use the `verifier` agent for acceptance) |
| Modifying the rules files or CLAUDE.md | `rules/40-maintenance.md` (read first — it has ownership rules) |
| Hit a pitfall | Add one line to `rules/50-lessons.md` |

Three iron rules, always in effect:

1. **Never claim completion without evidence** (test output / CI link / read-back). Grade every report: verified / pending CI / unverified.
2. **External actions require explicit authorization in the current session** — messages, email, issue tracker, merging PRs, pushing shared branches. This applies regardless of verification tier.
3. **The producer's judgment is never acceptance evidence.** Low-risk work may close on mechanical evidence (diff read-back, `rg` counts, test output); anything matching a verifier trigger in `rules/20-judgment.md` §5 goes to a fresh-context agent. `fork` inherits this conversation's biases — never use it for acceptance.

## Shell tools

Prefer dedicated tools over traditional commands: find files with `fd` (`fdfind` on Debian/Ubuntu; not find/ls -R) | search text with `rg` (not grep) | code structure with `ast-grep` | interactive selection with `fzf` | JSON with `jq` | YAML/XML with `yq`.
If one is missing, you may install **lightweight CLI utilities from this list only** (brew / apt / winget / scoop / pkg) — never compilers, SDKs, or heavy toolchains (see `rules/00-diagnosis.md` Risk 1). If installation fails, fall back to built-ins and note it in your report. Per-machine availability: `rules/05-hosts.md`.
