# 05 — Per-Host Facts

> The user's environment may span multiple machines. The rules files (00–40) are always environment-neutral;
> every "what can this machine do" fact lives only in this file. Usage: see the probing protocol in `00-diagnosis.md`.
> One section per machine; every fact must carry the date it was verified. Update stale facts on sight (this file may be modified directly, see `40-maintenance.md`).
>
> **This file ships with no machine sections — that is deliberate.**
> AI: when you're working on a machine with no section here, run the checklist below and add the results as a new section (follow the "Section template").

## Probing checklist (run on first work session on a new machine)

1. Identity: `hostname`, `uname -sm` (Darwin=macOS; Linux; FreeBSD; MINGW*/MSYS*=Windows Git Bash; in PowerShell use `$env:OS`)
2. Where the project root is; single-project or multi-project workstation
3. Toolchain inventory (list only what's relevant to the user's work): `which xcodebuild mise swift node python3 go php cargo` (on Windows use `where`)
4. CLI tools: `which rg fd fdfind ast-grep jq yq fzf gh`
5. `gh auth status` (can we track CI / open PRs?)
6. Special limitations: which kinds of projects **cannot run** on this machine, and what the alternative verification path is (the most important item — it directly determines how "done" in `20-judgment.md` is grounded)

## Platform notes (platform-level facts; machine-level facts still go in the sections below)

- Installing missing CLI tools: brew (macOS) | apt (Debian/Ubuntu) | winget/scoop (Windows) | **pkg** (FreeBSD — prefer `pkg install` over building from ports)
- FreeBSD package naming (verified against the ports tree 2026-07-05): the `fd` binary comes from package `fd-find` (port `sysutils/fd`); `rg` from `ripgrep`; ast-grep is `textproc/ast-grep`
- FreeBSD base system ships no bash (default shells are sh/tcsh) — run `pkg install bash` if needed, and don't assume bash-isms when writing probe commands

## Section template (copy and fill in)

```markdown
## <hostname> — <one-line description> (<OS arch>) | verified <YYYY-MM-DD>

- Project root: <path>; <single-project / multi-project workstation>
- Toolchain: <what's available; project types that can't run and their alternative verification paths>
- CLI: <rg/fd/jq/... availability and naming differences>
- gh: <logged in or not; can it track CI>
- Special limitations: <e.g., no Apple toolchain — iOS projects verify via CI (push → PR → gh pr checks)>
```

---

(Sections below are added automatically by the AI on first work session per machine)
