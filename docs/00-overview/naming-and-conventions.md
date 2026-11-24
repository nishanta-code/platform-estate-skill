# How this skill names things

This skill is written from a real, running production estate: a fleet of
programme/education platforms built over about a year on **Supabase + Lovable + React**,
sharing one Postgres database, one application portal and one learner portal.

Everything technical here is real — schemas, functions, triggers, flows, bugs, fixes and
numbers. Only the identifying labels are neutralised, so the skill reads as a reusable
architecture rather than one company's internal documentation.

## Substitutions used throughout

| Real-world thing | What this skill calls it |
|---|---|
| The operating company / brand | **the Group**, or just "the estate" |
| The group marketing site | `example-group.com` |
| The central application + checkout site | `apply.example-group.com` |
| A programme brand's own site | `<platform>.example.com` |
| Supabase project refs | `<ESTATE_REF>`, `<DATA_REF>`, `<BRAIN_REF>`, `<platform>_REF` (real refs are 20-char slugs) |
| Platform repositories | role names: Application Portal, Learner Portal, Pre-Med Platform, … (see the estate map) |
| Per-platform Postgres schemas | `portal`, `lms`, `premed`, `prelaw`, `finedu`, `research`, `testprep`, `fellowship`, `founders`, `purpose`, `ops`, `affiliate`, `finance`, `sales`, `leaders` |
| Central transactional tables | `public.applications`, `public.form_responses`, `public.masterclass_regs`, `public.<platform>_masterclass_regs`, `public.stripe_payments`, `public.razorpay_payments`, `public.promo_codes`, `public.promo_redemptions` |
| Brain RPC prefix | `brain_api_*` |
| People | omitted entirely |

Repo-relative file paths are kept exactly as they are in the real repos, with the repo folder
renamed to its role name in kebab case, e.g.
`application-portal/supabase/functions/_shared/createPaymentIntent.ts`.

## Vocabulary

- **Estate** — the shared Supabase project that many platforms live in, one schema each.
- **Platform** — one product/brand: its own repo, its own site, its own schema.
- **Catalogue** — the shared `public` tables describing partners, programmes, cohorts,
  mentors and masterclasses. Read by everyone, written by one console.
- **Own-project auth split** — a platform that authenticates users on its *own* small Supabase
  project while its server code reads and writes its schema on the estate with a service-role
  key.
- **Console** — the internal finance/admin app that is the single writer of the catalogue and
  the only minter of programme codes.
- **Portal** — the central application + checkout site.
- **Learner Portal / LMS** — the shared post-purchase learning platform.
- **Brain** — the research-to-revenue data platform, reachable only through
  `SECURITY DEFINER` RPCs.

## What is deliberately not here

No credentials, tokens, API keys, webhook secrets or connection strings — only the *names* of
environment variables and secrets. No customer, student, parent or staff data. No real domain
names, project refs, repository URLs or person names.
