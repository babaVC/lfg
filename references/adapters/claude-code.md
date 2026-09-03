# Claude Code adapter

## Install

```bash
git clone https://github.com/babaVC/lfg.git ~/.claude/skills/lfg
ln -sf ~/.claude/skills/lfg/lfp ~/.claude/skills/lfp
```

## Trigger

User says **lfg**, **LFG**, **let's fucking go** after plan approval — or add a slash command / hook if your setup supports it.

## Workflow

**Phase 0:** Plan review — infer tier M/L, smoke path, vertical slices.

**BUILD:** dispatch **builder subagent** with [`builder-prompt.md`](../builder-prompt.md). Conductor evaluates return, runs VERIFY, then enters [`conductor-loop.md`](../conductor-loop.md).

**VERIFY:** automated + invariant grep + e2e if discovered + smoke ([`e2e-smoke.md`](../e2e-smoke.md)).

**REVIEW:** scope-based dispatch — tier M typically §1+§2+§4; tier L adds §3–§7 when triggered.

**Loop:** tier floors M=2, L=3 — exit early when exit criteria met.

**DOGFOOD:** conductor runs or documents `RUNTIME UNVERIFIED`.

Sequential role-switch if no subagents — still separate BUILD context from REVIEW context.

Read-only: reviewers must not edit files. Conductor applies fixes in FIX wave (inline or subagent).

## Optional complements

- Project security review command / custom agent for §5
- Separate copy-review pass for §6

## Notes

- `AGENTS.md` at repo root — read if present; invariant greps may be **added during LFG**, not required upfront
- Terminal: LFG pre-authorizes VERIFY commands when user invoked LFG explicitly
