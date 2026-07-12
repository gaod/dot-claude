# Delegation Prompt Templates

> Usage: copy the matching template, fill in the `{...}` blanks, and use it as the Agent tool's `prompt`.
> Every template has the trio built in (goal & motivation / acceptance criteria / report format). Don't delete those sections — vague acceptance criteria are the #1 path to system decay.
> Model selection principles: see `SKILL.md` in this directory.

## 1. Search / locate (subagent_type: `Explore`, no model needed)

> Built-in `Explore` skips CLAUDE.md and inherits the main conversation's model (see SKILL.md verified findings). Restate in the prompt any project convention the search must honor — it won't see the project rules.

```
In {repo path}, find {target: definition and all usages of a symbol / all files matching a pattern}.
Motivation: {why we're looking — so you can judge whether edge cases count}
Search breadth: {quick / medium / very thorough}
Acceptance criteria:
- Every result includes file_path:line
- State which patterns and directories you searched (so I can judge whether anything was missed)
Report format: result list (path:line + one-line note), and a final line "Search scope: ...". If nothing is found, say so explicitly.
```

## 2. Implementation (subagent_type: `general-purpose`, model: `sonnet`)

```
Task: {what to do}
Motivation: {why, upstream context}
Conventions: first read {project CLAUDE.md path} and {relevant referenced files}, and follow the existing style of {reference implementation file path}.
Scope: only modify {directory/file list}; touch nothing else.
Acceptance criteria:
- {concrete checkable criterion 1}
- {criterion 2, e.g.: matching test file exists and covers the main state transitions and the error case}
- {verification action, e.g.: run `{test command}` and paste output; if this machine can't run it, list the static checks you did}
Report format: list of changed files (path + one line), verification results, points you're unsure about. Don't paste full code.
```

## 3. Refactoring (subagent_type: `general-purpose`, model: `sonnet`; if >10 files affected, report the plan before acting)

```
Task: change {old pattern} to {new pattern} across {scope}.
Motivation: {why}
Invariants (the definition of refactoring — behavior must not change):
- {e.g.: all public API signatures unchanged / tests unmodified and all green}
Steps: first use `rg` to list every site needing change and report the count; {if >10 files: wait for my confirmation before acting / if ≤10 files: proceed}.
Acceptance criteria:
- `rg '{old pattern}'` returns 0 results
- Number of sites changed = number counted up front (report both numbers)
- {test command} all green (if this machine can't run it, say explicitly: locally verified — or pending CI only if pushing is authorized for this task)
Report format: counted vs. actually changed, verification output, any exceptions you judged should be skipped (with reasons).
```

## 4. Research (subagent_type: `general-purpose`, model: `sonnet`; for questions about Claude Code / the Claude API itself use `claude-code-guide`)

```
Question: {what to answer}
Motivation: {what decision the answer feeds — so you can judge how deep is deep enough}
Source priority: official docs > official repo/changelog > high-quality third party. {If a library-docs lookup tool is available (e.g. Context7 MCP), prefer it}
Acceptance criteria:
- Every key conclusion cites a source URL
- Separate "confirmed" from "speculation"; if it can't be found, write that — fabrication is forbidden
- {If comparing options: table the trade-offs of 2–4 options, give one recommendation with reasons}
Report format: conclusions (≤10 lines) + itemized facts with sources. Long reports go to a file at {scratchpad path}; return the path.
```

## 5. Review / acceptance (subagent_type: `verifier`; or `claude` + model: `opus` for high-risk adversarial review)

> Use this template when a verifier trigger from the `verify-deliverable` skill fires. Low-risk changes that pass its self-acceptance conditions don't need it — closing them on mechanical evidence is correct, not a shortcut.
> If the current session's available-agents list has no `verifier` (definition at `~/.claude/agents/verifier.md`; it won't appear when the harness hasn't loaded it), use `subagent_type: "claude"` and paste verifier.md's body rules at the top of the prompt.

```
You are a fresh-context acceptance reviewer conducting an adversarial review of the deliverable below. You didn't take part in making it — that's your advantage. Don't fill in the maker's intent for them.
Deliverable: {file path list}
Acceptance criteria (judge each PASS/FAIL; every FAIL gets file:line and a reason):
- {criterion 1}
- {criterion 2}
- Contradictory rules, wrong paths or commands, ambiguous wording likely to be misread (always keep this item)
Open question: beyond the criteria above, what is this deliverable's biggest risk?
Report format: PASS/FAIL per item + evidence, answer to the open question. Return only conclusions and locations; don't restate file contents.
```

## General notes (all templates)

- When delegating, write the current machine's environment limits into the prompt (check `~/.claude/hosts.md`, e.g.: this machine can't compile a given project type, verification goes through CI) — otherwise the subagent will crash into the same wall itself.
- After a subagent reports, the main conversation spot-checks at least one cheaply verifiable claim (one `rg`, one read) before trusting the report as a whole.
