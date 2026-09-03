# LFG UI / E2E smoke

LFG does **not** require Playwright, e2e scripts, or smoke harnesses to exist before a run. Discover what the repo has; add what's missing **during the session** when the plan needs it.

## Phase 0 — discover (no prerequisites)

Scan the project — do not fail or block if missing:

| Look for | Where |
|----------|--------|
| Playwright / Cypress / e2e scripts | `package.json` scripts, `playwright.config.*`, `scripts/e2e/` |
| Existing smoke specs | `*.spec.js`, `*.e2e.*`, `test:ui`, `test:e2e` |
| Dev server URL | project docs, `AGENTS.md`, portless registry |

Record in scope lock: **`E2E harness: found | none`** and the command if found.

If UI in scope and **no harness**: infer manual smoke for VERIFY; note in plan boost whether BUILD should add a minimal spec (plan decision, not LFG default).

---

## VERIFY — run if present, don't require

After automated build/lint/test:

1. **Harness found** → run it (e.g. `npx playwright test --config …`). Evidence = pass/fail count + trace path on failure. One retry allowed.
2. **No harness, UI in scope** → manual smoke steps (conductor or dogfood) + `RUNTIME UNVERIFIED` if browser blocked
3. **Backend-only** → N/A

Playwright is **not** a reviewer subagent. It runs in VERIFY/smoke, same layer as invariant grep.

---

## Adding harness during LFG (in-session only)

When plan includes UI behavioral criteria and no e2e exists:

- **BUILD** may add a minimal happy-path spec + npm script — if plan explicitly includes it
- **New dependencies** (e.g. `@playwright/test`) → user approval per project rules; not bundled into LFG skill
- Do not pre-seed repos before LFG starts; do not maintain a central LFG catalog of per-project scripts

After adding specs in BUILD, VERIFY runs them in the same session.

---

## Gate mapping

| Gate | When harness exists | When no harness |
|------|---------------------|-----------------|
| **A5** Smoke | Run e2e or script; FAIL if defined but not run | Manual smoke or RUNTIME UNVERIFIED |
| **A6** E2E | PASS when script exits 0 | N/A |

---

## Deliverable line

```markdown
**E2E:** PASS (3/3 onboarding-ui.spec.js) | N/A (backend-only) | MANUAL (no harness) | ADDED (spec + script this session)
```

Dogfood follows e2e: if Playwright passed, dogfood focuses on unhappy paths e2e doesn't cover.
