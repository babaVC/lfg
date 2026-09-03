# LFG Invariant Grep

Conductor runs this **after VERIFY green, before first REVIEW dispatch**. No subagent — ~30 seconds.

Output becomes `{INVARIANT_CHECKS}` pasted into every reviewer prompt.

**No repo prerequisites.** The project does not need a pre-configured Constraints section or grep catalog. Generic checks always run; project-specific checks are discovered or invented during the session.

## How to use

1. Run **generic checks** below on changed files (always)
2. Read project `AGENTS.md` if present — use any existing Constraints / invariant greps
3. If the diff suggests a risk class (e.g. `src/lib/*` + `api/*`), run **ad-hoc greps** even if not documented yet
4. Record results: clean, or findings fixed before REVIEW
5. **Persist** new greps to `AGENTS.md` during this LFG session when you identify a repeatable bug class — not as a pre-LFG setup task

If findings are BLOCKER-level, FIX before dispatching reviewers.

---

## Generic checks

Run on changed files in the diff:

```bash
# Secrets in diff — must be empty
git diff | grep -iE '(api_key|secret|password)\s*=' || true

# Debug noise in changed JS/TS files
grep -rn 'console\.\(log\|debug\)(' [changed files] || true
```

Use the Grep tool in Cursor; use `grep -E` in shell scripts on this machine.

---

## Project-specific greps (discover or add in-session)

**If `AGENTS.md` has Constraints** — run those greps.

**If not** — conductor infers from diff and plan (no upfront repo setup required). When a grep catches a real issue, add it to `AGENTS.md` before ship so the next LFG run inherits it.

Template when adding during LFG:

```markdown
## Constraints (hard rules)

### Invariant greps (LFG)
- `[description]`: `[grep command or pattern]`
```

---

## Example ad-hoc grep (run when diff warrants — not a repo prerequisite)

Isomorphic modules imported by `api/*` must not use `@/` path aliases (Node cannot resolve them):

```bash
grep -rl "from '@/" src/lib/ --include='*.js' 2>/dev/null | while read f; do
  grep -rl "$f" api/ 2>/dev/null && echo "BLOCKER: $f uses @/ and is imported by api"
done
```

Run when plan touches shared lib + API routes — even if never documented before.

---

## Output block

Paste into reviewer prompts:

```markdown
## Invariant checks (conductor, pre-review)

| Check | Result | Notes |
|-------|--------|-------|
| Secrets in diff | PASS/FAIL | ... |
| Debug noise | PASS/FAIL/N/A | ... |
| [ad-hoc or AGENTS.md] | PASS/FAIL/N/A | ... |

**Overall:** CLEAN | FINDINGS FIXED | BLOCKERS OPEN
**Persisted to AGENTS.md:** yes/no — [what was added, if any]
```

Gate **I1**: PASS when CLEAN or findings fixed before REVIEW.
