---
name: lfp
description: >-
  Structured planning conductor for coding agents — goal lock, parallel research,
  option decision interviews, draft plan via harness Plan feature, scope interview,
  LOCKED plan with LFG handover. Use when the user says "lfp", "LFP", "let's fucking plan".
  Pairs with the lfg skill. Not for trivial fixes or greenfield product validation.
disable-model-invocation: true
---

# LFP — Let's Fucking Plan

Turn a brief into a **backlog-ready plan** any agent can execute via `/lfg` — without chat context.

**Universal protocol** — [`references/lfp-prompt.md`](references/lfp-prompt.md). **Conductor loop** — [`references/conductor-loop.md`](references/conductor-loop.md). **Cursor** — [`references/adapters/cursor.md`](references/adapters/cursor.md). **Sibling skill** — LFG at repo root ([`../SKILL.md`](../SKILL.md)).

## When to trigger

| Trigger | Action |
|---|---|
| User says **lfp**, **LFP**, **let's fucking plan** | Read this skill. Run planning conductor workflow. |
| User has a brief but no locked plan before LFG | LFP first, LFG second |
| User needs route decisions, scope add-ons, or LFG handover | LFP |

## When NOT to use LFP

| Situation | Use instead |
|---|---|
| Quick fix, typo, one-line change (≤15 files) | Direct execution |
| Locked plan already exists | `/lfg` directly |
| Brand-new product validation / 7 artifacts | new-product skill |
| Research-only, no build intent | Analysis mode |

**Output:** LOCKED plan at `docs/plans/` or `{project}/plans/` + §12 LFG Handover.

## Conductor workflow

```
GOAL LOCK → RESEARCH → DECIDE → DRAFT → SCOPE INTERVIEW → SOLIDIFY
```

| Phase | Action |
|---|---|
| 0 Goal lock | Brief → goal statement; CreatePlan / DRAFT file ([`harness-plan.md`](references/harness-plan.md)) |
| 1 Research | Parallel read-only subagents ([`research-prompts.md`](references/research-prompts.md)) |
| 2 Decide | Option matrix ([`option-matrix.md`](references/option-matrix.md)) + AskQuestion |
| 3 Draft | [`plan-template.md`](references/plan-template.md) into harness plan |
| 4 Scope | Add-ons ([`scope-addons.md`](references/scope-addons.md)), tier, testing bar — AskQuestion |
| 5 Solidify | LOCKED + project path + [`lfg-handover.md`](references/lfg-handover.md) |

Update harness plan file **after every phase**. Chat is not the source of truth.

## Division of labor

| Role | Who | Job |
|------|-----|-----|
| **Conductor** | Parent | Orchestrate, integrate, interview, write plan |
| **Researchers** | Subagents | Codebase, external, validator — read-only |
| **User** | Decision owner | Route, scope add-ons, lock |

## Research pre-auth

LFP invocation pre-authorizes **read-only:** codebase search, file reads, web search, `git log`/`git status`. No target-repo writes until LOCKED and user invokes `/lfg`.

## Single-route rule

Present 2+ routes when they exist. If only one is viable: say so explicitly — **never invent fake option B**.

## Companion skills (discover at runtime)

Scan `.cursor/skills/`, `AGENTS.md`, and available skills for project-specific add-ons. Do not hardcode org-specific skill names.

## Kickoff (when skill activates)

1. Read brief fully. Read project `AGENTS.md` if present.
2. Phase 0 goal lock + CreatePlan / DRAFT file.
3. TodoWrite checklist (steps 0–5 below).
4. Read and execute [`references/lfp-prompt.md`](references/lfp-prompt.md) with brief attached.
5. AskQuestion for interview rounds — batches of 2–4.

No fluff openings.

## LFP prompt

**Canonical one-shot:** [`references/lfp-prompt.md`](references/lfp-prompt.md).

## Stop criteria

| Condition | Action |
|---|---|
| LOCKED plan + §12 LFG Handover | **Done.** Deliver path + `/lfg` paste block |
| Blocked on user / access / new dep | **Pause.** One clear question |
| Trivial scope | **Redirect** — direct execution |
| Greenfield product | **Redirect** — new-product |

## TodoWrite checklist

```
0. Goal lock + harness plan file
1. Dispatch research subagents (parallel)
2. Integrate → option matrix → first interview
3. Draft plan (plan-template)
4. Scope interview
5. Solidify → LOCKED file → LFG handover deliverable
```

## Reference index

- [`references/lfp-prompt.md`](references/lfp-prompt.md) — **universal paste** at kickoff
- [`references/conductor-loop.md`](references/conductor-loop.md) — **planning loop spec**
- [`references/harness-plan.md`](references/harness-plan.md) — CreatePlan / plan file rules
- [`references/plan-template.md`](references/plan-template.md) — LOCKED artifact shape
- [`references/lfg-handover.md`](references/lfg-handover.md) — §12 required fields
- [`references/research-prompts.md`](references/research-prompts.md) — subagent dispatch copy
- [`references/option-matrix.md`](references/option-matrix.md) — Phase 2 format
- [`references/scope-addons.md`](references/scope-addons.md) — Phase 3/4 add-on menu
- [`references/adapters/README.md`](references/adapters/README.md) — Cursor · Claude Code · generic

## What this skill is NOT

- Not a 70% plan — scale research until one-shot executable
- Not LFG — no BUILD, no VERIFY, no target-repo edits
- Not new-product — scoped change planning, one plan file
- Not permission to commit — plan files only unless user asked
