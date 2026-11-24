---
name: platform-estate
description: Build and operate a fleet of production platforms on one Supabase/Postgres estate — schema-per-platform with a shared read-only catalogue, central application portal and checkout (Stripe/Razorpay/Flywire, promo codes, payment links, installments), a shared learner portal (enrollment, credits/vault, referrals, certificates), internal sales/meeting-intelligence and research-to-revenue attribution pipelines, plus the auth/RLS/grants conventions, edge-function skeletons, React/TanStack frontend shells and design systems, and the playbooks and real incident log behind them. Use when working on any multi-tenant Supabase estate, a schema-per-brand architecture, a central checkout or application portal, an LMS/enrollment engine, promo codes and payment links, server-side registration hardening, RLS/grant debugging, Lovable-built repos, or cross-repo work where several AI agents share one database.
---

# Platform estate: one Postgres, many platforms

This skill is the distilled build knowledge of a real production estate: ~18 repositories,
four Supabase projects, ~37,000 commits, ~1,400 SQL migrations, ~480 edge functions, ten
consumer-facing programme platforms, one application portal, one learner portal, an internal
sales platform and a research-to-revenue data platform — built over about a year with
Lovable + Supabase + React.

Names and identifiers are neutralised (see `docs/00-overview/naming-and-conventions.md`);
every pattern, schema shape, function, bug and fix is real and reusable.

## Start here

1. `docs/00-overview/estate-map.md` — every platform, project, schema, domain and flow.
2. `docs/00-overview/naming-and-conventions.md` — how things are named in this skill.
3. Then route by question:

| Question | Read |
|---|---|
| How is the whole thing wired? | `docs/00-overview/estate-map.md` |
| What happened over the year, month by month? | `docs/00-overview/timeline.md` |
| Shared catalogue, programme codes, slugs, URL contract | `docs/01-backend/catalogue-program-codes-and-slugs.md` |
| Checkout, Stripe/Razorpay/Flywire, promo codes, payment links, installments, webhooks | `docs/01-backend/payments-checkout-and-promo.md` |
| Logins, roles, registration hardening, sessions | `docs/01-backend/auth-and-identity.md` |
| RLS, grants, advisors, secrets, the security posture | `docs/01-backend/rls-security-and-grants.md` |
| Enrollment engine, credits/vault, referrals, certificates | `docs/01-backend/learner-portal-engines.md` |
| Meetings → AI analysis → follow-ups → WhatsApp agent → reports | `docs/01-backend/meeting-intelligence-engine.md` |
| Research → campaigns → tracking links → attribution → revenue | `docs/01-backend/research-attribution-and-rpc-gate.md` |
| How a specific platform is built | `docs/02-platforms/<platform>.md` |
| Stacks, routing, design tokens, app shells, responsive rules | `docs/03-frontend/` |
| Add a platform / write a migration / write an edge function / coordinate repos / verify | `docs/04-playbooks/` |
| What has gone wrong and what rule came from it | `docs/05-lessons/gotchas-and-incidents.md` |
| Live schema metadata (tables, columns, policies, functions, cron) | `reference/sql/` |

## The architecture in one page

- **One Supabase project is the estate.** One `auth.users`. One schema per platform. A shared
  catalogue in `public` that everyone reads and exactly one admin console writes.
- **Facts that are true for every platform live once** — partners, programmes, cohorts,
  mentors, masterclasses. Anything belonging to one platform (applications, leads, logs,
  users) lives in that platform's schema and is never merged with another's.
- **Two auth topologies.** Platforms that need shared identity live directly on the estate.
  Marketing platforms that want blast-radius isolation authenticate on their own small project
  and reach their estate schema through service-role edge functions.
- **The portal is the single checkout.** Programme sites either hand off to it or run a copy
  of its checkout against the same central promo and payment-link tables.
- **Everything privileged is an edge function.** Deployed with `verify_jwt=false` and in-code
  auth, in four shapes: bearer+role, anonymous+unguessable token+rate limit, webhook
  signature, cron secret.
- **The database enforces the invariants.** RLS on every table, grants scoped to what the
  policies intend, `SECURITY DEFINER` RPCs where browsers must not touch tables, CHECK
  constraints on every enumeration, append-only audit tables, triggers for guards and
  `updated_at`.

## The ten rules that matter most

1. **One platform, one schema.** Read the shared catalogue; never write it. Report gaps
   instead of filling them.
2. **No global `has_role()`.** Roles are schema-scoped, the function lives in a schema that is
   not exposed through the API, and every function pins `search_path = <schema>, public`.
3. **Identifiers are minted in one place.** Programme codes `CATEGORY-ORG-PROGRAM-DDMMYY` come
   from the console only; `pay_slug` drives both `/apply/{slug}` and `/pay/{slug}`; links are
   built at render time from the catalogue, never hardcoded.
4. **Money is computed server-side**, can never exceed the list price, and never reaches zero
   through a discount. Promo codes are validated centrally; the client never calculates one.
5. **RLS without GRANTs is a silent no-op**, and RLS with zero policies returns empty rows
   rather than an error. Check grants before policy logic.
6. **Never ship a browser-side `signUp()`** for a flow that matters. Register server-side:
   point lookup by email, rate limit, honeypot, neutral errors, rollback on partial failure.
7. **Migration files, chat summaries and docs are not the live system.** Verify with
   `pg_policies`, `pg_trigger`, `has_function_privilege`, the deployed function body, and a
   real login.
8. **Another agent can push to your branch at any moment.** Fetch immediately before every
   merge and push; read every diff.
9. **Additive-only, feature branches, one session serialises schema changes.**
10. **Assessment and migration are separate documents.** Stop and report when asked to assess.

## Layout

```
SKILL.md                     this file
README.md                    overview for a human browsing the repo
docs/00-overview/            estate map, naming, timeline, glossary
docs/01-backend/             catalogue, payments, auth, RLS, the four engines
docs/02-platforms/           one deep document per platform
docs/03-frontend/            stacks, design systems, shells, responsive rules, frontend auth
docs/04-playbooks/           add a platform, migrations, edge functions, cross-repo, AI-builder workflow, verification
docs/05-lessons/             gotchas and the rules they produced
reference/sql/               live Postgres metadata dumps
reference/inventory/         machine-readable inventories
```

## Safety

No credentials, tokens, keys or customer data — secret *names* only. Keep it that way when
extending: copy structure, never rows.
