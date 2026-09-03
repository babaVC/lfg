# Harness Plan integration

LFP builds the plan **as you go** using the host's Plan feature. Chat is not the source of truth.

## Cursor

1. **Phase 0 (goal lock):** call `CreatePlan` with name, overview, and todo phases 0–5 from [`conductor-loop.md`](conductor-loop.md).
2. **After each phase:** edit the plan file directly — integrate research, lock decisions, update scope.
3. **Phase 5 (solidify):** export LOCKED content to project path (see path rule below) in addition to harness plan file.

Path rule for LOCKED export:
- `docs/plans/{YYYY-MM-DD}-{slug}.md` if `docs/plans/` exists
- else `{project}/plans/{YYYY-MM-DD}-{slug}.md`

## Generic (no CreatePlan)

1. Create `{project}/plans/{date}-{slug}.md` at Phase 0 with DRAFT header.
2. Update same file after each phase.
3. Set **Status: LOCKED** at Phase 5.

## Plan file vs project file

| Artifact | Purpose |
|----------|---------|
| Harness plan (Cursor `.cursor/plans/` or equivalent) | Living doc during LFP session |
| Project plan (`docs/plans/` or `plans/`) | Backlog pickup — any agent, any session |

At LOCKED, both should converge — same content, project path is canonical for `/lfg`.

## Do NOT

- Keep draft only in chat
- LOCK without §12 LFG Handover ([`lfg-handover.md`](lfg-handover.md))
- Skip plan file updates between phases
