# Sales Command

The internal sales / meeting-intelligence platform. See
`docs/01-backend/meeting-intelligence-engine.md` for the full pipeline — this page is the repo
profile.

## Identity

- Framework: Vite + React, npm (plus a stray `bun.lockb`) — one of only two repos in the
  estate using npm as its primary package manager.
- Supabase: the repo's own `supabase/config.toml` names one project reference, but the
  **deployed, live backend now runs on the data project's `sales` schema** — the repo includes
  its own `migrate-to-own/` toolkit (data export, auth-user generation, RLS, cron rewiring,
  verification) documenting exactly how that migration was carried out, function by function,
  with each edge function's `verify_jwt` posture read programmatically from config rather than
  re-decided by hand.
- Scale: ~2,989 commits (the oldest start date among the sibling platforms, 2025-10), ~134
  migrations, ~79 edge functions — nearly as many functions as the Application Portal.

## What it is

Turns calendar bookings into revenue: Calendly webhook → Zoom call → AI transcript analysis →
segment-aware lead scoring → commitment extraction → follow-up drafting → human approval or a
WhatsApp agent → Gmail send → reply classification → weekly reports. Full detail in the
meeting-intelligence engine document, including the Email Intelligence Agent SOP (a
classification scheme and scenario playbook built from 27,236 real email interactions).

## Distinctive engineering

- A single knowledge module assembles the organisation's identity, programme names, voice
  rules and scoring rubric fresh on every AI call, so every agent — analysis, scoring,
  drafting — stays provably consistent instead of drifting across dozens of prompt copies.
- Every model output is validated against a closed enum before being trusted or written
  (interest levels, programme names, temperature, segment, goal).
- An AI kill switch is checked first in every AI-facing function, with "skip without touching
  anything, pick it up again later" as the disabled behaviour.
- A claim/assignment layer (`wa-claim-webhook`/`wa-claim-poll`) lets a human staff member claim
  a WhatsApp conversation so a human and an automation never double-message the same lead.
- `core-sync` explicitly mirrors central payments/applications into this platform's own
  read-only view of the funnel — the central tables stay the one authoritative source.
