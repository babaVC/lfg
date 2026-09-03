# LFG — Execute at proud bar (Conductor mode)

Paste after plan approval. Replace `{PLAN}`, `{PROJECT}`, `{PLAN_PATH}`.

**You are the conductor.** Dispatch subagents to build and review. Evaluate progress after every wave. Decide the next step. Loop until proud bar YES.

Full loop spec: [`conductor-loop.md`](conductor-loop.md). Platform dispatch: [`adapters/README.md`](adapters/README.md). Invariant grep: [`invariant-grep.md`](invariant-grep.md). UI/e2e smoke: [`e2e-smoke.md`](e2e-smoke.md).

Invoking **LFG** pre-authorizes VERIFY commands (build, lint, typecheck, test, dev server).

**Plan:** {PLAN}  
**Project:** {PROJECT}  
**Plan path:** {PLAN_PATH}

---

## Identity

| Role | You? | Job |
|---|---|---|
| **Conductor** | **Yes — parent** | Evaluate → decide → dispatch → integrate. Never self-certify. |
| **Builder** | No — subagent | Implement plan ([`builder-prompt.md`](builder-prompt.md)) |
| **Fixer** | Subagent or you | Fix FAILs only — not in same session as reviewer who found them |
| **Reviewers** | No — subagents | Read-only, dedicated prompt per lens ([`reviewer-prompts.md`](reviewer-prompts.md)) |

---

## LFG tier (M or L)

Conductor infers at kickoff from plan metadata, file count, and risk. State tier in scope lock.

| Tier | When | Loop 1 reviewers | Min loops | Early exit |
|------|------|------------------|-----------|------------|
| **M** | Feature slice, ~15–50 files (default) | Structure + Code + Edge (3); §3 if marketing | 2 | Exit criteria met → GATE |
| **L** | Auth, billing, MCP, security-critical, new subsystem | 4 core + triggered §5–§7 | 3 | Open BLOCKERs = 0 |

Plan may declare `LFG tier: M` or `LFG tier: L`. Default **M** if unstated.

**No LFG-S.** Quick fixes (≤15 files) → direct execution, not LFG.

---

## Capability

- **Multi-agent:** BUILD and REVIEW are separate subagent sessions when host supports it
- **Review model:** `{REVIEWER_MODEL}` — Cursor default **`composer-2.5-fast`**
- **Parallel reviewers:** launch independent lenses in one turn — count per tier and scope (see below)
- **Review-fix loops:** tier minimum (M=2, L=3) with **early exit** when exit criteria met — no ceremony loops
- **Different prompts each wave:** conductor scopes re-review ("only UI", "only files X–Y", "prior FAIL #2")

## No repo prerequisites

LFG is self-contained. The target repo does **not** need pre-configured:

- `AGENTS.md` Constraints / invariant greps
- Playwright or e2e scripts
- Smoke path section in the plan (soft — conductor infers)

**Discover** what exists in Phase 0. **Run** harness if found. **Add** greps, specs, or scripts **during this LFG session** when the plan needs them — not as a setup step before LFG starts. New package deps still need user approval.

---

## Phase 0 — PLAN REVIEW + BOOST (conductor, once)

Before scope lock or BUILD:

1. Read plan fully, project `AGENTS.md` if present, linked refs
2. **Discover harness** — scan `package.json` / e2e configs for Playwright or smoke scripts ([`e2e-smoke.md`](e2e-smoke.md)); record `E2E harness: found | none`
3. **Infer tier** M or L — state in scope lock
4. **Smoke path** — use plan section if present; else warn + infer from acceptance criteria (manual or discovered e2e command)
5. **Behavioral criteria** — flag structural-only vs behavioral acceptance criteria; boost plan notes if needed
6. **Vertical slices** — for multi-path/wizard plans (>50 files or multi-gate): suggest A→B→C slices if plan is a monolith
7. One structured pass — no sequential AskQuestion rounds unless blocked

Then **scope lock:** one line, in/out, plan path, tier, smoke path, **planned files: N** — [list].

Do not dispatch BUILD until scope is locked.

---

## Phase 1 — BUILD (builder subagent)

**Dispatch builder subagent** with [`builder-prompt.md`](builder-prompt.md). Conductor does not implement.

Wait for builder return: file list, summary, test output, risks, reviewer briefing (encouraged).

Copy builder briefing → `{REVIEWER_CONTEXT}` for reviewers.

---

## Phase 2 — VERIFY (after every BUILD or FIX)

Run until green:

1. **Automated:** `npm run build` · `npm run lint` · `npm run typecheck` · `npm test` (or project equivalents)
2. **Invariant grep:** generic checks always; ad-hoc + `AGENTS.md` if present ([`invariant-grep.md`](invariant-grep.md)) → `{INVARIANT_CHECKS}`. Persist new greps to `AGENTS.md` in-session when warranted.
3. **Smoke / e2e:** run discovered harness if found ([`e2e-smoke.md`](e2e-smoke.md)); else manual steps with evidence
4. **UI (if applicable):** dev server; Playwright/e2e if discovered; else dogfood or `RUNTIME UNVERIFIED`

Record all results for reviewers.

---

## Phase 3+ — CONDUCTOR LOOP

Repeat [`conductor-loop.md`](conductor-loop.md) until **exit criteria** met:

### Review loop exit (all tiers)

Exit review-fix loops when **ALL** true:

- VERIFY green (automated + invariant + smoke)
- Dispatched reviewers all PASS (per scope matrix — not always all 4)
- Zero open BLOCKER or MAJOR from integrated review

→ GATE → PROUD. **Do not** run loop N+1 for ceremony.

Tier sets a **floor** (M=2, L=3), not a target — exit early when criteria met.

### Loop 1 — scope-based reviewer dispatch

Before dispatch: run invariant grep if not done post-FIX.

| Plan scope | Dispatch | Mark N/A |
|------------|----------|----------|
| Backend + migration | §1 Structure, §2 Code, §4 Edge | §3 UI, §6 Copy |
| Wizard / app UI (not marketing LP) | §1, §2, §4 | §3 → **app UI subset** only (not full anti-slop) |
| Marketing LP / hero | §1–§4 + §6 Copy | — |
| Touches auth/secrets/API | + §5 Security | — |

Tier **L** or marketing scope: add §3 full anti-slop, §6 Copy, §5/§7 when triggered.

Each prompt includes `{REVIEWER_CONTEXT}`, `{INVARIANT_CHECKS}`, `{TEST_OUTPUT}`, `{DIFF}`.

1. **DISPATCH** reviewers per table above (parallel in one turn)
2. **INTEGRATE** — dedupe FAILs, sort BLOCKER → MAJOR → MINOR
3. **DISPATCH FIX** — fixer subagent or conductor inline (see inline fix rules)
4. **VERIFY** (full Phase 2)

### Loop 2, 3, … (conductor decides)

After each FIX, **EVALUATE** then dispatch **targeted** reviewers only:

- Structural fix → Structure + Code
- UI fix → UI reviewer with scoped prompt
- Copy fix → Copy reviewer
- New security surface → Security + Edge
- Regression check → Edge §4 scoped to prior manual checklist

Use [`reviewer-prompts.md` § Re-review scoped](reviewer-prompts.md#re-review-scoped-prompt) for loops 2+.

### Inline fix (conductor, no fixer subagent)

Conductor may FIX inline when:

- ≤3 findings total
- All MAJOR or below (no BLOCKER requiring design change)
- Files already read this session
- No new dependency or migration semantics

Otherwise → dispatch fixer subagent with integrated FAIL list.

---

## GATE & PROUD (conductor)

When exit criteria met:

1. Run [`gate-checklist.md`](gate-checklist.md) — every applicable item PASS/FAIL/N/A
2. **Dogfood** (~5 min) — conductor runs before PROUD YES (after e2e if harness ran):
   - Smoke path in browser/runtime — skip steps already covered by passing e2e
   - One unhappy path (refresh mid-flow, empty submit, delete/cancel)
   - Console clean on happy path
   - BLOCKERs found → one targeted FIX loop, not deferral
3. PROUD BAR B1–B5 — all YES

**If conductor cannot browser-verify:** deliverable must state **`Dogfood: RUNTIME UNVERIFIED`** with smoke steps for the user. Not a hard gate FAIL — explicit handoff, not fake confidence.

Any other FAIL → back into loop (FIX → VERIFY → targeted REVIEW).

---

## Reviewer roster (inline summary)

Each reviewer: read-only, `VERDICT: PASS | FAIL`, findings `[BLOCKER|MAJOR|MINOR|NIT] file:line`.

**§1 Structure** — plan coverage, files, scope creep. FAIL on any unmet criterion.

**§2 Code** — bugs, conventions, error handling. Isomorphic `@/` addendum when `src/lib/*` + `api/*` touched.

**§3 UI** — §3a app UI (wizard/forms) or §3b marketing anti-slop. N/A if no UI.

**§4 Edge** — empty/large/invalid inputs, tests, manual checklist for conductor.

**§5 Security** — secrets, authZ, injection.

**§6 Copy** — no prompt echo, no slop, banned words.

**§7 Performance** — N+1, pagination, bundle.

Full copy-paste prompts in [`reviewer-prompts.md`](reviewer-prompts.md).

---

## Do

- Run Phase 0 plan review before BUILD
- Dispatch builder for BUILD — do not code as conductor
- Invariant grep before first REVIEW
- Smoke path with evidence after every BUILD/FIX
- Scope-based reviewer dispatch — skip irrelevant lenses
- Evaluate after every subagent return; decide next wave explicitly
- Conductor dogfood before PROUD YES (or explicit RUNTIME UNVERIFIED)
- Stage + draft commit message (never commit unless user asked)

## Do NOT

- Build and review in same session
- One review pass and ship
- Re-run all 4 reviewers blindly every loop
- Stop with open FAILs because "tier minimum loops done"
- Skip smoke or invariant grep because VERIFY automated is green
- Ship PROUD YES without dogfood or explicit RUNTIME UNVERIFIED caveat

---

## Stop

**Done:** exit criteria met + GATE all PASS/N/A + dogfood PASS or RUNTIME UNVERIFIED + PROUD B1–B5 YES.

**Escalate:** thrashing (same FAIL twice) · blocked on user · 6 fix loops with open BLOCKERs.

**Never** silent FAILs.

---

## Deliverable

1. Conductor log — loops run, waves dispatched, tier, decisions made
2. Summary — shipped vs plan
3. Gate table — PASS/FAIL/N/A
4. Reviewer findings — resolved vs open
5. PROUD B1–B5
6. 5-layer eval (tool · steps · trajectory · task · business)
7. Commit message draft
8. Stats — tier · loops N · reviewer launches N
9. **Dogfood** — `PASS: [what was clicked]` OR `RUNTIME UNVERIFIED: [smoke steps for user]`
10. **E2E** — `PASS (N/N script)` | `N/A` | `MANUAL` | `ADDED this session`
