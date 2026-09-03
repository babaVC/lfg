# LFG Handover — Speed Without Quality Loss

**Date:** 2026-08-14  
**Author:** Valera (ContextCore onboarding session)  
**Audience:** Agent tasked with improving the LFG skill  
**Source case study:** ContextCore three-gate onboarding (`docs/specs/2026-08-13-onboarding-three-gates.md`, ~60 files, LFG-M scope shipped as LFG-L)

---

## Executive summary

LFG reached **PROUD YES** on plan coverage and static review, then the user spent a **second full session** fixing runtime bugs reviewers never flagged: isomorphic `@/` imports crashing Node, welcome chat flicker, project-delete schema collision, StrictMode double-fire guards.

**Diagnosis:** The protocol optimizes for **agent launch count and static review depth**, not **time-to-working software**. Slowness comes from mandatory ceremony (min 3 loops, always 4 reviewers) and missing **runtime verification** + **project invariant checks** in the critical path.

**Goal of this handover:** Edit the skill so first prompt → user-verifiable end result is **~2× faster** on feature-sized work (LFG-M) without dropping the proud bar on failures users actually hit.

**Do not** weaken builder ≠ reviewer separation, VERIFY-after-fix, scope lock, or thrashing stops.

---

## Evidence — ContextCore onboarding run

### What LFG did (worked)

| Step | Outcome |
|------|---------|
| Scope lock + BUILD subagent | ~50 files implemented per plan |
| VERIFY | `npm run build` PASS, `npm test` 15/15 PASS |
| Loop 1 — 4 parallel reviewers | Found real BLOCKERs (BriefStep crash, empty gap interview, resume broken) |
| Fixer subagent | Fixed BLOCKERs + MAJORs |
| Loop 2 — targeted reviewers | Caught ref-guard gaps; conductor fixed 2 inline |
| Loop 3 — single reviewer | Verified ref guards — **ceremony, low signal** |
| GATE + deliver | Staged files, commit draft, AGENTS.md gap noted |

### What LFG missed (user found post-ship)

| Bug | Root cause | Would have caught |
|-----|------------|-------------------|
| Welcome turn `FUNCTION_INVOCATION_FAILED` | `sourceProvenance.js` used `@/` import in module imported by `api/*` | Invariant grep |
| Welcome chat flicker / empty state | Runtime remount + persist timing + StrictMode | Smoke path + browser VERIFY |
| `chatEligible is not defined` | Refactor prop not passed | Smoke path |
| Project delete failed | `token_usage` unique index + `ON DELETE SET NULL` | Edge reviewer if migration in scope — **smoke would not catch**; needs schema review prompt |
| Ingest "Thinking…" stall | Pump poll / stall UI — partial catch in loop 1 | Smoke path (watch ingest) |

### Time sinks (approximate agent wall time)

| Phase | Issue |
|-------|-------|
| Pre-LFG plan | 2 explore subagents + 3 sequential AskQuestion rounds |
| BUILD | Full subagent cold start on known codebase patterns |
| Loop 1 | 4 reviewers + fixer subagent — **appropriate** |
| Loops 2–3 | Loop 3 should not have run (exit criteria unclear vs TodoWrite) |
| Post-LFG | ~8 user-driven fix iterations — **declared "done" too early** |

---

## Root causes (protocol level)

1. **One size fits all** — LFG-L ceremony on LFG-M scope (wizard UI, no marketing LP).
2. **Reviewers are static** — no browser, no smoke script, no project-specific greps.
3. **Minimum 3 loops** contradicted by "unless zero FAILs after loop 1" — conductors follow TodoWrite ("min 3 loops").
4. **Fixer subagent for trivial fixes** — conductor already had files open; inline was faster.
5. **Plan lacked SMOKE criterion** — acceptance criteria were structural, not behavioral.
6. **Builder return contract thin** — reviewers re-derived risk map from diff alone.
7. **No dogfood gate** — deliverable ended at GATE; user testing was informal.

---

## Recommended changes — map to files

Implement in this order. Each item lists **target file(s)** and **acceptance criteria for the edit**.

### 1. LFG tiers (S / M / L)

**Files:** `SKILL.md`, `references/lfg-prompt.md`, `references/conductor-loop.md`, `references/adapters/cursor.md`, `README.md`

Add tier selection at kickoff (conductor picks from plan metadata or file count + risk):

| Tier | Scope | BUILD | Reviewers loop 1 | Min review-fix loops | Early exit |
|------|-------|-------|------------------|----------------------|------------|
| **S** | ≤15 files, no new tables, known patterns | Conductor inline OK | Code + Edge (2) | 1 | All PASS + smoke → GATE |
| **M** | 15–50 files, feature slice, migrations OK | Subagent default | Structure + Code + Edge (3); UI if marketing/LP | 2 | Loop 1 zero FAILs → GATE after smoke |
| **L** | New subsystem, auth, billing, MCP, security-critical | Subagent required | 4 core + triggered §5–§7 | 3 (current) | Only when open BLOCKERs = 0 |

**Replace everywhere:** "minimum 3 review-fix loops" → tier-specific minimum + shared **exit criteria** (below).

**Add to plan prerequisite (SKILL.md):** Plan must declare `LFG tier: S|M|L` or conductor infers and states tier in scope lock.

---

### 2. Unified loop exit criteria

**Files:** `SKILL.md`, `references/conductor-loop.md`, `references/lfg-prompt.md`, TodoWrite in `SKILL.md`

**STOP review loops when ALL true:**

- VERIFY green (build + lint + test per project)
- **Smoke path PASS** (see §3)
- **Invariant greps clean** (see §4)
- All **dispatched** reviewers PASS (not all 4 if tier S/M skipped lenses)
- No open BLOCKER or MAJOR from integrated review

Then → GATE → PROUD. **Do not** run loop N+1 for ceremony.

**Fix TodoWrite step 3:**

```
Old: Loop: REVIEW → FIX → VERIFY (min 3 loops)
New: Loop: REVIEW → FIX → VERIFY until tier exit criteria met
```

---

### 3. Smoke path — mandatory plan + VERIFY gate

**Files:** `SKILL.md` (prerequisites), `references/lfg-prompt.md` (Phase 2 VERIFY), `references/gate-checklist.md` (A5), `references/builder-prompt.md`

**Plan must include:**

```markdown
## Smoke path
- Manual: [clicks/URL/assertions] OR
- Script: `npm run <script>` (add script if missing)
```

**VERIFY phase (after BUILD and after every FIX):**

1. Automated: build · lint · typecheck · test
2. **Smoke:** run script OR conductor documents manual steps for user
3. UI features: dev server + happy path in browser when host supports it (Cursor: optional browser subagent or conductor note)

**Gate A5:** FAIL if smoke path defined in plan but not executed with evidence.

**Builder return** — add item 6 (see §6).

---

### 4. Invariant grep — conductor pre-review (no subagent)

**Files:** new `references/invariant-grep.md`, `SKILL.md`, `references/lfg-prompt.md`, `references/adapters/cursor.md`

Before first REVIEW dispatch, conductor runs project-specific greps (30s). Template:

```markdown
# Invariant grep template

Run after VERIFY green, before REVIEW. Read project AGENTS.md "Constraints (hard rules)" and add greps.

## Generic
- Secrets in diff: `git diff | rg -i '(api_key|secret|password)\s*='` → must be empty
- Debug noise: `rg 'console\.(log|debug)\(' --glob '*.js' [changed files]`

## Project-specific (add per repo)
- ContextCore example: `rg '@/constants' src/lib/ --glob '*.js'` → flag files also imported from api/*
```

Output: `{INVARIANT_CHECKS}` block pasted into every reviewer prompt.

**New gate (optional):** I1 Invariant grep — PASS when clean or findings fixed before REVIEW.

---

### 5. Scope-based reviewer dispatch (extend lite-mode logic)

**Files:** `references/conductor-loop.md`, `references/gate-checklist.md`, `references/conductor-playbook.md`, `SKILL.md` reviewer table

Gate checklist already has **lite mode** (≤5 files). **Mirror that matrix for REVIEW dispatch:**

| Plan scope | Dispatch | Mark N/A in prompts |
|------------|----------|---------------------|
| Backend + migration | §1 Structure, §2 Code, §4 Edge | §3 UI, §6 Copy |
| Wizard / app UI (not marketing LP) | §1, §2, §4 | §3 anti-slop → use §3 **app UI** subset only |
| Marketing LP / hero | §1–§4 + §6 Copy | — |
| Touches auth/secrets/API | + §5 Security | — |

**Add reviewer prompt §2 addendum** for isomorphic modules:

```
If plan touches src/lib/* AND api/*:
- Grep for @/ imports in files reachable from api/*
- FAIL on any @/ alias in isomorphic modules (Node cannot resolve)
```

---

### 6. Builder return — reviewer briefing

**Files:** `references/builder-prompt.md`, `references/lfg-prompt.md`

Add to builder **Return to conductor:**

```markdown
6. **Reviewer briefing** (5 bullets max):
   - Happy path to verify manually
   - Riskiest file(s) and why
   - Migrations / env vars to apply
   - Shortcuts or TODOs taken
   - AGENTS.md rules that might be violated
```

Conductor copies this into `{REVIEWER_CONTEXT}` for all reviewers.

---

### 7. Inline fix threshold (conductor / small FIX)

**Files:** `references/conductor-loop.md`, `references/adapters/cursor.md`

**Conductor may FIX inline** (no fixer subagent) when:

- ≤3 findings total
- All MAJOR or below (no BLOCKER requiring design change)
- Files already read this session
- No new dependency or migration semantics

Otherwise → dispatch fixer subagent with integrated FAIL list.

Document in conductor anti-patterns: **Fixer subagent for one-line typos** = waste.

---

### 8. Plan phase efficiency (pre-LFG, not LFG proper)

**Files:** `SKILL.md` (prerequisites), optional `references/plan-prerequisites.md`

Before LFG activates:

- Max **1** explore subagent if >5 anchor files unknown; else conductor reads directly
- **One** structured decision batch (multi-select), not sequential AskQuestion rounds
- Plan must contain: tier, smoke path, planned file list, in/out scope, refusals

Cross-link from SKILL.md: "Plan quality is LFG speed."

---

### 9. Dogfood checklist — deliverable §9

**Files:** `references/lfg-prompt.md` (Deliverable), `references/gate-checklist.md` (new D section or fold into A5)

Add to deliverable:

```markdown
9. **Dogfood** (conductor or user, ~5 min):
   - [ ] Smoke path executed in browser/runtime
   - [ ] One unhappy path (refresh mid-flow, empty submit, delete/cancel)
   - [ ] Console clean on happy path
   - BLOCKERs found → LFG-FIX (one targeted loop), not silent deferral
```

**Gate B4** ("No deferred known bugs"): FAIL if dogfood BLOCKERs listed but not fixed or escalated.

---

### 10. Vertical slices (optional plan pattern)

**Files:** `SKILL.md` (when NOT to use LFG-L on multi-path features), `references/conductor-playbook.md`

For multi-path features (e.g. scratch / GitHub / files):

- Plan may define slices A → B → C, each with own tier and smoke path
- Each slice: BUILD → VERIFY → REVIEW → GATE → dogfood → next slice
- Avoids single 60-file BUILD + LFG-L review monolith

Not mandatory — document as **recommended for multi-gate wizards**.

---

## Concrete edit checklist for implementing agent

Use this as your task list. All items merged 2026-08-14.

- [x] **SKILL.md** — M/L tiers; Phase 0 plan review; prerequisites (smoke soft); exit criteria; TodoWrite; invariant grep; dogfood
- [x] **README.md** — tier table; smoke + invariant grep
- [x] **references/lfg-prompt.md** — Phase 0; VERIFY smoke; exit criteria; deliverable §9 dogfood; tier M/L
- [x] **references/conductor-loop.md** — tier table; exit criteria; inline fix; scope dispatch; vertical slices
- [x] **references/conductor-playbook.md** — synced; vertical slices; dogfood
- [x] **references/builder-prompt.md** — reviewer briefing item 6 (encouraged)
- [x] **references/gate-checklist.md** — A5 smoke; I1 invariant; D1–D4 dogfood; B4; scope matrix
- [x] **references/reviewer-prompts.md** — §2 isomorphic addendum; §3a/§3b split; {REVIEWER_CONTEXT} {INVARIANT_CHECKS}
- [x] **references/adapters/cursor.md** — tier defaults; scope reviewer counts; inline fix; dogfood
- [x] **references/adapters/generic.md** + **claude-code.md** — tier/exit criteria
- [x] **NEW references/invariant-grep.md** — template + ContextCore example
- [ ] **OPTIONAL references/plan-prerequisites.md** — folded into Phase 0 in lfg-prompt.md per David

---

## Draft text snippets (paste-ready)

### SKILL.md — tier kickoff (insert after "Kickoff")

```markdown
## LFG tier (pick at kickoff)

| Tier | When |
|------|------|
| **S** | ≤15 files, no schema, known patterns |
| **M** | Feature slice, migrations, 15–50 files (default for most plans) |
| **L** | Auth, billing, MCP, new subsystem, security-critical |

State tier in scope lock. Plan may include `LFG tier: M`. Default **M** if unstated and >15 files.

Loop minimums: S=1, M=2, L=3 — **unless tier exit criteria met earlier** (see conductor-loop.md).
```

### VERIFY block (replace in lfg-prompt.md Phase 2)

```markdown
## Phase 2 — VERIFY (after every BUILD or FIX)

1. **Automated:** build · lint · typecheck · test (project scripts)
2. **Invariant grep:** run project checklist → `{INVARIANT_CHECKS}` (conductor, no subagent)
3. **Smoke path:** plan-defined script or documented manual steps — **evidence required**
4. **UI (if applicable):** dev server; happy path verified or assigned to dogfood with BLOCKER if blocked

Record all results for reviewers.
```

### Stop criteria (unify in conductor-loop.md)

```markdown
## Review loop stop (all tiers)

Exit review-fix loops when:
- VERIFY green
- Invariant grep clean (or findings fixed)
- Smoke path PASS
- Dispatched reviewers all PASS
- Zero open BLOCKER/MAJOR

→ GATE → PROUD. Do not increment loop count for ceremony.
```

---

## What NOT to change

| Keep | Reason |
|------|--------|
| Builder ≠ reviewer ≠ fixer same session | Prevents self-certification |
| VERIFY after every BUILD/FIX | Cheap compile/test signal |
| Scope lock before BUILD | Stops drift |
| Parallel independent reviewers | Loop 1 caught real BLOCKERs in onboarding |
| Thrashing stop (same FAIL twice) | Prevents infinite loops |
| Gate checklist PASS/FAIL/N/A | Forces explicit ship decision |
| Review model `composer-2.5-fast` on Cursor | Cost/speed balance for read-only |

---

## Success metrics (how to know the edit worked)

Re-run a **LFG-M** feature similar to onboarding (multi-step UI + API + migration):

| Metric | Before (onboarding) | Target after |
|--------|---------------------|--------------|
| Reviewer subagent launches | 4 + 2 + 1 = 7 | ≤5 (tier M, early exit) |
| Review-fix loops | 3 (loop 3 ceremonial) | 2 max with early exit |
| Post-LFG BLOCKERs found by user | ≥4 | 0–1 (invariant + smoke catch) |
| Time BUILD → GATE | ~full session | ~50–60% of prior |

---

## Case study smoke path (example for docs)

If onboarding were planned today:

```markdown
## Smoke path
1. `npm run build && npm test`
2. Manual: `/new` → GitHub path → connect (or mock) → ingest completes → approve one suggestion → rules continue → dashboard
3. Assert: chat rail opens "Getting started" thread; assistant message appears (not FailedTurnCard)
4. Unhappy: refresh at interview step → resumes without toast error
```

Invariant grep for ContextCore:

```bash
# api-reachable isomorphic imports must not use @/
rg "from '@/" src/lib/ --glob '*.js' -l | while read f; do
  rg -l "$f" api/ && echo "BLOCKER: $f uses @/ and is imported by api"
done
```

---

## Open questions for David (optional)

1. Should **LFG-S** allow conductor BUILD by default, or always subagent on Cursor?
2. Should smoke scripts be **required in plan** or conductor-inferred for tier M+?
3. Store per-project invariant grep in `.cursor/skills/lfg/projects/context-os.md` or in repo `AGENTS.md`?

---

## Handoff complete

**Status: implemented 2026-08-14** per David's decisions (M/L tiers only, smoke + invariant grep + scope reviewers as top priority, Phase 0 plan review, conductor dogfood with RUNTIME UNVERIFIED fallback).

Implementing agent: see edit checklist below for what was merged.
