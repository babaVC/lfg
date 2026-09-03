# LFP Conductor Loop

The conductor is the **parent agent**. Researchers are read-only subagents. The conductor **integrates, interviews, writes the plan** — never skips research synthesis when subagents are available.

```
GOAL LOCK → RESEARCH → DECIDE → DRAFT → SCOPE INTERVIEW → SOLIDIFY
```

---

## Roles

| Role | Who | May edit files? |
|---|---|---|
| **Conductor** | Parent | Plan files only during LFP — not target repo |
| **Researcher** | Subagent | **Never** — read-only |
| **User** | Decision owner | Route, scope, lock |

Researcher ≠ Conductor in the same research task.

---

## The loop

```
┌──────────────┐
│  GOAL LOCK   │  Phase 0 — brief → goal statement; CreatePlan / draft file
└──────┬───────┘
       ▼
┌──────────────┐
│  RESEARCH    │  Phase 1 — parallel subagents; integrate Option Brief
└──────┬───────┘
       ▼
┌──────────────┐
│  DECIDE      │  Phase 2 — option matrix + AskQuestion; lock route
└──────┬───────┘
       ▼
┌──────────────┐
│  DRAFT       │  Phase 3 — fill plan-template; present DRAFT
└──────┬───────┘
       ▼
┌──────────────┐
│  SCOPE       │  Phase 4 — add-ons, open Qs, tier, testing bar
└──────┬───────┘
       ▼
┌──────────────┐
│  SOLIDIFY    │  Phase 5 — LOCKED + project path + LFG handover
└──────────────┘
```

After every phase, **update harness plan file** ([`harness-plan.md`](harness-plan.md)).

---

## Wave catalog

| Phase | Subagent? | When |
|---|---|---|
| **GOAL LOCK** | No | Always first |
| **CODEBASE** | Yes — explore | Code changes |
| **EXTERNAL** | Yes — generalPurpose | New APIs, unfamiliar domain |
| **VALIDATOR** | Yes — generalPurpose | Brief proposes solution |
| **DEPENDENCIES** | Yes — explore | Auth, billing, MCP, migrations |
| **DECIDE** | No | AskQuestion — user picks route |
| **DRAFT** | No | Conductor writes plan |
| **SCOPE** | No | AskQuestion — add-ons, tier, blockers |
| **SOLIDIFY** | No | Write LOCKED to project path |

Launch independent research waves **in one parent turn** when possible.

---

## Research depth

**No fixed tiers.** Conductor scales until plan is one-shot executable:

- Trivial / single-file → CODEBASE only, skip EXTERNAL
- Ambiguous architecture → all applicable waves
- Single viable route → say so; never fake option B

---

## Stop criteria

| Condition | Action |
|---|---|
| LOCKED plan + §12 LFG Handover complete | **Done.** Deliver path + `/lfg` paste block |
| Blocked on access, architecture fork, new dep | **Pause.** One clear AskQuestion |
| Trivial scope (≤15 files, no decisions) | **Redirect** — direct execution, not LFP |
| Greenfield product validation | **Redirect** — new-product skill |
| User cancels / pivots | **Pause.** Save DRAFT state in plan file |

---

## Deliverable

1. Plan path (project `docs/plans/` or `plans/`)
2. One-paragraph summary
3. Paste-ready: attach plan + **/lfg**
4. Conductor log — phases run, subagents dispatched, decisions locked
