# Ops Tool

An internal staff tool for AI-driven calling operations: independent calling agents, agent
campaigns, captured business contacts, and staff account access. This is where the owner's
own staff login originates on the shared estate. Its GitHub repository still carries its
previous identity's name (an earlier "Marketplace" product) from before it was repurposed —
worth knowing before assuming the repo name describes what the code actually does today.

## Identity

- Framework: Vite + React, bun (+ stray package-lock); Playwright e2e tests and a
  `central-tests`/`central-migrations` folder pattern distinct from its sibling platforms.
- Supabase: the **shared estate directly** (not the own-project-auth split), schema `ops`.
- Scale: ~2,615 commits, ~114 migrations, only **5 edge functions** — most of the surface area
  is internal admin UI, not public-facing checkout.

## What it owns

`ops.profiles`, `user_roles`, `agent_profiles`, `agent_campaigns`, `agent_campaign_contacts`,
`agent_campaign_calls`, `payment_followup_leads`, `contact_suppressions`.

## Staff provisioning

No public self-signup exists. The one admin edge function (`ops-admin-users`) is the sole way
an account is created — the reference implementation for "staff provisioning only, never
self-service" cited by other platforms' auth hardening. It follows the same lookup-before-create
pattern as every other safe-registration function on the estate: a point lookup by email
through a `SECURITY DEFINER` RPC, never a full `listUsers()` page scan.

## Pages

`Admin`, `AdminAgentCampaigns`, `AdminAgents`, `AdminAuth`, `AdminCapturedContacts`,
`AdminUsers`, a public `Index`/`Privacy`/`Terms`, `ResetPassword` — a small, admin-dense
surface consistent with its internal-tool purpose.
