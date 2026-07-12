# Per-Host Facts

> **This is a template.** The live copy is `~/.claude/hosts.md` — gitignored, because it accumulates hostnames, home/project paths, and auth status that must never land in a public repo.
> The user's environment may span multiple machines. The skills are always environment-neutral; every "what can this machine do" fact lives only in the live file.
> Probing checklist and section template: see the `task-kickoff` skill. One section per machine; every fact must carry the date it was verified. Update stale facts on sight (the live file may be modified directly, see the `maintain-claude-config` skill).
>
> **Ships with no machine sections — that is deliberate.**
> AI: when you're working on a machine with no section in the live file, run the `task-kickoff` probing checklist and add the results as a new section.

## Platform notes (platform-level facts; machine-level facts go in the sections below)

- Installing missing CLI tools: brew (macOS) | apt (Debian/Ubuntu) | winget/scoop (Windows) | **pkg** (FreeBSD — prefer `pkg install` over building from ports)
- FreeBSD package naming (verified against the ports tree 2026-07-05): the `fd` binary comes from package `fd-find` (port `sysutils/fd`); `rg` from `ripgrep`; ast-grep is `textproc/ast-grep`
- FreeBSD base system ships no bash (default shells are sh/tcsh) — run `pkg install bash` if needed, and don't assume bash-isms when writing probe commands

---

(Machine sections below are added automatically by the AI on first work session per machine)
