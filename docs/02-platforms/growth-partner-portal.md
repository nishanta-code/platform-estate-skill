# Growth Partner Portal (Affiliate)

A partner programme platform serving three partner types: student referrals (already live on
the Learner Portal), affiliate partners (counsellors/creators with unlimited-use promo codes
and payment links), and school ambassadors. Full spec and a reference mockup predate the real
build; both were treated as illustrative, not approved requirements, until independently
verified against the running code.

## Identity

- Framework: Vite + React, bun.
- Supabase: estate project, schema `affiliate`.
- Scale: ~1,110 commits, only 6 migrations but **16 edge functions** — a schema kept
  intentionally small while the business logic (approval, commission, attribution) lives in
  functions rather than extra tables.
- This repository's own `docs/` folder holds the two **estate master architecture documents**
  that every other platform's auth/catalogue/payments rules are drawn from in this skill — see
  `docs/01-backend/research-attribution-and-rpc-gate.md` for how that pattern of a
  centrally-documented, widely-read architecture doc works in practice.

## What it owns

`affiliate.partners` (auth linked to the shared estate's own `auth.users` — identity lives
centrally, partner application data lives here), `promo_code_links` and `payment_link_links`
(join tables recording which partner created which row in the **central** `public.promo_codes`
/ `portal.payment_links` tables — the authoritative pricing/discount record lives centrally so
checkout actually sees it; this schema only tracks ownership), `admins`, `schools`, `payouts`,
`school_achievements`, `announcements`, `resources`, `registration_attempts`.

## The cross-schema write exception

Writing real rows into the central catalogue tables from a partner-facing schema is normally
forbidden (see the ten rules in `SKILL.md`). This platform is the one deliberate, narrow,
explicitly-approved exception in the whole estate: partner-created promo codes and payment
links must exist in the central tables or the checkout flow will never validate them. The
exception required explicit sign-off before the first migration that wrote outside this
platform's own schema, and is documented in the repo's own `AGENTS.md` so it is never mistaken
for a general license to write centrally.

## Known gaps (confirmed by direct code read, not assumed from the spec)

- **No price floor** on payment links — only `amount > 0` is enforced, despite a spec claim
  that links are "never below the floor we set." If a real floor is needed it has to be built
  fresh; no ready template exists for this specific guard.
- **No partner-owner column** on the central promo/payment-link tables. Attribution today is
  `created_by_platform`/`created_by_email` plus a free-text note — this exactly matches what
  the original spec itself listed as a known gap, which is a useful signal that the spec was
  written with accurate awareness of the system rather than against a misunderstanding of it.

## Visual design

Homepage built as a section-by-section, movie-like 3D experience (React Three Fiber / drei): a
WebGL constellation hero, a scroll-pinned shader-driven "journey" curve, and lit 3D gem cards
per partner track, with a `prefers-reduced-motion` 2D fallback for the accessibility-only
subset of visitors with that OS setting on — the full 3D build is what ships to everyone else
by default, including a reviewer checking a preview link. Every animated section was verified
headlessly (desktop + mobile, motion + reduced-motion, zero console errors, zero horizontal
overflow, WebGL context confirmed) before being pushed.

## Registration hardening

`affiliate-register-partner` and `affiliate-admin-signup` replaced direct client-side
`supabase.auth.signUp()` calls for partner self-signup and admin password creation
respectively. The admin path additionally requires the email to already exist as a
pre-provisioned admin row — closing a gap where any stranger could otherwise mint a shared
identity, even though admin *access* itself was already checked correctly elsewhere.
