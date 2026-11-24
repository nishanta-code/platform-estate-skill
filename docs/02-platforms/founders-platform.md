# Founders Platform

A YPO/EO-style institutional membership organisation for teenage founders — explicitly modeled
on how adult peer organisations (Young Presidents' Organization, Entrepreneurs' Organization)
structure chapters and membership, not invented from scratch. Runs regional chapters/forums, a
blog, a founder-identity document product, and an AI Q&A feature.

## Identity

- Framework: Vite + React, bun (+ stray package-lock); includes **Remotion** (programmatic
  video rendering) — the only sibling platform doing this, likely for generated
  founder-identity or chapter-recap videos.
- Supabase: own small project for auth; real data on the estate's `founders` schema.
- Scale: ~617 commits since mid-year, ~81 migrations, ~19 edge functions.

## What it owns

Chapters (`Chapters.tsx`), regional programme variants (named school/region pages e.g.
`YCYWQingdao`, `YCYWYantai`, `BateenWorldAcademy`), a blog (`Blog`, `BlogPost`), `AskAI`
(a Q&A assistant distinct from the sales pipeline's meeting-intelligence AI — this one is
public-facing, member/visitor-oriented), `PitchPrompts`, `Resources`, `Online6Week` (a
structured online cohort format), `admin`, `apply`, `camps`, `marketing`, `pay`,
`presentations`, `schools` route groups.

## Checkout history (the misattribution case study)

This platform is the estate's canonical cautionary example on payment attribution: its earlier
loose payment-matching logic (matching on amount and rough timing rather than the mandatory
programme-code metadata) misattributed nine real payments and sent a pending-registration
email to people who were not actually registering for this platform's programme. The rule
this produced, now applied estate-wide: **route every webhook on the programme code first**,
and when loose matching has already caused a real misattribution, **do not fall back to it at
all** — unattributed payments go to a human review queue instead, never an automatic email.
