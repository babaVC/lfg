# LFG Handover — required section spec

Every LOCKED plan must end with §12 filled from this spec. An execution agent with **only the plan file** must be able to run `/lfg` without chat history.

Copy into plan §12:

```markdown
## 12. LFG Handover

**Plan path:** {path to this file}
**LFG tier:** M | L
**Suggested tier rationale:** {one line — file count, risk surface}

### In scope
- …

### Out of scope
- …

### Acceptance criteria
- [ ] …

### Smoke path
{Steps or command — manual or discovered e2e}

### Testing instructions (VERIFY bar)
- build: `{command or project default}`
- lint: …
- typecheck: …
- test: …
- e2e: found | none | add in-session
- manual smoke: …

### Technical decisions (for execution agents)
- …

### Constraints from brief/interviews (verbatim)
> …

### Do-not-touch / in-flight work
- …

### Patterns to copy
| Pattern | Path |
|---------|------|
| … | `path/to/file` |

### Hard rules from AGENTS.md
- …

### Dependency approvals needed
- {none | list — user must approve before BUILD}

### Execution remarks
{Anything said in chat that is NOT captured above — tone, timing, comms, "David said…"}

### Paste to execute
Attach this plan and invoke **/lfg** (or say **lfg** with plan path).
```

## Conductor checklist before LOCKED

- [ ] Every interview decision appears in §2 or §10
- [ ] Research in §5 — no "go re-read the codebase" without pointers
- [ ] LFG tier M or L (no S)
- [ ] Smoke path exists or explicit "infer in LFG Phase 0"
- [ ] Zero open `pre-execution` questions unless user explicitly deferred with owner
- [ ] §12 complete — handoff test: could a fresh agent start LFG?
