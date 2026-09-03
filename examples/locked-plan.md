# Example — Feature flag for beta users (LOCKED)

**Status:** LOCKED  
**Date:** 2026-09-02  
**Project:** `example-saas/`  
**LFG tier:** M

> Synthetic example showing LFP output shape. An execution agent should run `/lfg` from this file alone.

---

## 1. Goal

Add a `beta_features` flag on the user model so support can enable beta UI for selected accounts without a deploy. Success: admin toggles flag → user sees beta nav on next request.

## 2. Decisions made — do not relitigate

- Store flag on `users.beta_enabled` boolean, default `false`
- Admin-only toggle via existing `/admin/users/:id` — no new admin surface
- No gradual rollout percentage — boolean per user only
- User said: "Don't touch billing or auth middleware"

## 3. Rejected alternatives

| Route | Why not |
|-------|---------|
| LaunchDarkly | New dependency; user declined |
| JSON `features` column | Overkill for one flag |

**Alternatives considered:** none else worth pursuing.

## 4. Repo state / do-not-touch

- In-flight: `feat/checkout-v2` branch — do not merge or rebase
- Frozen: `@company/ui-kit` — report gaps upstream

## 5. Research findings — don't redo this

- `users` table: `supabase/migrations/001_users.sql`
- Admin guard: `api/_shared.js` `requireAdmin()`
- Nav beta link gated in `src/components/Nav.jsx:42` — add `{user.beta_enabled && …}`

## 6. Implementation order

### Phase 1 — Schema
- [ ] Migration: `users.beta_enabled boolean default false`
- [ ] RLS: admin update only on this column

### Phase 2 — API + UI
- [ ] PATCH `/admin/users/:id` accept `beta_enabled`
- [ ] Nav conditional render

## 7. Technical decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Storage | Postgres column | Matches existing user fields |
| Default | false | Safe rollout |

## 8. Tradeoffs accepted

| Optimizing for | Sacrificing |
|----------------|-------------|
| Simplicity | Multi-flag framework |

## 9. Open questions

| Question | Tag | Resolution |
|----------|-----|------------|
| Cache invalidation on toggle? | resolved | Session refresh on next request — no cache layer today |

## 10. Scope

**In:** migration, admin PATCH, nav gate  
**Out:** percentage rollout, user self-opt-in  
**Add-ons included:** Docs — one line in admin runbook

## 11. Definition of done

- [ ] Admin can toggle; non-admin cannot
- [ ] Beta nav visible only when flag true
- [ ] `npm run build` && `npm test` green

**Smoke path:** Admin toggles test user → log in as user → see beta link → toggle off → link gone.

## 12. LFG Handover

**Plan path:** `example-saas/plans/2026-09-02-beta-flag.md`  
**LFG tier:** M  

### In scope
- Migration, admin PATCH, Nav gate, runbook line

### Out of scope
- LaunchDarkly, billing, auth middleware

### Acceptance criteria
- [ ] Admin toggle works; RLS enforced
- [ ] Nav conditional correct
- [ ] build + test green

### Smoke path
Admin → toggle user → login as user → verify nav → toggle off → verify hidden

### Testing instructions
- build: `npm run build`
- test: `npm test`
- e2e: none — manual smoke above

### Technical decisions
- `users.beta_enabled boolean default false`
- Admin-only PATCH

### Constraints (verbatim)
> "Don't touch billing or auth middleware"

### Do-not-touch
- `feat/checkout-v2` branch

### Patterns to copy
| Pattern | Path |
|---------|------|
| Admin guard | `api/_shared.js` |
| User PATCH | `api/admin/users.js` |

### Hard rules from AGENTS.md
- No new dependencies without approval

### Dependency approvals needed
- none

### Paste to execute
Attach this plan and invoke **/lfg**.
