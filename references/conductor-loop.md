# LFG Conductor Loop

The conductor is the **parent agent**. The conductor does not implement BUILD and does not self-certify. The conductor **evaluates progress, decides the next wave, and dispatches dedicated subagents**.

This replaces a fixed 7-step script. The loop below is the protocol.

---

## Roles (never merge in one session)

| Role | Who | May edit files? |
|---|---|---|
| **Conductor** | Parent agent | FIX wave only — or dispatch Fixer subagent |
| **Builder** | Subagent | Yes — plan implementation only |
| **Fixer** | Subagent or conductor | Yes — address FAIL findings only |
| **Reviewer** | Subagent | **Never** — read-only |

Builder ≠ Reviewer. Reviewer ≠ Fixer in the same session.

---

## LFG tier (M or L)

| Tier | When | Loop 1 reviewers | Min loops |
|------|------|------------------|-----------|
| **M** | Feature slice, ~15–50 files (default) | Per scope matrix (typically §1+§2+§4) | 2 |
| **L** | Auth, billing, MCP, security-critical | 4 core + §5–§7 when triggered | 3 |

Conductor infers tier at Phase 0 plan review. **No LFG-S** — small fixes use direct execution.

---

## The loop

Repeat until **PROUD BAR YES** or **escalate to user**:

```
┌──────────────┐
│  EVALUATE    │  What shipped? What's open? What failed last wave?
└──────┬───────┘
       ▼
┌──────────────┐
│  DECIDE      │  Pick next wave(s) from catalog — see decision tree
└──────┬───────┘
       ▼
┌──────────────┐
│  DISPATCH    │  One or more subagents, each with a dedicated prompt
└──────┬───────┘
       ▼
┌──────────────┐
│  INTEGRATE   │  Merge findings, update FAIL tracker, re-run VERIFY if needed
└──────┬───────┘
       │
       └──► loop or STOP
```

After every DISPATCH, conductor writes a **one-paragraph status** (internal): tier, cycles completed, FAILs open, next wave rationale.

---

## Wave catalog

Conductor picks **one or more** per loop iteration. Launch parallel when independent.

| Wave | Subagent? | Prompt source | When |
|---|---|---|---|
| **PLAN REVIEW** | No | conductor | Phase 0 — tier, smoke, vertical slices, scope lock |
| **LOAD** | No | conductor | Scope lock with planned files |
| **BUILD** | **Yes — default** | `builder-prompt.md` | First implementation; major plan additions |
| **VERIFY** | Conductor | automated + invariant + smoke | After every BUILD or FIX |
| **REVIEW:structure** | Yes | `reviewer-prompts.md` §1 | After first BUILD; after structural FIX |
| **REVIEW:code** | Yes | §2 | After first BUILD; after logic FIX |
| **REVIEW:ui** | Yes | §3a or §3b | UI surfaces; after design FIX |
| **REVIEW:edge** | Yes | §4 | After first BUILD; after behavior FIX |
| **REVIEW:security** | Yes | §5 | Auth, secrets, API boundaries |
| **REVIEW:copy** | Yes | §6 | Reader-facing strings, marketing |
| **REVIEW:perf** | Yes | §7 | Lists, fetches, bundle-sensitive |
| **REVIEW:regression** | Yes | §4 scoped to prior FAILs | Re-review after FIX |
| **FIX** | Yes or conductor | plan + FAIL list | After any REVIEW with open FAILs |
| **GATE** | No | `gate-checklist.md` | When exit criteria met |
| **DOGFOOD** | Conductor | `lfg-prompt.md` §9 | Before PROUD — browser/runtime check |
| **PROUD** | No | B1–B5 | After GATE + dogfood |

**First BUILD:** dispatch **Builder subagent** — conductor does not code.

**Before first REVIEW:** invariant grep ([`invariant-grep.md`](invariant-grep.md)) → `{INVARIANT_CHECKS}`.

**First REVIEW:** dispatch per **scope matrix** (not always 4 core):

| Plan scope | Dispatch | N/A |
|------------|----------|-----|
| Backend + migration | §1, §2, §4 | §3, §6 |
| Wizard / app UI | §1, §2, §4 | §3 → §3a app UI subset |
| Marketing LP | §1–§4 + §6 | — |
| Auth/secrets/API | + §5 | — |

Tier **L**: default to 4 core + triggered optional lenses.

**Later REVIEW waves:** conductor selects **only lenses that match what changed**. Use scoped re-review prompts.

---

## Review loop exit (replaces fixed minimum)

**Stop review-fix loops when ALL true:**

- VERIFY green (automated + invariant grep + smoke)
- Dispatched reviewers all PASS
- Zero open BLOCKER or MAJOR

→ GATE → DOGFOOD → PROUD. **Do not** increment loop count for ceremony.

**Tier floors** (minimum loops before you *may* stop — still need exit criteria):

| Tier | Floor |
|------|-------|
| M | 2 |
| L | 3 |

If exit criteria met before floor, still stop. If floor met but FAILs open, keep looping.

Track: `tier`, `loop N`, `reviewer launches`, `open FAILs`.

---

## Inline fix threshold

Conductor may FIX inline (no fixer subagent) when:

- ≤3 findings total
- All MAJOR or below (no BLOCKER requiring design change)
- Files already read this session
- No new dependency or migration semantics

Otherwise → dispatch fixer subagent with integrated FAIL list.

**Anti-pattern:** fixer subagent for one-line typos = waste.

---

## Vertical slices (multi-path features)

For multi-gate wizards or multi-path features (e.g. scratch / GitHub / files):

- Phase 0: conductor suggests slices A → B → C if plan is a monolith
- Each slice: BUILD → VERIFY → REVIEW → GATE → dogfood → next slice
- Avoids single 60-file BUILD + L-tier review monolith

Recommended when plan has independent paths and >50 files.

---

## Conductor decision tree (after EVALUATE)

```
Plan not reviewed?
  → Phase 0 PLAN REVIEW → scope lock

No BUILD yet?
  → DISPATCH BUILD

BUILD done, VERIFY red?
  → FIX → VERIFY — stay in pre-review

BUILD done, VERIFY green, invariant not run?
  → INVARIANT GREP → fix BLOCKERs

Invariant clean, no REVIEW yet?
  → DISPATCH reviewers per scope matrix (parallel)

REVIEW returned FAILs?
  → FIX (inline or subagent) → VERIFY
  → DECIDE which reviewers to re-dispatch
  → loop until exit criteria

Exit criteria met, GATE not run?
  → RUN GATE

GATE PASS, dogfood not run?
  → DOGFOOD (or RUNTIME UNVERIFIED in deliverable)

PROUD YES?
  → STOP — deliver
```

---

## Thrashing & escalation (only hard stops)

| Trigger | Action |
|---|---|
| Same FAIL id / file:line twice after fix | STOP — escalate root cause to user |
| Blocked on user (scope, dep, architecture) | PAUSE — one question |
| Reviewer infra failed twice | Conductor inline pass or escalate |
| 6 fix loops with open BLOCKERs | STOP — report; user decides |

Do **not** stop early because "looks good." Do **not** stop at tier floor if FAILs remain.

---

## Parallel dispatch rules

- **Independent reviewers** → one parent turn, multiple subagents (Cursor: parallel Task calls)
- **BUILD** → one subagent; wait for return before REVIEW
- **FIX + REVIEW** → never same subagent; FIX completes before next REVIEW
- Optional: **FIX subagent** for large FAIL sets; conductor integrates

See [`adapters/README.md`](adapters/README.md) for platform plumbing.

---

## Model defaults (Cursor)

| Role | Model |
|---|---|
| Conductor | Parent picker (capable) |
| Builder | `inherit` |
| Fixer | `inherit` or builder model |
| Reviewers | **`composer-2.5-fast`** — fast, read-only, parallel |

Other platforms: see adapter — role separation matters more than model.

---

## Conductor anti-patterns

| Anti-pattern | Why it fails |
|---|---|
| Conductor implements BUILD | No separation; no review isolation |
| One REVIEW pass total | Not LFG |
| Re-run identical 4 reviewers every loop | Waste; misses targeted coverage |
| Full anti-slop on wizard/app UI | Wrong lens; use §3a |
| Conductor self-reviews instead of dispatch | Builder cannot certify |
| Stop at tier floor with open FAILs | Violates proud bar |
| Skip smoke/invariant because build passed | VERIFY ≠ runtime quality |
| Fixer subagent for trivial fixes | Wasted cold start |
| PROUD YES without dogfood or RUNTIME UNVERIFIED | User finds bugs post-ship |
