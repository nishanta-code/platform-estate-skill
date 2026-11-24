# Fellowship Platform

A university-mentor-led fellowship for grades 5–9, run across many regional variants: by
country/region (US, Brazil, China, India, Indonesia, Middle East, Asia, Africa), by school
partnership (dozens of named partner schools, each with its own application, landing and
sometimes showcase page), and by pathway (STEM, entrepreneurship, leadership, communication,
sustainability, creative writing, test prep). Also runs a "Week Without Walls" variant and a
young-debaters/young-entrepreneurs track.

## Identity

- Framework: Vite + React, bun (+ stray package-lock).
- Supabase: **its own small project for auth only**; real data lives on the estate's
  `fellowship` schema.
- Scale: ~4,520 commits since mid-year — the single largest commit count in the estate besides
  the Application Portal, driven almost entirely by the volume of per-partner-school landing
  and application pages (well over 150 page components).

## What it owns

`fellowship.program_map`, `cohort_settings`, `email_templates`, `interest_signups`,
`program_attachments`, `program_email_settings`, `program_entry_tiers` (differentiated
pricing/entry levels per cohort), `program_feedback` (+ `program_feedback_mentors`),
`sent_emails`, `whatsapp_send_log`, `whatsapp_templates`, `profiles`, `user_roles`,
`edge_rate_limits`, `promo_slot_claims`, `unmatched_payments`.

## A custom DB login role

Unusually for a sibling platform, this one's own site writes to the central catalogue through
a **dedicated, non-standard login role** (not the usual service-role client) with its own
targeted RLS policies on the shared masterclass-registration and mentor tables — a distinct
access pattern from every other platform's "own small project + estate service-role client"
split, worth checking before assuming the standard pattern applies everywhere.

## Checkout history

Already fully correct in the 2026-11-05 cross-platform pricing audit: a dedicated, **unit-tested**
pricing module with proper null-when-no-discount handling and correct dual-branch
strike-through logic. This is the one sibling implementation worth copying wholesale rather
than patching — see `docs/01-backend/payments-checkout-and-promo.md` §11.

A separate same-day sweep found one narrow bug outside the tested module: a summary tile
elsewhere on the page ignored the countdown deadline and kept showing the early-bird price
after it should have expired, while a sibling component on the same page handled the same
condition correctly — fixed by porting that sibling's exact condition rather than inventing a
new one.

## Design and structure reference

This platform's own structure/layout and content patterns have been referenced when building
sibling internal tools; conversely, when building new surfaces here, the estate's established
reference points are the Learner Portal's app shell and the central marketing site's brand
tokens — never invent a third visual language.
