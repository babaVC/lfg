# LFP — Planning Protocol

**Let's Fucking Plan.** Structured planning for coding agents — before `/lfg`.

Part of [babaVC/lfg](https://github.com/babaVC/lfg). Install the repo once; get both skills.

## Install

```bash
npx skills add babaVC/lfg
# LFP skill path: lfp/ inside the repo

# Or manual:
git clone https://github.com/babaVC/lfg.git ~/.cursor/skills/lfg-repo
ln -sf ~/.cursor/skills/lfg-repo/lfp ~/.cursor/skills/lfp
ln -sf ~/.cursor/skills/lfg-repo ~/.cursor/skills/lfg
```

## Trigger

```
/lfp
```

Or say **lfp** / **LFP** / **let's fucking plan**.

## Flow

```
Brief → /lfp → LOCKED plan → /lfg → ship
```

See root [README](../README.md) for the full Plan & Go methodology.

## What you get

| File | Purpose |
|------|---------|
| `SKILL.md` | Cursor skill entry |
| `references/lfp-prompt.md` | Universal planning prompt |
| `references/conductor-loop.md` | Planning orchestration loop |
| `references/plan-template.md` | LOCKED artifact template |
| `references/lfg-handover.md` | Required §12 for `/lfg` pickup |

## License

MIT — see [LICENSE](../LICENSE).
