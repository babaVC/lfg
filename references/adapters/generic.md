# Generic adapter (any agent)

Use when your host has no subagents, no skills system, or you want maximum portability.

## Install

1. Copy `references/lfg-prompt.md` (execution) and `lfp/references/lfp-prompt.md` (planning) into your project
2. Add to agent instructions: *"Plan: LFP protocol in `docs/lfp-prompt.md`. Execute: LFG protocol in `docs/lfg-prompt.md`."*

Works with: ChatGPT, Gemini CLI, OpenCode, Windsurf, Copilot Workspace, custom scripts, or any agent that accepts a system/user prompt.

## Trigger

User says **lfg** / **LFG** / **let's fucking go** after plan approval.

## Workflow

**Phase 0:** Plan review — discover tier M/L, e2e harness (optional), smoke path, vertical slices.

**BUILD:** dispatch builder with [`builder-prompt.md`](../builder-prompt.md). Conductor does not implement.

**VERIFY:** automated + invariant grep + e2e if discovered + smoke. No repo harness required upfront — add during session when plan needs it.

**REVIEW:** scope-based dispatch per [`conductor-loop.md`](../conductor-loop.md). Separate BUILD context from REVIEW context.

**Loop:** tier floors M=2, L=3 — exit early when exit criteria met.

**DOGFOOD:** conductor runs or documents `RUNTIME UNVERIFIED`.

## Wave dispatch

**Mode:** `{DISPATCH_MODE}` = `sequential-role-switch` or `multi-session`

**Strict rule:** builder session ≠ reviewer session. If you only have one chat:

1. Conductor pastes [`builder-prompt.md`](../builder-prompt.md) → BUILD session
2. Conductor runs VERIFY (auto + invariant + smoke)
3. Loop per [`conductor-loop.md`](../conductor-loop.md) — paste reviewer prompts into **fresh sessions**
4. FIX in separate session from reviewers

Better: run each reviewer in a **fresh session** with plan + diff + test output + `{INVARIANT_CHECKS}` — zero implementation context pollution.

**Model:** `{REVIEWER_MODEL}` → any capable model; use a cheaper model for review if available.

No subagents required. The protocol still works — separation of roles matters more than parallelism.

## Parallelism without subagents

- Shell script that calls your agent CLI N× with different reviewer prompts (scope-based N)
- Human pastes prompts into multiple browser tabs
- CI job per reviewer lens on the PR diff

## Optional §5–§7

Run when triggered — same sequential discipline.
