# Test-Prep Platform

An SAT coaching platform with a drill engine, trap-pattern teaching, mentor-led guidance,
school partnerships, and a scoring-potential tool for students and parents.

## Identity

- Framework: Vite + React, bun (+ stray package-lock).
- Supabase: own small project for auth; real data on the estate's `testprep` schema.
- Scale: ~1,543 commits, ~84 migrations, ~17 edge functions.

## What it owns

`testprep.sat_questions`, `questions`, `drill_sessions`, `drill_responses`,
`user_trap_stats` (per-student tracking of which "trap" question patterns they fall for —
the pedagogical core of the brand), `homework_submissions`, `student_potential_snapshots`
(point-in-time estimated score potential, presumably used to motivate continued enrollment),
`leads`, `lead_events`, `session_analytics`, `captured_leads`, `sent_emails`,
`whatsapp_sends`, `school_inquiries`, `user_roles`, `country_dial_codes`.

## Pages

Public: `Landing`, `Curriculum`, `FreeResources`, `PsatPrep`, `SatTrapTypes`, `Traps`,
`Schools`, `MentorGuide`, `FAQ`, `Join`, `Apply` (+ `ApplySuccess`), `Masterclass`. Authed:
`Auth`, `Dashboard`, `Drill`, a calculator tool, plus `admin`, `mentor`, `student` role
sections, and a public masterclass registration path at `public.testprep_masterclass_regs`.

## Operational note

The estate master architecture document flags this platform's schema as "built early by its
own agent — verify RLS policies before any app cutover that leans on it." Treat its RLS as
unverified rather than trusted-by-default until it has been checked directly against
`pg_policies`, unlike platforms whose policies have already been spot-checked.
