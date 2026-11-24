# Purpose Platform

"The success playbook for every child" — a self-discovery and project-based success
curriculum delivered in-school and online, with a counselor-facing surface, a project
library/guide/execution/history flow, and a subscription product alongside one-time academy
fees.

## Identity

- Framework: Vite + React, bun (+ stray package-lock).
- Supabase: own small project for auth; real data on the estate's `purpose` schema.
- Scale: ~1,284 commits since late 2025, ~59 migrations, ~26 edge functions.

## What it owns

A ~29-table schema (per the estate master document) covering settings, content and checkout
for both delivery modes (`AcademyInSchool`, `AcademyOnline`) and roles (`Fellow`, `Counselor`).
Pages: `Apply`, `Checkout`, `Pricing`, `Dashboard`, `ProjectLibrary` (+ `ProjectLibraryDetail`,
`ProjectDetail`, `ProjectGuide`, `ProjectExecution`, `ProjectHistory`), `SuccessStories` (+
detail), brand pages (`BrandAssets`, `BrandIdentity`, `LogoOptions`).

## The one platform with a second payment product

Documented explicitly in the estate's payment-routing rules: this is the only sibling site
selling **subscriptions** alongside the usual one-time academy fee, on what may be a Stripe
account shared with other platforms. The rule it is built under: webhook handlers here must
pass non-academy (subscription) events through untouched rather than forcing them into the
one-time-programme code path that every other sibling's webhook assumes.

## Status

As of the last architecture sweep, Stripe wiring for this platform was still pending and a
completed April cohort's applications/payments needed backfilling once supplied — treat its
payment tables as less populated/proven than the platforms with a long checkout history until
re-verified.
