# LFG Gate Checklist

Hard pass/fail gates. **One FAIL blocks ship.** Conductor marks each item PASS or FAIL with one-line evidence.

Run after reviewer waves. Map reviewer findings to items below.

## Lite mode (small non-UI changes)

Use when **≤5 files changed** and **zero UI/copy** changes. Required gates only:

**P1–P4 · A1–A4 · I1 · C1–C5 · G1–G3 · B1–B5**

Mark U*, R*, E*, S*, F* as **N/A**. Dispatch reviewers per scope matrix — they return N/A for irrelevant sections.

## Scope → applicable gates & reviewers

| Scope | Applicable gates | Reviewers | Mark N/A |
|---|---|---|---|
| Backend-only, no copy | P, A, I, C, E, S, G, B | §1, §2, §4 | U1–U12, R1–R5 |
| Wizard / app UI | P, A, I, C, E, G, B | §1, §2, §4; §3a if UI | §3b anti-slop, R unless copy |
| Docs-only | P, G, B | §1 | A3–A5, I, C, U, R, E, S, F |
| Marketing UI | All sections | §1–§4 + §6 | — |
| Lite (≤5 files, no UI) | Lite set above | §2 + §4 | U, R, E, S, F |

---

## Full mode (default)

All sections below apply when scope warrants.

---

## Plan coverage

| # | Gate | PASS when |
|---|---|---|
| P1 | All acceptance criteria met | Each criterion has evidence (file, test, screenshot, smoke) |
| P2 | No unplanned scope | Diff matches plan; extras flagged or removed |
| P3 | Named outputs exist | Every planned file path present |
| P4 | Plan refusals honored | Do-not-do items from plan not violated |

---

## Automated verification

| # | Gate | PASS when |
|---|---|---|
| A1 | Build | `npm run build` (or equivalent) exits 0 |
| A2 | Lint | Linter exits 0 if script exists; N/A documented if no linter |
| A3 | Typecheck | `tsc --noEmit` / typecheck exits 0 if TS project |
| A4 | Tests | All existing tests pass; new logic has tests if project tests touched code |
| A5 | Smoke path | Plan-defined, inferred, or discovered e2e executed with evidence. **FAIL** if smoke/e2e command defined this session but not run. **N/A** if zero runtime behavior |
| A6 | E2E harness | When Playwright/e2e script exists in repo: run in VERIFY; PASS on exit 0. **N/A** when no harness (manual smoke + dogfood instead). **FAIL** only if harness existed and was not run |

---

## Invariant grep

| # | Gate | PASS when |
|---|---|---|
| I1 | Invariant grep | Generic checks run; ad-hoc or AGENTS.md greps if applicable; CLEAN or fixed before REVIEW. New greps may be persisted in-session |

---

## Code quality

| # | Gate | PASS when |
|---|---|---|
| C1 | No blockers | Zero BLOCKER findings from code reviewer |
| C2 | No majors open | Zero unresolved MAJOR code findings |
| C3 | Conventions | Matches project patterns in touched files |
| C4 | Error handling | Failure paths handled, not swallowed |
| C5 | Hygiene | No debug logs, dead code, TODO/FIXME unless in plan |
| C6 | Dependencies | No new deps without user approval |

---

## UI / design (anti-slop)

Apply to **marketing surfaces**, heroes, outreach, LP sections, drop intros. **Wizard/app UI** uses §3a subset — mark U1–U6 N/A if only app UI (forms, wizards, dashboards).

Reference: anti-slop-frontend skill if installed; project brand guidelines if present.

| # | Gate | PASS when |
|---|---|---|
| U1 | Design Read | Page kind + audience + vibe match plan |
| U2 | Typography | Display hierarchy, body line-height ≥1.55, ~65ch prose |
| U3 | Color & surfaces | Brand tokens; no AI gradient hero; one accent; `#111` not `#000` |
| U4 | Layout | No three-equal-cards-only; asymmetric where earned; `100dvh` not `100vh` |
| U5 | AI tells absent | No scroll cues, fake dashboard hero, status dots, version pills |
| U6 | Motion | Short on-brand transitions; `prefers-reduced-motion` honored |
| U7 | States | Empty, loading, error states intentional |
| U8 | Responsive | Layout works mobile + desktop (verified in browser or code review) |
| U9 | UI reviewer | UI/design reviewer verdict PASS |

### UI spot-check (conductor or browser)

| # | Gate | PASS when |
|---|---|---|
| U10 | First paint | Hero/key view correct without scroll (if applicable) |
| U11 | Focus & contrast | CTA contrast WCAG AA; focus visible |
| U12 | Landmarks | `header`, `main`, `section` semantic |

---

## Copy / reader-facing

Mark **N/A** if zero reader-facing strings changed.

| # | Gate | PASS when |
|---|---|---|
| R1 | No prompt echo | Reader-facing text doesn't restate instructions |
| R2 | No UI narration | Copy doesn't explain what UI already shows |
| R3 | No agent meta | No "this section captures…" process lines |
| R4 | Banned words | No em-dashes, Elevate, Seamless, Unleash, etc. |
| R5 | Real placeholders | No Lorem, John Doe in ship-ready strings |

---

## Edge cases & testing

Mark E1–E3 **N/A** if no runtime behavior change. E5 always applies when edge reviewer ran.

| # | Gate | PASS when |
|---|---|---|
| E1 | Empty states | Zero/empty input handled gracefully |
| E2 | Error paths | Network/validation failures surfaced to user |
| E3 | Boundaries | Large lists paginated or bounded |
| E4 | Edge reviewer | Edge cases reviewer verdict PASS |
| E5 | Manual checklist | Conductor completed edge reviewer's manual verify list — evidence in gate report |

---

## Security

| # | Gate | PASS when |
|---|---|---|
| S1 | No secrets | No keys/tokens in diff |
| S2 | Auth boundaries | User data access correct if auth touched |
| S3 | Injection | No new XSS/SQL/command sinks unvalidated |

Mark S1–S3 N/A if change is docs-only with no runtime surface.

---

## Performance

| # | Gate | PASS when |
|---|---|---|
| F1 | No unbounded fetch | Lists/API calls paginated or limited |
| F2 | No obvious regressions | No multi-MB assets added without reason |
| F3 | Bundle discipline | No heavy deps for trivial features |

Mark N/A for non-runtime changes.

---

## Git & delivery

| # | Gate | PASS when |
|---|---|---|
| G1 | Staged | Relevant files staged (not secrets) |
| G2 | Commit draft | Paste-ready message follows repo style |
| G3 | No commit unless asked | `git commit` not run unless the user requested |

---

## Dogfood (conductor, before PROUD)

| # | Gate | PASS when |
|---|---|---|
| D1 | Smoke in runtime | Conductor ran smoke path in browser/runtime — evidence in deliverable §9 |
| D2 | Unhappy path | One edge tried (refresh mid-flow, empty submit, delete/cancel) |
| D3 | Console clean | No errors on happy path |
| D4 | Runtime handoff | If conductor could not browser-verify: deliverable states **`RUNTIME UNVERIFIED`** with steps for user — not silent deferral |

Mark D1–D3 N/A only for backend-only with no UI and smoke is script-only. D4 applies when browser unavailable.

---

## Proud bar (conductor — all must PASS)

| # | Gate | PASS when |
|---|---|---|
| B1 | Merge without caveats | Conductor answers YES (or YES with RUNTIME UNVERIFIED caveat documented) |
| B2 | Senior designer | Would show UI without apologizing — YES or N/A |
| B3 | Acceptance evidence | Every plan criterion traced — YES |
| B4 | No deferred known bugs | Zero known FAILs or dogfood BLOCKERs deferred — YES |
| B5 | No stray TODO/FIXME | Zero unless explicitly in plan — YES |

---

## Gate report template

Conductor returns this table filled:

```markdown
| Gate | PASS/FAIL/N/A | Evidence |
|------|---------------|----------|
| P1   | PASS          | ... |
| A5   | PASS          | smoke: /new → GitHub → ingest completes |
| I1   | PASS          | invariant grep clean |
| D1   | PASS          | dogfood: welcome chat opens, no FailedTurnCard |
| ...  | ...           | ... |
| B5   | PASS          | ... |

**Ship status:** READY | BLOCKED (list FAIL ids)
**Tier:** M | L
**Dogfood:** PASS | RUNTIME UNVERIFIED
**Cycles used:** N (floor: M=2, L=3)
**Reviewer launches:** N
**5-layer eval:** tool · steps · trajectory · task · business — PASS/FAIL each
```

Unmarked applicable row = FAIL. N/A requires one-line reason.

**READY** only when every applicable row is PASS.
