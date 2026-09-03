# Platform adapters

LFG is an **agent execution protocol** — waves, reviewers, gates, proud bar. The protocol is tool-agnostic. Adapters map Wave 3 dispatch to whatever your host supports.

| Adapter | Host | Reviewer dispatch |
|---|---|---|
| [cursor.md](cursor.md) | Cursor Agent | Task subagents, one turn, parallel |
| [claude-code.md](claude-code.md) | Claude Code | `.claude/agents/` subagents or parallel sessions |
| [generic.md](generic.md) | Any agent | Paste reviewer prompts into separate chats / sequential if no subagents |

**Before Wave 3:** read your adapter. Set `{REVIEWER_MODEL}` and `{DISPATCH_MODE}` in `lfg-prompt.md` if overriding defaults.

The canonical execution prompt lives in [`../lfg-prompt.md`](../lfg-prompt.md) — works everywhere; adapters only replace the plumbing section.
