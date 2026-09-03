# Plan & Go

**Plan with `/lfp`. Ship with `/lfg`.**

`/lfp` turns a brief into a backlog-ready plan. `/lfg` executes it at the proud bar — no 70% ship.

Works on Cursor, Claude Code, or any agent that accepts a prompt.

```bash
npx skills add babaVC/lfg
```

| Skill | Trigger | When |
|-------|---------|------|
| **LFP** | `/lfp` · **lfp** · **let's fucking plan** | Brief needs decisions, research, scope lock |
| **LFG** | `/lfg` · **lfg** · **let's fucking go** | LOCKED plan ready to execute |

```
Brief → /lfp → LOCKED plan → /lfg → proud bar ship
```

## Install

### Cursor

```bash
npx skills add babaVC/lfg
# or manual:
git clone https://github.com/babaVC/lfg.git ~/.cursor/skills/lfg
ln -sf ~/.cursor/skills/lfg/lfp ~/.cursor/skills/lfp
```

Marketplace: search **Plan & Go** or `/add-plugin lfg`.

### Claude Code

```bash
git clone https://github.com/babaVC/lfg.git ~/.claude/skills/lfg
ln -sf ~/.claude/skills/lfg/lfp ~/.claude/skills/lfp
```

### Any other agent

- Planning: [`lfp/references/lfp-prompt.md`](lfp/references/lfp-prompt.md)
- Execution: [`references/lfg-prompt.md`](references/lfg-prompt.md)

## LFP — Let's Fucking Plan

Planning conductor: goal lock → parallel research → option interview → draft plan → scope interview → LOCKED plan with LFG handover.

**Not for:** trivial fixes (≤15 files), greenfield product validation (use new-product), research-only.

See [`lfp/SKILL.md`](lfp/SKILL.md) · [`lfp/references/conductor-loop.md`](lfp/references/conductor-loop.md)

## LFG — Let's Fucking Go

Maximum-quality execution: plan review → builder subagent → VERIFY → scope-based reviewers → FIX loops → GATE → dogfood → proud bar.

**Not for:** quick fixes, 70% ship-it, time-boxed demos.

**Prerequisite:** LOCKED plan — ideally from LFP ([`lfp/references/lfg-handover.md`](lfp/references/lfg-handover.md)).

See [`SKILL.md`](SKILL.md) · [`references/conductor-loop.md`](references/conductor-loop.md)

## How this compares

| | **Plan & Go** | [Superpowers](https://github.com/obra/superpowers) | [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) |
|---|---|---|---|
| Planning | `/lfp` — interviews, research, LOCKED handover | brainstorming → writing-plans | ce-brainstorm → ce-plan |
| Execution | `/lfg` — proud bar, human scope lock | executing-plans, subagent review | `/lfg` — autopilot to green PR |
| Human in loop | Yes — two interview rounds | Yes — skill-by-skill | Minimal — hands-off pipeline |
| Quality bar | No 70% — tier M/L review loops | TDD, systematic debugging | CI repair loop |

## What you get

| Path | Purpose |
|------|---------|
| `lfp/SKILL.md` | Planning skill |
| `lfp/references/lfp-prompt.md` | Universal planning prompt |
| `lfp/references/plan-template.md` | LOCKED artifact template |
| `SKILL.md` | Execution skill |
| `references/lfg-prompt.md` | Universal execution prompt |
| `references/conductor-loop.md` | LFG multi-agent loop |
| `examples/locked-plan.md` | Example LFP output |

## LFG tier

| Tier | When | Loop floor |
|------|------|------------|
| **M** | Feature slice, ~15–50 files (default) | 2 |
| **L** | Auth, billing, MCP, security-critical | 3 |

## License

MIT — see [LICENSE](LICENSE).

## Author

[babaVC](https://babavc.com) · David Babayan ([@davidbbn](https://github.com/davidbbn))
