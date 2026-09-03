# Cursor adapter — LFP

## Install

```bash
npx skills add babaVC/lfg
# or:
git clone https://github.com/babaVC/lfg.git ~/.cursor/skills/lfg
ln -sf ~/.cursor/skills/lfg/lfp ~/.cursor/skills/lfp
```

## Trigger

```
/lfp
```

Or say **lfp** / **LFP** / **let's fucking plan**.

## Conductor + research subagents

**Conductor = parent agent.** Does not edit target repo during LFP. Orchestrates loop in [`conductor-loop.md`](../conductor-loop.md).

### Phase 0 — GOAL LOCK

- Extract goal from brief; read `AGENTS.md`
- **CreatePlan** with phases 0–5 todos ([`harness-plan.md`](../harness-plan.md))
- Update plan file after each phase

### Phase 1 — RESEARCH (parallel Task calls)

One parent turn, multiple dispatches when independent:

```
Task: description="LFP codebase research"
      subagent_type="explore"
      model="inherit"
      prompt=[research-prompts.md CODEBASE block filled]

Task: description="LFP external research"  # when needed
      subagent_type="generalPurpose"
      model="inherit"
      prompt=[research-prompts.md EXTERNAL block filled]
```

| Wave | subagent_type | When |
|------|---------------|------|
| CODEBASE | `explore` | Code changes |
| EXTERNAL | `generalPurpose` | New APIs, unfamiliar domain |
| VALIDATOR | `generalPurpose` | Brief proposes solution |
| DEPENDENCIES | `explore` | Auth, billing, MCP, migrations |

Wait for returns. Integrate before Phase 2.

### Phase 2 — DECIDE

**AskQuestion** — 2–4 questions per batch. Lock route in plan §2.

### Phase 3–4 — DRAFT + SCOPE

Conductor writes plan. **AskQuestion** for add-ons, tier, blockers.

### Phase 5 — SOLIDIFY

Write LOCKED to `docs/plans/` or `{project}/plans/`. Deliver `/lfg` paste block.

## Pre-auth

LFP pre-authorizes read-only: Grep, Read, WebSearch, WebFetch, `git log`/`git status`. No target-repo writes until LOCKED.

## Model defaults

| Role | Model |
|---|---|
| Conductor | Parent picker |
| Researchers | `inherit` |

## After LFP

User attaches plan path and invokes **/lfg** ([`../../../SKILL.md`](../../../SKILL.md)).
