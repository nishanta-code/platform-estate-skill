# Financial Literacy Platform

An AI-native financial-education platform for kids and teenagers: a flagship fellowship, a
business/finance competition, assessments, a curriculum, and a large SEO/content library
(programmatic landing pages for "best financial literacy programs," "how to write a stock
pitch," etc).

## Identity

- Framework: Vite + React, bun (+ a stray package-lock).
- Supabase: **its own small project for auth only** (`<FINEDU_REF>`); real data lives on the
  estate's `finedu` schema, reached from edge functions with a service-role client — the
  "own-project auth split" pattern.
- Scale: ~664 commits since October, ~97 migrations, ~30 edge functions.

## What it owns

- `finedu.app_settings`, `assessment_questions`, `ai_agent_logs`, `concierge_leads`,
  `email_*` (templates, send log, send state, attachments, unsubscribe tokens, suppressed
  emails), `page_events`/`page_visits` (its own lightweight analytics), `program_email_settings`,
  `school_proposals`, `whatsapp_messages`, `program_map`, `catalogue_audit`.
- A **Flywire** payment path alongside the estate's usual Stripe/Razorpay:
  `flywire_webhook_events`, `unmatched_payments`, `payment_reminder_log`.
- **Vouchers**: `voucher_redemptions` — a coupon mechanism distinct from the Learner Portal's
  Vault, scoped to this platform only.

## Checkout history (a cautionary example)

Confirmed by the 2026-11-05 cross-platform pricing audit to be the most severe early-bird bug
in the estate: the checkout function **never fetched `early_bird_price` at all** and always
charged the sticker price, silently overcharging every early-bird buyer until fixed. The fix
ported the estate reference rule (resolve early-bird with a guard against sticker, add a
null-unless-real-discount sticker field for the strike-through UI, and make the strike fire on
early-bird alone, not only when a promo is also applied). Worth remembering: a checkout bug
that **always works** (never errors, just charges the wrong amount) is exactly the kind that
silent testing misses — it needs a line-by-line audit against the reference rule, not just a
happy-path smoke test.

## Content strategy

A large library of standalone programmatic SEO pages (`BestFinancialLiteracyProgramsForTeens`,
`InvestingForTeensGuide`, `HowToWriteAStockPitch`, etc.), each a dedicated route and matching
static `.html` export, alongside the app's normal product pages — a two-pronged strategy of a
real product plus a content funnel feeding it.
