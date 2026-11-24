# Platform Estate — a complete Supabase multi-platform engineering skill

A reusable skill distilled from a year of building and running a real production estate:
**one Postgres database, many independent product platforms**, a central application portal
and checkout, a shared learner portal, and internal sales and data platforms — all built with
Supabase, Lovable and React.

It is written as a skill for coding agents (and engineers) to load and follow. Identifying
names are neutralised; the architecture, schemas, functions, flows, bugs and fixes are real.

## What is in here

- **The architecture** — schema-per-platform on one Supabase project, a shared read-only
  catalogue in `public`, two auth topologies, and the write-ownership rules that let several
  teams and AI agents work on one database without destroying each other's data.
- **The checkout contract** — server-side pricing, early-bird resolution, centrally validated
  promo codes with family/multi-programme handling and double cap checks, custom payment
  links with slug guards and atomic usage caps, a 20-day installment engine with autopay, the
  mandatory PaymentIntent metadata, webhook idempotency, and the "unattributed payments go to
  a human" rule.
- **The engines** — enrollment (paid application → learner account → throttled login rollout),
  credits/vault ledger, referral attribution, meeting intelligence (calendar → video call →
  AI analysis → follow-up drafting → messaging agent → reports), and research-to-revenue
  attribution behind a `SECURITY DEFINER` RPC gate.
- **The security model** — RLS everywhere, grants scoped to policies, schema-scoped roles,
  function EXECUTE lockdown, four edge-function auth shapes, server-side registration
  hardening, and advisor-driven cleanups.
- **The frontend** — two stack shapes (Vite + React Router, TanStack Start), the app shells
  that get copied between products, every design system's tokens, and a hard-won responsive
  checklist.
- **The playbooks and the scars** — how to add a platform, write a migration, write an edge
  function, work across sibling repos while an AI builder commits to the same branch, verify
  that something is actually done, and a catalogue of real incidents with the rule each one
  produced.

## Start here

1. [`SKILL.md`](SKILL.md) — entry point, routing table, the ten rules.
2. [`docs/00-overview/estate-map.md`](docs/00-overview/estate-map.md) — the whole estate on one page.
3. [`docs/00-overview/naming-and-conventions.md`](docs/00-overview/naming-and-conventions.md) — how things are named here.

## Layout

| Path | Contents |
|---|---|
| `docs/00-overview/` | Estate map, naming conventions, year timeline, glossary |
| `docs/01-backend/` | Catalogue contract, payments & promo, auth & identity, RLS & grants, the four engines |
| `docs/02-platforms/` | One deep document per platform |
| `docs/03-frontend/` | Stacks & layout, design systems & tokens, app shells, responsive rules, frontend auth |
| `docs/04-playbooks/` | Add a platform, migrations, edge functions, cross-repo working, AI-builder workflow, verification discipline |
| `docs/05-lessons/` | Gotchas, incidents and the rules they produced |
| `reference/sql/` | Live Postgres metadata: tables, columns, views, functions, triggers, policies, grants, cron, sizes |
| `reference/inventory/` | Machine-readable repo and edge-function inventories |

## Using it as a skill

Clone into a skills directory (for example `.claude/skills/platform-estate/`) so the agent
loads `SKILL.md` and follows its routing table, or read the docs directly.

## Safety

No credentials, tokens, API keys, connection strings or customer data — environment-variable
and secret *names* only, with every real domain, project reference, repository URL and person
name replaced.
