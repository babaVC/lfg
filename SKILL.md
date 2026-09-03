---
name: lfg
description: >-
  Maximum-quality agent execution protocol — exhaustive build, multi-wave review,
  hard gate checks. Conductor orchestrates multi-agent build, review-fix loops,
  and subagent dispatch. Use when the user says "lfg", "LFG", "let's fucking go".
  Builder subagent implements; reviewer subagents critique; conductor decides
  next waves. Not for quick fixes or time-boxed builds.
disable-model-invocation: true
---

# LFG — Let's Fucking Go

Maximum quality. No 70% bar. Leave no stone unturned.

**Universal protocol** — [`references/lfg-prompt.md`](references/lfg-prompt.md). **Conductor loop** — [`references/conductor-loop.md`](references/conductor-loop.md). **Cursor** — [`references/adapters/cursor.md`](references/adapters/cursor.md).

## When to trigger

| Trigger | Action |
|---|---|
| User says **lfg**, **LFG**, **let's fucking go** | Read this skill. Confirm plan exists. Run conductor workflow. |
| User approves a plan and says **go** / **execute** with LFG context | Same — plan is the contract. |
| User explicitly asks for **maximum quality** / **no shortcuts** | LFG unless scope is trivial (see below). |

## When NOT to use LFG

| Situation | Use instead |
|---|---|
| Quick fix, typo, one-line change (≤15 files) | Direct execution |
| Exploratory spike / "what if" | Plan mode or short session |
| 70% ship-it task with tight scope | Your project's fast-execution skill or direct mode |
| Time-boxed demo / hackathon round | Time-boxed build workflow |
| Single-pass eval flow | Lighter one-shot execution skill |
| Research-only, no build | Research / analysis mode |

**Prerequisite:** A LOCKED plan with LFG handover (§12). Ideally from **LFP** ([`lfp/SKILL.md`](lfp/SKILL.md) · [`lfp/references/lfg-handover.md`](lfp/references/lfg-handover.md)). If none exists — stop. Run `/lfp` first, then LFG.

**Plan quality is LFG speed.** Phase 0 reviews the plan and boosts smoke path, tier, and behavioral criteria before BUILD.

## No repo prerequisites

LFG does not require the target repo to ship with Playwright, e2e scripts, smoke paths, or `AGENTS.md` invariant greps pre-configured. Phase 0 **discovers** what exists; VERIFY **runs** it; BUILD **adds** harness during the session when the plan needs it. Persist greps/specs to the repo before ship — not as a separate pre-LFG setup task.

## LFG tier (M or L)

Conductor infers at kickoff. State tier in scope lock. Plan may include `LFG tier: M` or `LFG tier: L`.

| Tier | When | Loop 1 reviewers | Min loops | Early exit |
|------|------|------------------|-----------|------------|
| **M** | Feature slice, ~15–50 files (default) | Per scope matrix | 2 | Exit criteria met → GATE |
| **L** | Auth, billing, MCP, security-critical | 4 core + §5–§7 | 3 | Open BLOCKERs = 0 |

**No LFG-S.** Small fixes → direct execution, not LFG.

Loop floors: M=2, L=3 — **unless exit criteria met earlier** (see conductor-loop.md).

## Division of labor

- **User owns:** Plan approval, scope boundaries, final ship/no-ship, dependency approval.
- **Conductor (parent agent) owns:** Phase 0 plan review. Evaluate progress after every wave. Decide next step. Dispatch builder, reviewers, fixer. Invariant grep. Dogfood. Never implement BUILD. Never self-certify.
- **Builder subagent owns:** Wave BUILD — full plan implementation ([`builder-prompt.md`](references/builder-prompt.md)).
- **Reviewer subagents own:** Read-only critique — one dedicated prompt per lens. **`composer-2.5-fast`** on Cursor.
- **Fixer** (subagent or conductor): FIX waves only — not same session as reviewer who found the issue. Conductor may FIX inline for trivial sets (see conductor-loop.md).

## Conductor loop (not a fixed script)

```
PLAN REVIEW → EVALUATE → DECIDE → DISPATCH → INTEGRATE → repeat
```

Typical first path:

```
Phase 0 plan review → BUILD (subagent) → VERIFY (auto + invariant + smoke)
  → REVIEW (scope-based) → FIX → VERIFY → REVIEW (targeted)
  → … until exit criteria → GATE → dogfood → PROUD YES
```

Full spec: [`references/conductor-loop.md`](references/conductor-loop.md).

**Exit criteria** (stop review loops when ALL met — no ceremony loops):

- VERIFY green (automated + invariant grep + smoke)
- Dispatched reviewers all PASS
- Zero open BLOCKER or MAJOR

Reviewer copy-paste prompts: [`references/reviewer-prompts.md`](references/reviewer-prompts.md).

Hard gate checklist: [`references/gate-checklist.md`](references/gate-checklist.md).

Invariant grep: [`references/invariant-grep.md`](references/invariant-grep.md).

UI/e2e smoke: [`references/e2e-smoke.md`](references/e2e-smoke.md).

## Kickoff (when skill activates)

1. Confirm plan path or pasted spec. Read it fully. Read project `AGENTS.md` if present.
2. **Phase 0 plan review** — discover e2e harness; infer tier; smoke path (warn + infer if missing); vertical slices if multi-gate.
3. State **LFG scope** in one line: tier, in/out, smoke path summary.
4. Read and execute [`references/lfg-prompt.md`](references/lfg-prompt.md) with plan attached.
5. TodoWrite checklist (steps 0–6 below).
6. Do not ask the user mid-run unless blocked on architecture, scope, or new dependency.

**Terminal:** LFG invocation pre-authorizes VERIFY commands (build, lint, typecheck, test, dev server). The user opted into full blast — run them. Destructive or out-of-plan commands still need explicit approval.

No fluff openings.

## LFG prompt

**Canonical one-shot:** [`references/lfg-prompt.md`](references/lfg-prompt.md) — self-contained; paste after plan approval. Includes phases, tier, reviewer dispatch, gates, dogfood, deliverable.

Quick reference:

| Wave | Action |
|---|---|
| PLAN REVIEW | Tier, smoke, behavioral criteria, vertical slices |
| BUILD | **Builder subagent** — conductor does not code |
| VERIFY | build · lint · test · **invariant grep** · **e2e if discovered** · smoke |
| REVIEW | Scope-based dispatch; targeted loops 2+ |
| FIX | Fixer or conductor inline — then VERIFY |
| GATE + DOGFOOD + PROUD | Hard checklist + runtime check + B1–B5 |

Stop: exit criteria met + PROUD YES · or thrashing · or escalate. Not "tier floor loops done" with open FAILs.

---

## Reviewer dispatch (scope-based)

Not always four reviewers. See [`references/conductor-loop.md`](references/conductor-loop.md) scope matrix.

| Plan scope | Dispatch | Skip |
|------------|----------|------|
| Backend + migration | §1, §2, §4 | §3, §6 |
| Wizard / app UI | §1, §2, §4 | §3 → app UI subset only |
| Marketing LP | §1–§4 + §6 | — |
| Auth/secrets/API | + §5 Security | — |

Tier **L** or marketing: full 4 core + optional §5–§7.

Each reviewer returns: `VERDICT: PASS | FAIL`, findings `[BLOCKER|MAJOR|MINOR|NIT]`, file:line refs.

Include `{REVIEWER_CONTEXT}` and `{INVARIANT_CHECKS}` in every dispatch.

## Gate checklist (hard blocks)

Every applicable item is **PASS, FAIL, or N/A**. One FAIL blocks ship.

Full list: [`references/gate-checklist.md`](references/gate-checklist.md).

**Smoke (A5):** evidence required when smoke path defined. **Dogfood:** conductor runs or `RUNTIME UNVERIFIED` in deliverable.

**Optional complements:** platform diff/security review (Cursor: Bugbot; see adapter).

## Stop criteria

| Condition | Action |
|---|---|
| Exit criteria + gates PASS + proud bar YES | **Done.** Deliver summary + commit draft. |
| Tier floor met but FAILs remain | **Keep looping** or escalate at 6 fix loops with BLOCKERs |
| Same FAIL twice after fix | **Stop.** Likely thrashing — escalate with root cause. |
| Blocked on the user | **Pause.** One clear question; don't guess. |
| Scope creep detected | **Stop.** Diff vs plan; ask before continuing. |

## 5-layer eval (conductor applies at end)

| Layer | Question |
|---|---|
| Tool correctness | Did commands run and produce trustworthy output? |
| Step quality | Was each wave executed fully, not skipped? |
| Trajectory coherence | Did fixes align with plan, not drift? |
| Task completion | Every acceptance criterion evidenced? |
| Business outcome | Would this achieve what the plan intended for the user? |

## Optional companion skills

| Companion | Use with LFG when |
|---|---|
| **anti-slop-frontend** (or similar) | Marketing UI — §3b reviewer reads it for design gates |
| **review-bugbot / review-security** | Large diffs — complements §5, not a REVIEW substitute |
| **Project fast-execution skill** | Opposite mode — 70% ship vs LFG proud |

## TodoWrite checklist

```
0. Phase 0 plan review + scope lock (tier, smoke)
1. Dispatch BUILD subagent
2. VERIFY (automated + invariant grep + smoke)
3. Loop: EVALUATE → dispatch REVIEW(s) per scope → FIX → VERIFY until exit criteria
4. GATE + dogfood (or RUNTIME UNVERIFIED)
5. PROUD + deliver — include conductor log
```

## Reference index

- [`references/handover-2026-08-14-speed-improvements.md`](references/handover-2026-08-14-speed-improvements.md) — case study + rationale (implemented 2026-08-14)
- [`lfp/SKILL.md`](lfp/SKILL.md) — **planning skill** (`/lfp`)
- [`lfp/references/lfp-prompt.md`](lfp/references/lfp-prompt.md) — planning paste prompt
- [`references/invariant-grep.md`](references/invariant-grep.md) — pre-review grep (discover or add in-session)
- [`references/e2e-smoke.md`](references/e2e-smoke.md) — Playwright/e2e discover, run, add in-session
- [`references/lfg-prompt.md`](references/lfg-prompt.md) — **universal paste** after plan approval
- [`references/conductor-loop.md`](references/conductor-loop.md) — **multi-agent loop spec**
- [`references/builder-prompt.md`](references/builder-prompt.md) — builder subagent dispatch
- [`references/adapters/README.md`](references/adapters/README.md) — Cursor · Claude Code · generic
- [`references/conductor-playbook.md`](references/conductor-playbook.md) — detailed reference
- [`references/reviewer-prompts.md`](references/reviewer-prompts.md)
- [`references/gate-checklist.md`](references/gate-checklist.md)

## What this skill is NOT

- Not a 70% ship-it mode
- Not a time-boxed demo workflow
- Not a plan generator — use **LFP** ([`lfp/SKILL.md`](lfp/SKILL.md)); LFG Phase 0 only boosts an existing plan
- Not for quick fixes — use direct execution
- Not permission to commit — stage and draft only unless the user asked
