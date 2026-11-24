# Pre-Law Platform

A pre-law summer programme site built around moot court and mock trial, with a deliberately
dramatic "Supreme Court" visual identity (framer-motion-driven), a competition track, and a
blog of SEO content ("what is mock trial," "how to write a legal case brief," "is my teen
ready for a pre-law program").

## Identity

- Framework: **TanStack Start** (file routes under `src/routes/`, `*.server.ts` server
  functions) — the one sibling programme site that is not a classic Lovable Vite+React app,
  alongside Future Career/Challenge Platform and Builders Lab.
- Supabase: **its own small project for auth only**; real data on the estate's `prelaw`
  schema.
- Scale: ~1,926 commits, ~37 migrations, only **3 edge functions** — because pricing, promo and
  checkout logic live in `src/lib/*.server.ts` TanStack server functions instead of Supabase
  edge functions. Any fix to this platform's checkout must follow that shape; copying
  edge-function code from a sibling will not compile here.

## What it owns

`prelaw.email_templates`, `email_logs`, `whatsapp_messages`, `whatsapp_settings`,
`unmatched_payments`, `user_roles`, `promo_slot_claims` (slot-limited promo codes — a
constraint not modelled in the Application Portal's own promo table, built as an addition on
top of it).

## Routes

`_authenticated/` (admin), `apply`, `competition` + `competition/apply`, `masterclass` (+
`apply`), `moot-court`, `about` (+ a team sub-page), a dozen blog posts, and the admin auth
route.

## Checkout history

Confirmed in the 2026-11-05 pricing audit to have the same severe early-bird bug as the
Financial Literacy Platform (never fetched the discount, always charged sticker) — but because
the logic lives in `.server.ts` functions rather than edge functions, the fix had to be written
natively for that shape rather than ported verbatim. The fixed checkout page now shows a
dedicated early-bird strike row distinct from (and never shown simultaneously with) the
existing promo-discount strike row.
