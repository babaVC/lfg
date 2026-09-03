# Cursor adapter

## Install

```bash
git clone https://github.com/babaVC/lfg.git ~/.cursor/skills/lfg
# or project: .cursor/skills/lfg
ln -sf ~/.cursor/skills/lfg/lfp ~/.cursor/skills/lfp
```

## Trigger

```
/lfg
```

Or say **lfg** / **LFG** / **let's fucking go**.

## Conductor + multi-agent (default LFG shape)

**Conductor = parent agent.** Does not BUILD. Orchestrates loop in [`conductor-loop.md`](../conductor-loop.md).

### 0. PLAN REVIEW — conductor

Phase 0 before BUILD: discover e2e harness (no prerequisite); infer tier M/L; smoke path; vertical slices if multi-gate.

### 1. BUILD — builder subagent

```
Task: description="LFG builder"
      model="inherit"
      subagent_type="generalPurpose"
      prompt=[builder-prompt.md with plan filled]
```

Wait for return before reviewers. Copy reviewer briefing → `{REVIEWER_CONTEXT}`.

### 2. VERIFY — conductor

1. build / lint / typecheck / test
2. Invariant grep ([`invariant-grep.md`](../invariant-grep.md)) → `{INVARIANT_CHECKS}`
3. E2e/smoke if discovered ([`e2e-smoke.md`](../e2e-smoke.md)); else manual

Pre-authorized when LFG invoked.

### 3. REVIEW — parallel reviewer subagents

**One parent turn**, multiple Task calls — count per tier and scope:

| Tier / scope | Typical loop 1 launches |
|--------------|-------------------------|
| M + wizard/app UI | 3 (§1, §2, §4) |
| M + marketing | 4–5 (+ §3b, §6) |
| L | 4–7 (+ §5–§7 when triggered) |

```
model="composer-2.5-fast"
subagent_type="generalPurpose"
run_in_background=false
read-only in prompt
```

Include `{REVIEWER_CONTEXT}`, `{INVARIANT_CHECKS}` in every prompt.

Loops 2+: conductor dispatches **only needed lenses** with scoped re-review prompt.

### 4. FIX — conductor or fixer subagent

**Inline fix** when ≤3 findings, all MAJOR or below, files already read, no new deps/migrations.

Otherwise dispatch fixer subagent with FAIL list. **Never** same session as reviewer.

### 5. Loop

Evaluate → decide → dispatch → integrate. **Tier floors:** M=2, L=3 — **exit early** when exit criteria met. No wave cap.

### 6. DOGFOOD — conductor

Before PROUD YES: smoke in browser, one unhappy path, console clean.

If browser unavailable: deliverable **`RUNTIME UNVERIFIED`** with steps for user.

### Optional

- **Bugbot** / **security-review** — large diff complement
- **Browser** subagent — UI verification with URL + checklist

## Model defaults

| Role | Model |
|---|---|
| Conductor | Parent picker |
| Builder / Fixer | `inherit` |
| Reviewers | **`composer-2.5-fast`** |
