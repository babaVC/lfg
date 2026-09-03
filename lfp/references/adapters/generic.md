# Generic adapter — LFP

No subagent infrastructure required — run research **sequentially** in separate chats or turns.

## Setup

1. Copy [`../lfp-prompt.md`](../lfp-prompt.md) into your project
2. Fill `{BRIEF}`, `{PROJECT}`, `{PLAN_PATH}`

## Research (sequential)

For each applicable wave in [`../research-prompts.md`](../research-prompts.md):

1. Open read-only session (or paste "read only" in prompt)
2. Paste CODEBASE / EXTERNAL / VALIDATOR / DEPENDENCIES block
3. Copy return into conductor's Option Brief

## Plan file

Create and update `{project}/plans/{date}-{slug}.md` per [`../harness-plan.md`](../harness-plan.md).

## Interviews

Use structured questions (2–4 per round) — same content as AskQuestion batches in [`../conductor-loop.md`](../conductor-loop.md).

## After LFP

Attach LOCKED plan + LFG prompt from [`../../../references/lfg-prompt.md`](../../../references/lfg-prompt.md).
