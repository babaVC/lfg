# LFG Conductor Playbook

**Primary spec:** [`conductor-loop.md`](conductor-loop.md) — evaluate → decide → dispatch → integrate.

This file is the detailed reference. The fixed diagram below is a **first-run example**, not the only path.

---

## First run (typical tier M)

```
Phase 0 plan review → BUILD (subagent) → VERIFY (auto + invariant + smoke)
  → REVIEW ×3 (structure, code, edge — scope-based)
  → FIX → VERIFY → REVIEW (targeted, scoped prompts)
  → … until exit criteria → GATE → dogfood → PROUD YES
```

Conductor chooses reviewers and loop count per tier and scope. **Tier floors:** M=2, L=3 — exit early when criteria met.

---

## Prerequisites

- [ ] Approved plan with acceptance criteria
- [ ] Scope in/out stated
- [ ] Project AGENTS.md read **if present** (no Constraints section required)
- [ ] Planned file list for re-review scoping
- [ ] Phase 0 discovers e2e harness — **not required to exist**

---

## Phase 0 — Plan review + boost

Before scope lock or BUILD:

- Discover e2e/Playwright harness ([`e2e-smoke.md`](e2e-smoke.md)) — run if found, don't require upfront
- Infer tier M or L
- Smoke path: use plan section, discovered e2e command, or infer manual steps
- Flag structural-only acceptance criteria
- Suggest vertical slices for multi-gate wizards (>50 files)

See [`lfg-prompt.md`](lfg-prompt.md) Phase 0.

---

## Wave 0 — Load

Scope lock with planned files list, tier, smoke path summary.

---

## BUILD — subagent in LFG

Conductor **dispatches** [`builder-prompt.md`](builder-prompt.md). Conductor does not implement.

Parent builds only if subagent infra unavailable — document degradation.

---

## VERIFY

After every BUILD or FIX:

1. Automated: build, lint, typecheck, test
2. Invariant grep ([`invariant-grep.md`](invariant-grep.md)) — generic always; persist new greps in-session
3. E2e/smoke ([`e2e-smoke.md`](e2e-smoke.md)) — run if discovered; manual if not

Inline fixes do not count as a review-fix loop.

---

## REVIEW — conductor selects lenses

**Loop 1:** per scope matrix — not always 4 core. Model: **`composer-2.5-fast`** on Cursor.

| Plan scope | Dispatch |
|------------|----------|
| Backend + migration | §1, §2, §4 |
| Wizard / app UI | §1, §2, §4 (§3a only if UI polish needed) |
| Marketing LP | §1–§4 + §6 |
| Auth/secrets | + §5 |

**Loop 2+:** targeted reviewers + [`reviewer-prompts.md` § Re-review scoped](reviewer-prompts.md#re-review-scoped-prompt).

Include `{REVIEWER_CONTEXT}` and `{INVARIANT_CHECKS}` in every dispatch.

Parallel when independent. See [`adapters/cursor.md`](adapters/cursor.md).

---

## FIX

Every FAIL. Fixer subagent or conductor inline (≤3 findings, MAJOR or below). Then full VERIFY → conductor EVALUATE → next REVIEW wave.

---

## Vertical slices

For multi-path features (scratch / GitHub / files):

- Phase 0: suggest slices A → B → C if plan is monolith
- Each slice: BUILD → VERIFY → REVIEW → GATE → dogfood → next
- Avoids single 60-file BUILD + L-tier review

---

## GATE & PROUD

[`gate-checklist.md`](gate-checklist.md) then dogfood then B1–B5.

**Dogfood:** conductor runs smoke + unhappy path + console check. If browser unavailable: **`RUNTIME UNVERIFIED`** in deliverable §9.

Any FAIL → back into loop.

---

## Reviewer failure

| Situation | Action |
|---|---|
| Task errors | Retry once → inline `REVIEWER_DEGRADED` pass |
| Missing VERDICT | FAIL; re-run |
| All reviewers fail to launch | STOP — escalate |

---

## Stop & escalate

**Done:** exit criteria + GATE PASS/N/A + dogfood PASS or RUNTIME UNVERIFIED + PROUD YES.

**Escalate:** thrashing · blocked on user · 6 fix loops with open BLOCKERs.

**Not a stop reason:** "tier floor loops done" with open FAILs.

---

## Deliverable

Include **conductor log**: tier, loops, waves dispatched, decisions, reviewer launch count, dogfood status.

See [`lfg-prompt.md`](lfg-prompt.md) deliverable section.
