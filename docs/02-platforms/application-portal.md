# Application Portal

The central application + checkout site. The reference implementation every other platform's
`/pay` page is a copy of. See `docs/01-backend/payments-checkout-and-promo.md` for the full
checkout contract (that whole document is built from this platform).

## Identity

- Framework: Vite + React Router, bun.
- Supabase: estate project, schema `portal` for its own tables, `public` for the catalogue and
  the central `applications`/`promo_codes`/`promo_redemptions`/`stripe_payments` tables.
- Scale: ~10,300 commits, ~167 migrations, ~73 edge functions — the largest and oldest repo in
  the estate besides the Learner Portal.
- Domain: `apply.example-group.com`.

## What it owns

- **Applications** (`public.applications`, moved from `portal.applications` on a dated
  migration that left a `security_invoker` compatibility view behind — see the payments doc
  §10 for the exact pattern).
- **Form Builder**: a drag-and-drop 3-panel builder (`FormBuilder.tsx`, ~4,600 lines, the
  biggest file in the app) with a deliberate mobile fallback — `Sheet` drawers replace side
  panels and tap-to-move buttons replace drag-and-drop below the `lg` breakpoint.
- **Checkout**: `/apply/{pay_slug}`, `/pay/{pay_slug}`, `/pay?token=` — both directions
  (apply-then-pay and pay-then-apply) work from the same catalogue entry.
- **Admin portal**: ~31 admin pages behind `portal.user_profiles` roles — applications, forms,
  catalogue, promo codes, payment links, installments, media consent/signatures, WhatsApp
  broadcasts, school stats, automated emails.
- **Installments**: a 20-day cadence ladder with optional autopay consent.
- **Media consent**: a signature-pad component for parental consent capture.

## Design system

`hidden md:block` table + `md:hidden` card-list pairing used consistently across every
data-table page. The base dialog primitive bakes in
`w-[calc(100vw-1.5rem)] max-w-lg max-h-[85vh] overflow-y-auto` plus a 44px mobile close button
for every dialog in the app by default. Checkout pages have a sticky mobile-only CTA bar with
safe-area insets, iOS-zoom-prevention (16px input font) and scroll-into-view on panel unlock.

## Notable engineering

- `_shared/attributionSeparation.ts` treats the application's acquisition touch and the
  payment-click touch as two separate, frozen pieces of evidence — never blended.
- `_shared/centralPromo.ts` handles multi-programme promo "families" (`SAVE15`, `SAVE15-2`)
  because the code column is globally unique.
- The money guard in `create-payment-intent` is one line that matters more than the rest of
  the file: `amount > 0 && amount <= baseAmount`, computed entirely server-side.
- `reconcile()` in the payment webhook deliberately does **not** auto-match an unattributed
  payment to anything — it logs for manual review only, a direct response to a sibling
  platform's earlier misattribution incident.

## Audit discipline

A full responsive audit (51 files, every admin page, every dialog) found the codebase already
excellently built — six real gaps, all layout-only. A second sweep after a user-reported bug
found four more instances of the same bug class (`flex-1` search input missing `min-w-0`),
confirming the lesson: a page-level audit list built from the route table can still miss a
component that isn't named in it; cross-check against the actual file listing.
