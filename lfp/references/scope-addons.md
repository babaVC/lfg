# Scope add-ons menu

Surface in Phase 3 draft (§10). User picks IN/OUT in Phase 4 scope interview.

**Environment-agnostic:** discover project-specific skills from `.cursor/skills/`, `AGENTS.md`, and available agent skills at runtime. Never hardcode org-specific skill names in the skill itself.

## Generic add-on categories

| Category | Examples | When to suggest |
|----------|----------|-----------------|
| **Docs** | README, AGENTS.md, specs/, runbooks | Touches architecture, APIs, onboarding |
| **Marketing / LP** | Landing copy, hero, meta/OG | User-facing surface, launch |
| **Social / comms** | LinkedIn post, changelog, email draft | Shipped feature with external audience |
| **Customer comms** | In-app notice, support macro, status page | Breaking change, billing, downtime |
| **Eval / QA** | Multi-model review, contract tests | High-risk or editorial surfaces |
| **E2E harness** | Playwright spec, smoke script | New user flow, wizard, checkout |
| **Migration / rollout** | Backfill script, feature flag, rollback plan | Schema, auth, billing |
| **Observability** | Logging, alerts, dashboards | Production infra change |
| **Legal / compliance** | Privacy copy, ToS delta | PII, payments, EU users |

## Phase 3 presentation

List applicable add-ons as a menu:

```
Add-ons (pick in scope interview):
- [ ] Docs — …
- [ ] …
```

Default: **unchecked** until user confirms in Phase 4.

## Phase 4 AskQuestion

"Which add-ons are IN for this execution?" — multiple choice from menu + "none" + "defer all to follow-up."

Selected add-ons move to §10 **Add-ons included this pass** and become acceptance criteria in §11.
