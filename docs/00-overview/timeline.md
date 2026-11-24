# The estate's year, in commits

A fleet this size does not appear at once. This is the shape of how it actually grew — useful
for calibrating how much engineering judgement a given repo's age and commit density implies.

## Master table

| Platform | Commits | Real start | Last commit | Note |
|---|---:|---|---|---|
| Application Portal | ~10,320 | 2025-10-22 | 2026-11-24 | the oldest and largest repo by commit count |
| Learner Portal | ~3,445 | 2026-03-22 (seed 2025-01-01) | 2026-11-24 | most mature engineering (unit-tested state machines) |
| Pre-Med Platform | ~2,900 | 2026-06-01 | 2026-11-23 | second-most actively developed sibling |
| Fellowship Platform | ~4,520 | 2026-06-08 | 2026-11-12 | highest commit count after the Portal, driven by scale of per-partner pages |
| Ops Tool | ~2,615 | 2026-04-08 (seed 2025-01-01) | 2026-11-13 | internal tool; repo still named after its prior identity |
| Sales Command | ~2,990 | 2025-10-16 | 2026-10-15 | oldest start date among sibling platforms besides the Portal |
| Builders Lab | ~2,055 | 2026-08-20 | 2026-11-24 | TanStack Start |
| Pre-Law Platform | ~1,930 | 2026-06-30 (seed 2025-01-01) | 2026-11-12 | TanStack Start, checkout in server functions |
| Test-Prep Platform | ~1,545 | 2025-12-21 (seed 2025-01-01) | 2026-11-12 | |
| Purpose Platform | ~1,285 | 2025-11-28 | 2026-11-12 | one of the oldest sibling-programme starts |
| Growth Partner Portal | ~1,110 | 2026-03-12 (seed 2025-01-01) | 2026-11-24 | small schema, heavy on functions |
| Challenge Platform | ~875 | 2026-08-09 | 2026-11-23 | TanStack Start |
| Financial Literacy Platform | ~665 | 2026-10-06 | 2026-11-24 | newest sibling-programme build |
| Founders Platform | ~615 | 2026-06-01 | 2026-11-12 | includes Remotion video rendering |
| Research Platform | ~500 | 2026-10-05 | 2026-11-16 | SSR + Playwright e2e |
| Research Outreach Platform | 124 | 2026-10-16 | 2026-11-06 | youngest active platform, fast iteration |
| Marketing Site | 25 | 2026-11-06 | 2026-11-13 | a deliberate rebuild, not an organic history |
| Counselling CRM | 27 | 2026-11-18 | 2026-11-23 | the fastest full build-to-live cycle in the estate |

**Total: roughly 37,000 commits across the fleet.** Several repos carry a single template-seed
commit dated `2025-01-01` from the app builder's own initial scaffold — that date is never the
real start of work; always find the first real burst of daily commits before treating a repo's
age as evidence of maturity.

## The arc, in five phases

**Phase 1 — the backbone (late 2025).** The central Application Portal and the internal
Sales Command platform start first, months before any consumer-facing programme site. This
ordering matters: the checkout/application contract and the sales pipeline existed before most
of the brands they would eventually serve did.

**Phase 2 — the first programme sites (late 2025 into Q1 of the following year).** The
earliest consumer platforms (Purpose Platform, Test-Prep Platform) begin, each starting as its
own isolated Supabase project with its own auth — the "own-project-auth split" pattern that
later becomes the estate-wide default for new marketing platforms.

**Phase 3 — the estate consolidation (Q1–Q2).** The Learner Portal, the Ops Tool and the
Growth Partner Portal all begin inside the same few-week window, all pointed at the shared
estate project directly rather than their own project — this is the moment "schema per
platform, one shared `auth.users`" becomes a deliberate, named architecture rather than an
accident of a few platforms happening to share a project.

**Phase 4 — the programme-site expansion (Q2–Q3).** The Pre-Med, Pre-Law, Fellowship and
Founders platforms all start within a few months of each other — the busiest period of new
platform creation in the estate's history, and the period where the catalogue/program-code/
pay-slug contract had to scale from describing two or three programmes to describing dozens.

**Phase 5 — the data and attribution layer, plus rapid recent builds (Q3–Q4, the most recent
few months).** The Research Platform, the Financial Literacy Platform, the Challenge Platform,
Builders Lab and the Research Outreach Platform all start in this window — the estate's
attention visibly shifts from "add another storefront" to "understand and act on the data the
existing storefronts already generate" (attribution, lead scoring, a dedicated research-to-
revenue data platform). The Marketing Site rebuild and the Counselling CRM, the two youngest
and fastest-built repos, both land in the final weeks — evidence that by this point the
patterns documented in this skill were mature enough to execute a full build (spec → schema →
live, reviewed, hardened product) in days rather than months.

## What the shared estate's own migration count implies

The shared estate project accumulated roughly 1,400 SQL migration files across its platform
schemas and the Application Portal/Learner Portal's especially migration-dense histories (167
and 80 respectively) — a visible proxy for how much of this estate's design was **discovered
by building**, not planned up front: the catalogue's own v1→v2 column cleanup, the
applications-table schema relocation, the edge-function namespace migration, and the shared
identity/registration hardening pass were all later corrections to an earlier, reasonable-at-
the-time design, not mistakes. Treat a mature schema's current shape as the result of several
rounds of real-world correction, and expect the next platform built the same way to need the
same kind of correction in its own first few months.
