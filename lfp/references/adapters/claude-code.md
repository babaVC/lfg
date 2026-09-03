# Claude Code adapter — LFP

## Install

```bash
git clone https://github.com/babaVC/lfg.git ~/.claude/skills/lfg
# LFP: ~/.claude/skills/lfp → lfg/lfp/
```

## Trigger

Say **lfp** / **let's fucking plan** in Claude Code.

## Research dispatch

Use Claude Code subagents or separate sessions with prompts from [`../research-prompts.md`](../research-prompts.md).

Launch parallel when independent:

1. **CODEBASE** — explore codebase (read-only)
2. **EXTERNAL** — web research when needed
3. **VALIDATOR** — when brief proposes a solution
4. **DEPENDENCIES** — auth, billing, migrations

Conductor integrates → option matrix → AskQuestion interviews → write plan.

## Plan file

If no CreatePlan equivalent: write DRAFT/LOCKED to `{project}/plans/{date}-{slug}.md` per [`../harness-plan.md`](../harness-plan.md).

## After LFP

Paste plan into new session with LFG prompt from [`../../../references/lfg-prompt.md`](../../../references/lfg-prompt.md).
