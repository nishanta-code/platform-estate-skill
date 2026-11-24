# Pre-Med Platform

A university-mentor-led pre-medicine programme site: MMI (multiple mini interview) coaching,
a pre-med fellowship and club, a flagship competition, university application tools, and an
AI concierge chat.

## Identity

- Framework: Vite + React, bun; branch `master` (not `main`) — a naming quirk worth knowing
  before scripting across repos.
- Supabase: estate project, schema `premed` (the largest sibling schema besides the Learner
  Portal and the Application Portal — ~45 tables).
- Scale: ~2,900 commits since mid-year, ~219 migrations, ~39 edge functions — the second-most
  actively developed sibling platform.

## Distinctive surfaces

- **MMI AI coach**: practice interview sessions and attempt history
  (`mmi_coach_sessions`, `mmi_practice_attempts`), documented separately in the repo's own
  `docs/mmi-ai-coach/`.
- **University tools**: a `universities` catalogue, `saved_universities`,
  `student_university_applications`.
- **Concierge AI chat**: rate-limited (`concierge_rate_limits`, `concierge_ai_usage`),
  logs conversations (`concierge_conversations`), and promotes its most-asked answers into a
  public FAQ only after admin review (`concierge_faqs` — "raw chat answers are never shown
  publicly").
- **AEO (answer-engine optimisation)**: a full pipeline of its own —
  `aeo_settings`, `aeo_pages`, `aeo_page_optimizations`, `aeo_prompts`, `aeo_keywords`,
  `aeo_outputs` — tuning pages for AI answer engines, not just search engines.
- **Fellowship/club/competition pages**: a quiz embed, project consulting and showcase pages,
  a "coming soon" camp page, masterclass series, mentor and student portals.
- **Payment reconciliation**: `stripe_charge_cache` + `stripe_reconcile_state` — a cache layer
  in front of Stripe plus explicit reconciliation state, rather than trusting the webhook alone.
- **Side-effects queue**: `side_effects_queue` — a logged, retryable replacement for
  fire-and-forget sync, one of four platforms that built this pattern after the estate-wide
  "verify the remote write" lesson.

## Checkout history

Early-bird pricing was originally gated by a hardcoded date constant for its flagship
competition; removed to be purely database-driven, matching the estate reference rule (early
price wins while populated and greater than zero, no expiry date in code). That change is a
genuine behaviour change: the discount no longer auto-expires, so an admin must clear the
catalogue's `early_bird_price` field by hand when the offer should end.

## Registration hardening

`premed-student-register` replaced a bare client-side `supabase.auth.signUp()` — confirmed
dead code at the time it was replaced (no page called it, zero rows in the profile table), so
the change was zero-risk pre-launch hardening rather than a live cutover. The existing admin
user-creation function was aligned to the same lookup-before-create pattern in the same pass.
