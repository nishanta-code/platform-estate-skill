# Group Research & Outreach ("Research Brain" / lwl-data-outreach)

> Documented 2026-11-24 from a read-only pass over the local clone, its git history, and two read-only
> catalogue queries against the *other* Group Supabase project (`<DATA_REF>`). The Brain project
> (`<BRAIN_REF>`) was **not** reachable from this session's Supabase MCP, so everything about
> Brain's database is sourced from the repo (code, the one committed migration, commit messages, generated
> types) and is labelled as such. Nothing here is guessed; where something could not be verified it says so.

---

## 1. Identity

| Item | Value |
|---|---|
| Local folder | `D:\the Group\Vs Code Studio\Research Outreach` |
| Git remote | `https://github.com/<org>/<repo>` (origin, fetch+push) |
| Commits | **124** on the checked-out branch, 2026-10-16 (template) -> 2026-11-06 (last) |
| Authors (by commit count) | `gpt-engineer-app[bot]` 98 (Lovable editor), `nishanta-code` 25 (local Claude Code sessions, 11-05/11-06), `Lovable` 1 |
| Lovable project | Yes. `.lovable/project.json` = template `tanstack_start_ts_current`, revision `tanstack_start_ts_current-3341755609d5`. Lovable plan files in `.lovable/plan/`. `AGENTS.md` carries the Lovable "do not rewrite history" banner. |
| Published URL (per README/code) | `https://brain-preview.lovable.app` (login at `/login`, OAuth consent at `/oauth/consent`; hard-coded as `LOGIN_URL` in `src/components/onboarding-guide.tsx`). README mentions a custom-domain intent `research-auth.example-group.com` in the 10-26 plan, **not verified as live**. |
| Page title brand | "LwL Research Brain" (`pageHead()` suffix in `src/lib/route-guard.ts`); sidebar brand "Research Brain"; author meta "Learn With Leaders". |
| Stack | TanStack Start 1.168 + TanStack Router 1.170 (file routes, `ssr: false` on most routes), React 19.2, TanStack Query 5, Tailwind CSS 4 (`@tailwindcss/vite`), shadcn/ui (new-york, Radix), lucide-react, zod 3, sonner, recharts (installed, unused in routes), `react-email-editor` (Unlayer), `@supabase/supabase-js` 2.118. Build via `@lovable.dev/vite-tanstack-config` 2.26.5 (wraps Vite 8 + nitro 3 beta, Cloudflare default target -> `.wrangler/` dir present). |
| Package manager | **bun** (`bun.lock`, `bunfig.toml` with `minimumReleaseAge = 86400` supply-chain guard). `package.json` also has `"overrides": { "rolldown": "1.2.1" }` which npm treats differently from bun (see Gotchas). README still says `npm i`. |
| Test runner | vitest 4 + jsdom + @testing-library/react (11 test files, `bun run test`). |
| Lint/format | eslint 9 flat config + typescript-eslint + prettier; `react-refresh/only-export-components` warn. |
| TypeScript | strict + `exactOptionalPropertyTypes`, `noUncheckedIndexedAccess`, `noPropertyAccessFromIndexSignature` (hence `process.env["..."]` bracket access everywhere). |
| Supabase CLI config | `supabase/config.toml` -> `project_id = "<BRAIN_REF>"` (since 2026-10-29; was `txmlngnuwqqhgdtqjydc` from 10-26 to 10-29). |
| MCP | `.mcp.json` -> Supabase hosted MCP scoped to `project_ref=<BRAIN_REF>` (features: docs, account, database, debugging, development, functions, branching, feedback, storage). No credentials stored. |
| Agent skills | `skills-lock.json` + `.agents/skills/` = `supabase` and `supabase-postgres-best-practices` from `supabase/agent-skills` (installed 2026-11-06, commit e202c40). |

### Environment variable NAMES (values deliberately not reproduced)

| Name | Where read | Purpose |
|---|---|---|
| `LWL_BRAIN_PUBLISHABLE_KEY` | `src/lib/brain-config.functions.ts`, `src/lib/brain.server.ts` (`process.env`) | The **only** runtime secret the app reads. Must be a publishable key; code refuses anything starting `sb_secret_` and reports "Brain configuration unavailable". No service-role key exists or is supported. |
| `SUPABASE_URL`, `SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_PROJECT_ID`, `VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY`, `VITE_SUPABASE_PROJECT_ID` | present in `.env` (Lovable editor binding; point at `<BRAIN_REF>`) | **Not consumed by application code** (AGENTS.md rule 1). `src/integrations/supabase/client.ts` (Lovable-generated) hard-codes the Brain URL + publishable key but nothing in `src/lib` or `src/routes` imports it. |

Three Supabase project refs appear in history:

| Ref | Name (from plans/tests) | Role for this repo |
|---|---|---|
| `txmlngnuwqqhgdtqjydc` | "LwL Master Research" (legacy; `.lovable/plan/...2026-10-26.md`) | Original backend 10-26 -> 10-29. Removed at the "Brain cutover"; `src/test/brain-research.test.ts` asserts no research code references it. |
| `<BRAIN_REF>` | "Brain" | The only runtime backend since 2026-10-29. All auth + all data via `brain_api_*` RPCs. |
| `<DATA_REF>` | "the Group" (org project) | **Zero references anywhere in the repo or its history** (`git log -S` returns nothing). See section 9. |

---

## 2. Business purpose

A staff-only internal portal for Learn With Leaders' school-research and outreach team, evolving in two phases:

1. **Identity & Access Portal (10-26 -> 10-29)** – Brain Auth sign-in, password reset, OAuth 2.1 consent screen so
   ChatGPT Business (the "LwL Intelligence Brain" MCP app at `.../functions/v1/brain-mcp`) can connect staff
   accounts, and an admin "Team Access & Approvals" page that grants `researcher`/`admin` membership in Brain.
2. **Research-to-Revenue Intelligence Dashboard (11-05 -> 11-06)** – read-only research views (schools, contacts,
   research ops, historical coverage) plus a campaign system: create campaigns in Brain *before* sending, map NPF/Meritto
   Job IDs and Instantly campaign IDs to them, generate per-contact three-UTM tracking links, export NPF-ready CSVs,
   upload NPF "Communication Performance" reports, see Instantly webhook event counts, source-segment (UTM medium)
   attribution, email deliverability/suppressions, and a Data Health page that shows where attribution is incomplete.

Design stance repeated throughout the code: **never fabricate counts** – a failed or denied RPC renders
"Not yet available in Brain" (`UnavailableData`) rather than zero.

---

## 3. Frontend

### 3.1 Route table (file-based, `src/routes/`)

| File | URL | Guard | Purpose |
|---|---|---|---|
| `__root.tsx` | – | – | HTML shell: Google Fonts (Sora 500/600/700, Manrope 400/500/600), `styles.css`, favicon `/lwl-bird.png`, `QueryClientProvider`, `Toaster`, 404 + error boundary (reports to Lovable telemetry), subscribes to Brain `onAuthStateChange` to clear/invalidate queries. |
| `index.tsx` | `/` | – | `beforeLoad`: signed-in -> `/dashboard`, else `/login`. |
| `login.tsx` | `/login?next=` | – | Email+password sign-in against Brain (`signInWithPassword`), 5-attempt/60s client lockout, "forgot password" (`resetPasswordForEmail` -> `/reset-password`), post-login routing via `postLoginPath()` (only consent paths are honoured as `next`). |
| `auth.callback.tsx` | `/auth/callback` | – | Handles `?code=` (PKCE `exchangeCodeForSession`) or `token_hash`+`type` (`verifyOtp`); recovery/invite -> `/reset-password`. |
| `reset-password.tsx` | `/reset-password` | – | Waits for `PASSWORD_RECOVERY`/session, then `auth.updateUser({password})` (min 8 chars). |
| `account.tsx` | `/account` | – | Legacy alias -> redirect `/profile`. |
| `access-denied.tsx` | `/access-denied` | – | "Access Not Approved" with sign-out. |
| `oauth.consent.tsx` + `[.]lovable.oauth.consent.tsx` | `/oauth/consent`, `/.lovable/oauth/consent` | – | Same `ConsentPage` component served at both aliases (Supabase OAuth 2.1 server consent). |
| `_app.tsx` | pathless layout | Brain session + approved membership | `beforeLoad`: `supabase.auth.getUser()` -> `/login?next=`; `getMyMembership()` must be `approved` else `/access-denied`; puts `{membership: {role, displayName, email, timezone}}` on context. Renders sidebar + h-12 topbar (title, `LiveClock`, `NotificationBell`, settings icon). |
| `_app.dashboard.tsx` | `/dashboard` | `overview` | Stat tiles (schools, contacts, verified contacts, open research jobs, evidence awaiting review) + for lead/admin: recent campaigns, attribution health, Sources link, 30-day deliverability (delivered/bounced/unsubscribed/active suppressions). |
| `_app.schools.tsx` | `/schools?q&page` | `schools` | Paged (50) school search via `listSchools`. |
| `_app.schools_.$id.tsx` | `/schools/:id` | `schools`+`contacts` | School header + contacts table via `getSchool`. |
| `_app.contacts.tsx` | `/contacts?q&status&page` | `contacts` | Paged contacts with validated/not_validated filter. |
| `_app.research.tsx` | `/research` | `research` | "My saved research" (caller-scoped dashboard), historical school coverage search (25/page), research allocations + claims. |
| `_app.outreach.tsx` | `/outreach` | `outreach` | Tracking events list (`listOutreach`). Hidden from nav. |
| `_app.programs.tsx`, `_app.revenue.tsx` | `/programs`, `/revenue` | `programs`/`revenue` | Explicit "unavailable" stubs; hidden from nav since bc50c72. |
| `_app.campaigns.tsx` | `/campaigns` | `campaigns` | Segment picker: NPF/Meritto card vs Instantly card with live quick stats. |
| `_app.campaigns_.npf.tsx` | `/campaigns/npf?q&page&source` | `campaigns` | NPF campaign list (25/page, source chip filter, tracking-health badge) + **Create Campaign** dialog. |
| `_app.campaigns_.$id.tsx` | `/campaigns/:id?tab=` | `campaigns` | Campaign detail tabs: Overview (KPIs, source mapping, NPF job linking), Email Performance (NPF report upload + recipient table), Contacts & Schools, Tracking Links (generate by Contact IDs / bulk, list, preflight, NPF CSV export, resolver), Instantly (link external campaign, event KPIs), Applications/Revenue (Phase 3/4 placeholders). |
| `_app.campaigns_.instantly.tsx` | `/campaigns/instantly?tab=` | `campaigns` | Tabs: Campaigns (all Instantly campaigns with events, linked or not), Templates (plain-HTML CRUD + link to visual builder), Upload Contacts (CSV -> dedupe -> `checkContactRows` check/commit). |
| `_app.campaigns_.instantly.templates.$id.tsx` | `/campaigns/instantly/templates/:id` (`new` or uuid) | `campaigns` | Unlayer drag-and-drop editor; merge tags limited to `{{name}}` and `{{tracking_url}}`; saves HTML + design JSON. |
| `_app.sources.tsx` | `/sources` | `campaigns` | Per-UTM-medium source segment table (campaigns, reach, delivered/opened/clicked, applications, paid, revenue per currency) + "Not mapped" card. |
| `_app.data-health.tsx` | `/data-health` | `dataHealth` | Verified counts: campaigns, NPF reporting match status, native tracking/conversion, research base, Instantly campaigns not linked. |
| `_app.settings.tsx` | `/settings` | `settings` | Timezone (saved to Brain Auth `user_metadata.timezone`); admin-only Instantly integration card (status, webhook URL, rotate secret, workspace ID, API key). |
| `_app.profile.tsx` | `/profile` | `profile` | Email, role badge, display-name edit (Brain Auth `user_metadata.display_name`), sign out. |
| `_app.admin.members.tsx` | `/admin/members` | `members` | Team Access & Approvals: `OnboardingGuide` (5-step ChatGPT connection wizard, copies MCP URL) + `TeamAccess` (confirm-gated grant/revoke of researcher/admin). |

`src/routeTree.gen.ts` is generated; `src/routes/README.md` documents the file-route conventions.

### 3.2 App shell & components

- `src/components/app-sidebar.tsx` – shadcn `Sidebar collapsible="icon"`, 14rem wide, groups **Workspace** (Overview, Schools, Contacts, Research), **Revenue Intelligence** (Campaigns, Sources, Data Health), **Account** (Team Access / Approvals, Profile). Items filtered by `can(role, section)`. Footer: initials avatar (brand gradient), name, `titleCase(role)`, sign-out.
- `src/components/topbar-widgets.tsx` – `LiveClock` (1s tick, `Intl.DateTimeFormat` in saved timezone, `tabular-nums`), `NotificationBell` (real "needs attention" tray derived from `getDataHealth` + `getInstantlyUnmapped`).
- `src/components/data-view.tsx` – `PageHeader`, `Card` (`.card-surface`), `DataTable<T>` (`Col<T>`), `StatusBadge` (always `titleCase`), `fmtDate`, `Loading`, `LoadError` ("Your role doesn't have access" when message is `Forbidden`), `schoolName`.
- `src/components/list-controls.tsx` – `SearchBox` (submit-on-enter), `Pager`.
- `src/components/portal-shell.tsx` – `BrandMark` (`/public/lwl-bird.png`), `Brand`, `CenteredShell`, `Panel`, `StatusPanel` (auth pages).
- `src/components/consent-page.tsx` – OAuth consent state machine (`loading|missing|not_enabled|pending_config|unapproved|invalid|timeout|redirecting|ready`), uses `supabase.auth.oauth.{getAuthorizationDetails,approveAuthorization,denyAuthorization}` via `src/lib/oauth-client.ts`, re-checks `getMyBrainAccess` before approving, only follows https redirects.
- `src/components/team-access.tsx`, `src/components/onboarding-guide.tsx`, `src/components/unavailable-data.tsx`.
- `src/components/ui/*` – 40+ shadcn primitives (sizes tightened in 3ad8aad to match the LMS Portal density: Button h-8/h-7/h-9, 13px labels, h-12 header).
- `src/hooks/use-mobile.tsx` – 768px breakpoint used by the sidebar sheet.

### 3.3 Data-access pattern (frontend -> server function -> Brain RPC)

```
React component
  └─ useServerFn(fn) + useQuery/useMutation            (TanStack Query, keys like ["campaign-detail", id])
       └─ createServerFn({method:"POST"}).middleware([requireBrainAuth]).inputValidator(zod).handler(...)
            ├─ src/start.ts attachSupabaseAuth (client middleware): adds Authorization: Bearer <Brain access_token>
            ├─ src/lib/brain-auth.ts requireBrainAuth: authenticateBrain(header) -> { db, userId, user, token }
            │     (db = supabase-js client built with the PUBLISHABLE key + the caller's JWT; getUser(token) must succeed)
            ├─ src/lib/brain.server.ts authorizeBrain(context, section): rpc("brain_api_my_roles") -> can(role, section)
            └─ db.rpc("brain_api_<name>", {...}) -> jsonb envelope {status:"ok"|"access_denied"|...}
```

Rules enforced in code and tests:
- **Zero direct table reads**: `grep '\.from(' src` (excluding generated types) returns nothing. `src/integrations/supabase/client.server.ts` exports a `supabaseAdmin` Proxy that throws on any access.
- CSRF middleware re-added explicitly in `src/start.ts` (defining `start.ts` opts out of the default).
- Every list is paged server-side: 50 (schools/contacts), 25 (campaigns/recipients/coverage), 200 (links/contacts of a campaign), 500 (bulk link generation), 2000 contact codes, 25 000 NPF rows, 5 000 contact-check rows.
- Server fns return `{ rows, total, pageSize, available }` or discriminated unions; `access_denied` -> `throw new Error("Forbidden")`; RPC transport error -> `"Brain reporting data not yet available"`.

### 3.4 Auth flow

1. Browser client: `getBrainClient()` (`src/lib/brain-client.ts`) fetches `{url,key}` from the server fn `getBrainConfig` (publishable key only) and creates one supabase-js client with `storageKey: "brain-auth"` (deliberately not the default `sb-<ref>` key so a legacy Master Research session is never mistaken for Brain).
2. `/_app` layout `beforeLoad` requires a Brain user **and** `getMyMembership()` = `approved`.
3. `getMyMembership` -> `brainAccess()` -> `rpc("brain_api_my_roles")` (no args; identity from `auth.uid()`), parsed by `parseBrainRoles`/`decideBrainAccess`: known roles only (`admin`, `lead`, `researcher`), `[]` = not approved, RPC error = `unavailable` (fails closed, distinct message "verification unavailable"). `brainWorkspaceRole` picks the highest role (admin > lead > researcher).
4. Section permissions live once in `src/lib/permissions.ts` (`SECTIONS`) and are enforced twice: `guard(section)` in route `beforeLoad` (redirect to `/dashboard`) and `authorizeBrain(context, section)` in every server fn.
5. Profile name / timezone are Brain Auth `user_metadata` written with a direct `PUT {BRAIN_URL}/auth/v1/user` using the caller's JWT – never a source of roles.
6. Sign-out: `signOutEverywhere()` -> `auth.signOut()` + `location.replace("/login")`; root subscribes to `SIGNED_OUT` to cancel and clear the query cache.
7. OAuth consent for ChatGPT: `/oauth/consent?authorization_id=` validated by `isValidAuthorizationId` (`^[A-Za-z0-9._~-]{8,256}$`), membership re-checked server-side, CSP `frame-ancestors 'none'` + `X-Frame-Options: DENY` only on consent paths (`src/server.ts`). Other pages allow framing from `*.lovable.app`/`*.lovable.dev` for the editor preview.

### 3.5 Design tokens

`src/styles.css` (Tailwind 4 `@theme inline`): dark violet theme ported from "a sibling email SaaS" (commit 30ae201), no light mode (`.dark` mirrors `:root`).

| Token | Value |
|---|---|
| `--font-serif` / `--font-sans` | "Sora" (headings h1–h3) / "Manrope" (body) |
| `--radius` | 0.75rem |
| `--background` / `--card` / `--sidebar` | `hsl(260 20% 7%)` / `hsl(260 18% 11%)` / `hsl(260 22% 5%)` |
| `--primary` / `--accent` | `hsl(270 76% 58%)` / `hsl(280 60% 52%)` |
| `--destructive` / `--success` / `--warning` | `hsl(0 72% 56%)` / `hsl(160 72% 42%)` / `hsl(38 88% 52%)` |
| `--gradient-brand` | 135deg violet -> magenta -> rose (`.gradient-brand`, sidebar active rail, avatar, login hero) |
| `--gradient-canvas` | three radial glows (`.canvas-gradient` on app shell + login) |
| `.tile-sky` / `.tile-lavender` / `.tile-neutral` | KPI tile washes (delivery vs conversion vs warning) |
| `--shadow-card` | layered dark shadow |
| `--chart-1..5` | violet, sky, green, amber, rose |

Responsive: sidebar collapses to icons / becomes a sheet under 768px; login is a 2-column grid at `lg`; tables wrap in `overflow-x-auto`; KPI grids `sm:grid-cols-2 lg:grid-cols-4`; clock hidden under `sm`. Main padding `p-4 md:p-6`.

---

## 4. Server-function catalogue (`src/lib/*.functions.ts`)

All are `createServerFn({ method: "POST" }).middleware([requireBrainAuth])`; "section" is the `authorizeBrain` gate.

### `src/lib/membership.functions.ts`, `brain-membership.functions.ts`, `brain-config.functions.ts`
| Function | Section | RPC | Notes |
|---|---|---|---|
| `getMyMembership` | – | `brain_api_my_roles` | Returns `MembershipStatus` incl. displayName/timezone from `user_metadata`. |
| `getMyBrainAccess` | – | `brain_api_my_roles` | Used by consent page. |
| `getBrainConfig` (GET) | – | – | Returns `{url, key}` publishable only. |

### `src/lib/admin.functions.ts`
| Function | Section | RPC / call | Notes |
|---|---|---|---|
| `updateMyProfile({displayName})` | profile | `PUT /auth/v1/user` | 1–80 chars. |
| `updateMyTimezone({timezone})` | settings | `PUT /auth/v1/user` | |
| `listMembers()` | members | `brain_api_admin_list_members` | zod-validated via `readMemberList`; malformed = error, never empty queue. |
| `setMemberAccess({userId, role, active})` | members | `brain_api_admin_set_member_role` | `ASSIGNABLE_ROLES = ["admin","researcher"]` (note: `lead` is recognised but not assignable from UI). Maps `cannot_remove_own_admin`. |
| `getInstantlyIntegration()` | integrations | `brain_api_get_instantly_integration(p_actor)` | configured/enabled/workspace/webhook URL/last event/total events/has_api_key. |
| `rotateInstantlyWebhookToken()` | integrations | `brain_api_rotate_instantly_webhook_token(p_actor)` | Token shown once. |
| `setInstantlyWorkspace({workspaceId})` | integrations | `brain_api_set_instantly_workspace(p_actor, p_workspace_id)` | |
| `saveInstantlyApiKey({apiKey})` | integrations | `brain_api_save_instantly_api_key(p_actor, p_api_key)` | Key stored in Brain DB, never returned. |

### `src/lib/data.functions.ts`
| Function | Section | RPC | Notes |
|---|---|---|---|
| `getOverview()` | overview | `brain_api_status()`, `brain_api_research_metrics(p_actor)` | schools, school_contacts, verified_named_contacts, worker_jobs_pending+running, submission_evidence_unreviewed. |
| `listSchools({q,page})` | schools | `brain_api_list_schools(p_actor,p_q,p_limit,p_offset)` | 50/page. |
| `listContacts({q,status,page})` | contacts | `brain_api_list_contacts(p_actor,p_q,p_status,p_limit,p_offset)` | |
| `getSchool({id})` | schools + contacts | `brain_api_get_school(p_actor,p_school_id)` | |
| `listResearch()` | research | `brain_api_list_research_ops(p_actor)` | allocations + claims. |
| `listOutreach()` | outreach | `brain_api_list_tracking_events(p_actor,p_limit)` | |
| `listPrograms()`, `listRevenue()` | programs/revenue | – | `{available:false}`. |
| `getMyResearchDashboard()` | research | `brain_api_my_research_dashboard()` | caller-scoped, parsed by `parseMyDashboard`. |
| `searchHistoricalCoverage({q,country,city,page})` | research | `brain_api_school_coverage_self(p_query,p_country,p_city,p_limit,p_offset)` | 25/page; parser drops canonical/verification/personal fields. |

### `src/lib/campaigns.functions.ts` (all section `campaigns` unless noted)
| Function | RPC | Notes |
|---|---|---|
| `listCampaigns({q,status,source,programme,page})` | `brain_api_campaigns_list(p_search,p_status,p_source,p_programme,p_limit,p_offset)` | 25/page. |
| `getCampaignDetail({id})` | `brain_api_campaign_detail(p_campaign_id)` | `not_found` handled. |
| `listCampaignRecipients({id,q,match,page})` | `brain_api_campaign_recipients(...)` | NPF recipient-job records. |
| `listCampaignContacts({id})` | `brain_api_campaign_contacts(p_campaign_id,p_limit=200,p_offset)` | `unmatched_records`. |
| `getDeliverabilityOverview()` | `brain_api_outreach_overview(p_days=30,p_limit=0)` | provider_health, totals by event_category, suppressions. |
| `getDataHealth()` (section dataHealth) | `brain_api_data_health()` | |
| `getSourcesSummary()` | `brain_api_sources_summary()` | revenue per currency. |
| `listCampaignLinks({id})` | `brain_api_campaign_links_list(p_campaign_id,p_limit,p_offset)` | |
| `setCampaignSource({id,sourceSlug})` | `brain_api_set_campaign_source(p_campaign_id,p_source_slug)` | `immutable_after_launch`, `invalid_source_slug`. |
| `generateTrackingLink({campaignId,contactId,baseUrl,linkKind,channel})` | `brain_api_generate_tracking_link(...)` | linkKind ∈ calendly/apply/payment/programme/other; channel ∈ email/whatsapp/sms/call/social/in_person/other. |
| `generateTrackingLinksBulk({campaignId,baseUrl,...})` | `brain_api_generate_tracking_links_bulk(..., p_limit=500)` | For NPF-matched contacts (post-send). |
| `generateTrackingLinksForCodes({campaignId,baseUrl,contactCodes[]})` | `brain_api_generate_tracking_links_for_codes(p_campaign_id,p_contact_codes,p_base_url,p_link_kind,p_channel)` | Pre-send by ContactCode (1–2000). |
| `resolveTrackingUrl({utmCampaign,utmMedium,utmSource})` | `brain_api_resolve_three_utm(p_utm_campaign,p_utm_medium,p_utm_source)` | `resolver_version: v2_legacy|v3`. |
| `createCampaign({name,programme,sourceSlug?})` | `brain_api_create_campaign(p_name,p_programme,p_source_slug)` | returns campaign_id + campaign_code, status draft, `sending_enabled:false`. |
| `mapNpfJob({jobId,campaignId})` | `brain_api_map_campaign_job(p_provider='npf',p_job_id,p_campaign_id)` | `already_mapped_to_another_campaign`, `npf_job_not_found`. |
| `mapInstantlyCampaign({externalCampaignId,campaignId})` | `brain_api_map_campaign_job(p_provider='instantly',...)` | unified since e43fbbe. |
| `getCampaignPreflight({id})` | `brain_api_campaign_preflight_self(p_campaign_id)` | ready/blockers/shared_outreach_conflicts/npf_contact_risk. |
| `getCampaignExportRows({id})` | `brain_api_campaign_export_rows(p_campaign_id,p_link_kind=null)` | NPF CSV rows. |
| `importNpfReport({fileName,fileSha256,records[],exportAsOf?,sourceTimezone?,sourceBytes?})` | `brain_api_import_npf_report(p_file_name,p_file_sha256,p_records,p_export_as_of,p_source_timezone,p_source_bytes)` | idempotent by SHA-256: `identical_file_already_imported`. |
| `getInstantlyCampaignSummary({id})` | `brain_api_instantly_campaign_summary(p_campaign_id)` | sent/opened/clicked/bounced/replied/unsubscribed/meetings_booked/... |
| `getInstantlyUnmapped()` (section dataHealth) | `brain_api_instantly_unmapped()` | |
| `getInstantlyCampaignsList()` | `brain_api_instantly_campaigns_list()` | |
| `listEmailTemplates()` / `saveEmailTemplate({id?,name,subject,bodyHtml,designJson?})` / `deleteEmailTemplate({id})` | `brain_api_list_email_templates()`, `brain_api_save_email_template(p_id,p_name,p_subject,p_body_html,p_design_json)`, `brain_api_delete_email_template(p_id)` | `design_json` added 11-06 for Unlayer. |
| `checkContactRows({rows[{email,name?,school?}],commit})` | `brain_api_check_contact_rows(p_rows,p_commit)` | existing/new/created/invalid_email/skipped_missing_name. |

---

## 5. Backend (Brain, `<BRAIN_REF>`) – what the repo lets us verify

### 5.1 Sources of truth available locally
- **One committed migration**: `supabase/migrations/20261029121253_<uuid>.sql` (admin list/set member role RPCs; full text in `docs/01-supabase/brain-rpc-pattern.md`).
- **Generated types** `src/integrations/supabase/types.ts` (4 855 lines, last regenerated 2026-11-03 from Brain's `public` schema; AGENTS.md calls it inert – nothing imports it except the unused `client.ts`).
- **Commit messages** from 11-05/11-06 (nishanta-code) describe every RPC/trigger/cron/table touched; the SQL itself was applied directly to Brain through the Supabase MCP and **is not in the repo**.
- `src/lib/campaigns-data.ts` comment: campaign RPCs are "SECURITY DEFINER, gated to admin/lead via brain_ops.member_roles, read-only ... See migration applied 2026-11-05" (that migration is not committed).

### 5.2 Schemas and tables referenced

`brain_ops` (named in code/migration/commits): `member_roles` (user_id, role, active, granted_by, granted_at, revoked_at; unique(user_id, role)), `action_audit` (actor_user_id, workstream, tool_name, action, entity_kind, entity_id, result_status, details), `npf_jobs`, `instantly_campaign_links`, `campaign_jobs` (new 11-06 mirror of both), `email_templates` (+ `design_json` 11-06), `outreach_suppressions`, `outreach_history_v1` (Instantly webhook sink), functions `brain_ops.sync_outreach_suppressions()`, `brain_ops.campaign_preflight_unified_v2`.

`public` (from types.ts snapshot of Brain, 2026-11-03) – tables: `affiliations, consents, merge_lineage, org_aliases, org_group_domains, org_groups, organizations, people, person_identities, programme_aliases, programmes, research_agent_runs, research_agent_settings, research_allocations, research_campaigns, research_claims, research_cleaning_runs, research_contact_health, research_contacts, research_datasets, research_designation_proposals, research_domain_sites, research_duplicate_log, research_email_check_requests, research_email_check_runs, research_email_check_state, research_email_checks, research_email_health, research_field_changes, research_groups, research_identity_matches, research_job_locks, research_jobs, research_legacy_source_map, research_locations, research_optouts, research_outreach_claims, research_people, research_person_emails, research_person_links, research_person_schools, research_proposals, research_raw_rows, research_ref_cities, research_ref_countries, research_ref_states, research_registry_schools, research_review_outcomes, research_review_rows, research_school_facts, research_school_findings, research_school_intel, research_school_links, research_school_profiles, research_schools, research_settings, research_source_tools, research_spend, research_staff_profiles, research_tasks, research_tracking_events, research_tracking_tokens, research_upload_tabs, research_uploads, sensitive_access_log, source_systems, staff, staff_grants, staff_invitations, staff_role_assignments, staff_scopes, suppressions`; views `research_email_inventory_current_v1, research_email_inventory_v1, research_import_email_coverage_v1, researcher_contact_sources_v1, researcher_programme_claims_v1`; enums `channel, consent_basis, consent_purpose, data_zone, fact_origin, identity_kind, inbound_kind, lifecycle_stage, meeting_source, meeting_status, motion, payment_status, person_role, staff_grant, staff_role, staff_status, suppression_reason, verdict`.

Tables the app reads *through RPCs* per commit 8c3f205/4f50de0: `research_schools`, `research_contacts`, `research_allocations`, `research_claims`, `research_tracking_events` – all "RLS enabled with zero policies".

### 5.3 RLS state
Verified only indirectly: commits 8c3f205 and 4f50de0 state that `research_schools`, `research_contacts`, `research_allocations`, `research_claims`, `research_tracking_events` have RLS enabled and **zero policies**, so `.from()` reads under the caller's JWT returned empty sets without error. The app's response was to route every read through SECURITY DEFINER RPCs. The README (10-29) says effective per-user row visibility "has not been claimed or changed". No `CREATE POLICY` exists in the repo.

### 5.4 Complete `brain_api_*` catalogue

Legend – **Src**: `M` = committed migration (full SQL known), `C` = called from app code (signature from call site), `T` = present in types.ts snapshot (signature from generator), `G` = described in a commit message. **Role**: `admin` / `lead+` (admin or lead) / `member` (any active role) / `self` (caller-scoped, no role gate stated) / `token` (shared-secret p_token, not a user) / `?` = not stated anywhere.

| Function (args) | Src | Role | Tables touched (as stated) | Purpose |
|---|---|---|---|---|
| `brain_api_my_roles()` -> `text[]` | C,T | self | brain_ops.member_roles | Caller's active roles from `auth.uid()`. Core auth gate. |
| `brain_api_member_roles(p_user_id)` -> `text[]` | T | ? | brain_ops.member_roles | Older per-user variant; README asks Brain admin to confirm it only returns own roles. Not called by app. |
| `brain_api_admin_list_members()` -> jsonb | M,C,T | admin | auth.users, brain_ops.member_roles | All non-deleted users with roles (STABLE). |
| `brain_api_admin_set_member_role(p_user_id,p_role,p_active)` -> jsonb | M,C,T | admin | brain_ops.member_roles, brain_ops.action_audit | Grant/revoke `admin`/`researcher`; blocks self-admin removal; audits `workstream='admin'`. |
| `brain_api_status()` -> jsonb | C,T | ? | counts | `schools`, `school_contacts` totals (dashboard). |
| `brain_api_research_metrics(p_actor)` -> jsonb | C,T | ? | research_* | verified_named_contacts, worker_jobs_pending/running, submission_evidence_unreviewed. |
| `brain_api_list_schools(p_actor,p_q,p_limit,p_offset)` | C,G | member | research_schools | Paged search (added 8c3f205). |
| `brain_api_list_contacts(p_actor,p_q,p_status,p_limit,p_offset)` | C,G | member | research_contacts | Paged search with validated filter. |
| `brain_api_get_school(p_actor,p_school_id)` | C,G | member | research_schools, research_contacts | School + contacts. |
| `brain_api_list_research_ops(p_actor)` | C,G | member | research_allocations, research_claims | |
| `brain_api_list_tracking_events(p_actor,p_limit)` | C,G | member | research_tracking_events | Last direct read replaced (4f50de0). |
| `brain_api_my_research_dashboard()` | C,T | self | submissions/claims | own_total_submissions, own_pending_submissions, own_active_claims, own_recent_submissions. |
| `brain_api_school_coverage_self(p_query,p_country,p_city,p_limit,p_offset)` | C,T | self | historical staging | historical_schools rows with `has_more`, `warning`. |
| `brain_api_historical_school_coverage(p_actor,...)`, `_page(...)` | T | ? | | Older actor-passing variants; not called by app. |
| `brain_api_campaigns_list(p_search,p_status,p_source,p_programme,p_limit,p_offset)` | C,G | lead+ | campaigns, npf_jobs, recipient reports | Phase 1 reporting. |
| `brain_api_campaign_detail(p_campaign_id)` | C,G | lead+ | same | campaign + npf_jobs + reporting + latest_report_at. |
| `brain_api_campaign_recipients(p_campaign_id,p_search,p_match_status,p_limit,p_offset)` | C,G | lead+ | NPF recipient reports | per-recipient timestamps, match_status, vendor_response. |
| `brain_api_campaign_contacts(p_campaign_id,p_limit,p_offset)` | C,G | lead+ | contacts/schools joined to reports | |
| `brain_api_data_health()` | C,G | lead+ | many (counts) | Data Health page. |
| `brain_api_sources_summary()` | C,G | lead+ | campaigns by attribution_source_slug | Revenue per currency fixed 6829662. |
| `brain_api_outreach_overview(p_days,p_limit)` | C,G | lead+ | outreach events, outreach_suppressions | Pre-existing, wired 9f8f03d. |
| `three_utm_url_v3(...)` (helper, not api-prefixed) | G | – | – | Builds `utm_campaign=CampaignCode&utm_medium=<source>&utm_source=<ContactCode>`. |
| `brain_api_resolve_three_utm(p_utm_campaign,p_utm_medium,p_utm_source)` | C,G | lead+ | campaigns, contacts, schools | Auto-detects v2 (legacy) vs v3 by which value matches `^CT[A-Z0-9]{10}$`. |
| `brain_api_set_campaign_source(p_campaign_id,p_source_slug)` | C,G | admin (per 43801d6 "admin-only, audited") | campaigns, action_audit | `immutable_after_launch` from trigger. |
| `brain_api_generate_tracking_link(p_campaign_id,p_contact_id,p_base_url,p_link_kind,p_channel)` | C,G | lead+ | campaign_recipients, campaign_links, action_audit | One link. |
| `brain_api_generate_tracking_links_bulk(p_campaign_id,p_base_url,p_link_kind,p_channel,p_limit)` | C,G | lead+ | same | For NPF-matched contacts. |
| `brain_api_generate_tracking_links_for_codes(p_campaign_id,p_contact_codes text[],p_base_url,p_link_kind,p_channel)` | C,G | lead+ | same | Pre-send by ContactCode (d56f71b). |
| `brain_api_campaign_links_list(p_campaign_id,p_limit,p_offset)` | C,G | lead+ | campaign_links | |
| `brain_api_create_campaign(p_name,p_programme,p_source_slug)` | C,G | lead+ | campaigns, action_audit | Draft with immutable UUID + CampaignCode; actor from auth.uid(). |
| `brain_api_map_npf_job(...)` | G | lead+ | brain_ops.npf_jobs | Superseded by map_campaign_job (e43fbbe); may still exist. |
| `brain_api_map_campaign_job(p_provider,p_job_id,p_campaign_id)` | C,G | lead+ | brain_ops.npf_jobs / instantly_campaign_links / campaign_jobs, action_audit | Unified NPF+Instantly linking. |
| `brain_api_campaign_preflight_self(p_campaign_id)` | C,G | lead+ | wraps brain_ops.campaign_preflight_unified_v2 | eligibility, suppression, bounce history, conflicts. |
| `brain_api_campaign_preflight(p_campaign_code)` | T | ? | | Older variant by code. |
| `brain_api_campaign_export_rows(p_campaign_id,p_link_kind)` | C,G | lead+ | campaign_links + contacts | NPF CSV rows. |
| `brain_api_import_npf_report(p_file_name,p_file_sha256,p_records,p_export_as_of,p_source_timezone,p_source_bytes)` | C,G | lead+ | NPF recipient reports, import log | Wraps "existing, already-proven ingestion function"; idempotent by SHA. |
| `brain_api_instantly_campaign_summary(p_campaign_id)` | C,G | lead+ | brain_ops.outreach_history_v1, instantly_campaign_links | |
| `brain_api_instantly_unmapped()` | C,G | lead+ | same | |
| `brain_api_instantly_campaigns_list()` | C,G | lead+ | same | |
| `brain_api_ingest_instantly_v1(p_data,p_token)` | T,G | token | brain_ops.outreach_history_v1 | Instantly webhook receiver (called by Instantly, not the app; "53 real campaigns already sending" per f688508). |
| `brain_api_get_instantly_integration(p_actor)` / `_rotate_instantly_webhook_token(p_actor)` / `_set_instantly_workspace(p_actor,p_workspace_id)` / `_save_instantly_api_key(p_actor,p_api_key)` | C,G | admin (section `integrations`) | integration settings table (name not stated) | Settings card. |
| `brain_api_list_email_templates()` / `_save_email_template(p_id,p_name,p_subject,p_body_html,p_design_json)` / `_delete_email_template(p_id)` | C,G | lead+ | brain_ops.email_templates | |
| `brain_api_check_contact_rows(p_rows,p_commit)` | C,G | lead+ | research_contacts (create on commit) | CSV contact dedup/create. |
| `brain_api_create_draft_campaign(p_actor,p_name,p_programme_code)`, `_add_draft_recipient(p_actor,p_campaign_code,p_contact_code,p_medium)` | T,G | postgres/service_role only (per 2e2c72d) | | Older RPCs that trust a client-supplied `p_actor`; deliberately **not** used. |
| `brain_api_submit_research`, `_submit_claimed_school_research`, `_submit_contact_evidence`, `_append_research_evidence`, `_autosave_research_v1`, `_claim_research_school`, `_claim_research_school_v2`, `_review_research_submission`, `_review_source_evidence`, `_promote_research_school`, `_promote_research_contact`, `_enqueue_school_research`, `_find_canonical_schools`, `_school_candidates`, `_research_queue`, `_operating_rules`, `_outreach_check_v1`, `_record_funnel_v1`, `_revenue_report_v1`, `_register_research_file`, `_begin_import_run`, `_stage_import_rows`, `_ingest_legacy_batch`, `_master_migration_status`, `_verify_master_sync` | T | ? | | Exist in Brain (types snapshot) and are presumably consumed by the ChatGPT MCP (`brain-mcp`) and migration tooling; **not called by this app**. |
| `lwl_auto_review_claim_v1(p_token)`, `lwl_auto_review_finish_v1(...)`, `lwl_migration_*_v2(..., p_token)`, `npf_server_reconcile_job(p_job_id,p_after_email,p_limit)` | T | token | | Non-api-prefixed Brain functions in the snapshot; not called by this app. |

### 5.5 Triggers, cron, edge functions, integrations

| Kind | Name | Evidence | Behaviour |
|---|---|---|---|
| Trigger | `guard_campaign_tracking_identity_v2` | commit 43801d6 | Makes `attribution_source_slug` (tracking identity) immutable once a campaign is `live`/`done`. UI mirrors this (`SourceControl` locks when status is live/done). |
| Cron (pg_cron on Brain) | hourly `brain_ops.sync_outreach_suppressions()` | commit 9f8f03d | Scheduled 11-06; first run produced 2 161 active suppressions (2 142 NPF + 19 Instantly). Job name not recorded in repo. |
| Edge function | `brain-mcp` | `BRAIN_MCP_URL` in `src/lib/brain-config.ts`, `MCP_URL` in onboarding guide, README | ChatGPT MCP target. Source not in this repo; auth/tables unknown here. No other edge functions are referenced; `supabase/functions/` does not exist in this repo. |
| Integration: Instantly | API key + webhook token stored in Brain via RPCs (no env secret in this repo) | `admin.functions.ts`, settings page | Webhook -> `brain_api_ingest_instantly_v1(p_token)`; workspace-ID check on events. |
| Integration: NPF / Meritto | none (manual CSV) | campaign detail | Export NPF-ready CSV (Email, Name, School, Contact ID, Campaign Code, Source, Tracking URL, Link Kind; CRLF); upload "Communication Performance" CSV. |
| Integration: Unlayer | `react-email-editor` (client-side only, no key in code) | templates route | Design JSON persisted. |
| Integration: ZeroBounce | mentioned only in commit 9f8f03d as prior pre-send validation work with no per-email verdict data | – | Not integrated in this repo. |
| Amazon SES | "planned third provider, not yet connected" (`campaigns-data.ts`) | – | – |
| OpenAI / Anthropic / Lovable AI gateway / Resend | **none** (`grep` of src is empty; `@lovable.dev/ai-gateway` appears only in `bunfig.toml` exclusions) | – | – |
| Lovable telemetry | `window.__lovableEvents`, `__lovableReportRuntimeError` | `src/lib/lovable-error-reporting.ts` | Editor-preview-only error forwarding. |

`_shared` helpers: none (no edge functions in repo). The closest equivalents are `src/lib/brain.server.ts` (`authenticateBrain`, `brainAccess`, `authorizeBrain`, `brainKey`) and `callRpc()` in `campaigns.functions.ts`.

---

## 6. End-to-end flows

### 6.1 Campaign creation -> tracking link -> send -> report -> attribution (NPF)
1. Lead/admin opens `/campaigns/npf` -> **Create Campaign** (name 4–180, programme 2–180, optional source slug `^[a-z0-9-]{3,96}$`) -> `brain_api_create_campaign` -> campaign row `status=draft` with immutable `campaign_uuid` + `campaign_code`. Navigates to `/campaigns/:id`.
2. Overview tab -> **Source mapping** (`brain_api_set_campaign_source`). Allowed only while draft/paused; `guard_campaign_tracking_identity_v2` rejects after launch (`immutable_after_launch`). Source slug becomes `utm_medium`.
3. Tracking Links tab -> paste ContactCodes (`CT` + 10 alnum) + destination URL -> `brain_api_generate_tracking_links_for_codes` -> per-contact `campaign_recipients` + `campaign_links` rows; URL built by `three_utm_url_v3`: `utm_campaign=<CampaignCode>&utm_medium=<source slug>&utm_source=<ContactCode>`.
4. **Run preflight** (`brain_api_campaign_preflight_self` -> `brain_ops.campaign_preflight_unified_v2`): eligibility, suppressions, prior bounce/unsubscribe, shared-outreach conflicts -> `ready`/`blockers[]`.
5. **Download NPF-ready CSV** (`brain_api_campaign_export_rows`, client builds CSV with NPF-friendly headers) -> upload into NPF/Meritto, send.
6. NPF assigns a Job ID -> Overview tab **Link NPF Job ID** (`brain_api_map_campaign_job(provider='npf')`) -> `brain_ops.npf_jobs` + `campaign_jobs`.
7. Email Performance tab -> **Upload NPF report** CSV: browser parses (`parseCsv`), hashes (`sha256Hex`), posts up to 25 000 records -> `brain_api_import_npf_report` -> inserted/updated/unchanged/stale; re-uploads update only changed rows; identical file short-circuits.
8. Recipient rows are matched to contacts (`match_status` matched/unmatched/ambiguous), surfaced on detail KPIs, Data Health, Sources, and the notification bell.
9. Any later click on the tracking URL can be resolved with `brain_api_resolve_three_utm` (campaign + contact + school + source). Applications (Phase 3) and payments (Phase 4) are **not connected** – tabs show explicit placeholders; `provisionally_attributed_applications`, `recorded_enrollments`, `recorded_net_payment_minor` fields exist in the summary row but are 0 today per commit 6829662.

### 6.2 Instantly (live)
Admin: `/settings` -> generate webhook secret (`brain_api_rotate_instantly_webhook_token`, shown once) -> paste URL into Instantly -> set workspace ID -> optionally store API key. Instantly posts events to Brain (`brain_api_ingest_instantly_v1`) into `brain_ops.outreach_history_v1`. Lead: `/campaigns/instantly` lists every external campaign with events; campaign detail Instantly tab links an external ID (`map_campaign_job(provider='instantly')`) and shows sent/opened/clicked/bounced/replied/unsubscribed/meetings_booked. Data Health lists unlinked Instantly campaigns. Templates (plain HTML or Unlayer) use `{{name}}`/`{{tracking_url}}` merge tags; the actual send-from-portal step is described in UI copy as "launch" but **no send RPC exists in this repo**.

### 6.3 Contact list ingest (Instantly tab)
CSV with Email/Name/School -> client dedupes by lower-cased email -> `brain_api_check_contact_rows(p_commit=false)` reports existing (with ContactCode) / new / invalid / skipped_missing_name -> `p_commit=true` creates new contacts and returns their ContactCodes.

### 6.4 Deliverability & suppressions
Hourly `brain_ops.sync_outreach_suppressions()` folds NPF bounces/unsubscribes and Instantly bounces into `outreach_suppressions`; dashboard reads `brain_api_outreach_overview(30 days)`; preflight consults the same suppressions.

### 6.5 Data health
`brain_api_data_health()` returns 18 verified counts (campaigns with/without NPF jobs, missing source slug, NPF jobs unmapped, recipient reports matched/unmatched/ambiguous, native recipients/links, applications (+ without attribution), enrollments, payment events, canonical schools/contacts, generated_at). Nothing is auto-corrected.

### 6.6 Team access / ChatGPT onboarding
Admin approves a signed-up Brain user as researcher (confirmation dialog) -> `brain_api_admin_set_member_role` -> `brain_ops.action_audit` row (`workstream='admin'`). The onboarding wizard then walks through inviting the person to ChatGPT Business, publishing the MCP app, connecting (sign in at the portal + consent page), and testing `brain_status`.

---

## 7. Research data flow and the `<DATA_REF>` relationship

**Verifiable from this repo**
- The app talks to exactly one database: Brain `<BRAIN_REF>` (`BRAIN_PROJECT_REF` constant, tests assert it).
- Brain's `public` schema (types snapshot 2026-11-03) contains the same family of `research_*` tables (`research_contact_health`, `research_contacts`, `research_schools`, `research_campaigns`, ...) plus `people/organizations/consents/suppressions/staff*` and the `lwl_migration_*_v2(p_token)` / `brain_api_ingest_legacy_batch(p_batch_key,p_rows,p_run_id,p_source_cursor,p_source_table)` / `brain_api_verify_master_sync(p_payload_text,p_signature)` / `brain_api_master_migration_status()` functions – i.e. Brain has token-authenticated **ingestion endpoints for batches from a "legacy"/"master" source**. The `research_legacy_source_map` table and the Research page copy ("Historical research from older systems is not linked to your account here") corroborate a migration from an older system.
- The repo never names `<DATA_REF>`; the only "old" project it names is `txmlngnuwqqhgdtqjydc` ("LwL Master Research").

**Verified by read-only catalogue queries on `<DATA_REF>` (2026-11-24, this session)**
- Schemas: `attribution, brain, core_ro, leads, legacy_ro, legacy_sat, lwl, migration, quarantine, revenue_brain, staging, public` (+ Supabase system schemas).
- `public` has **133** `research_*` tables; `public.research_contact_health` planner estimate ≈ **600 772** rows; `public.research_contacts` and `public.research_campaigns` exist.
- **No `brain_ops` schema and no `brain_api_*` functions** exist there.
- **No function body in that project mentions `<BRAIN_REF>`.**
- `attribution` schema holds `campaigns, contacts, outreach, schools, suppressions, settings, circulation_overrides`; `brain` holds `knowledge_*` tables (a "knowledge base", not research data); `migration` holds `id_map, run_log, baseline_20260902, payments_*`; `quarantine` holds `research_contacts, research_schools, import_*, datasets, email_evidence...`; `revenue_brain` holds outreach/sequence/conversion/pilot tables; pg_cron has ~100 `portal-*`/`lm_*`/`spine_*` jobs incl. `research_retry_tick_30m`.

**Not verifiable (do not assume)**
- Whether Brain's `public.research_*` tables were seeded from `<DATA_REF>` (vs. from `txmlngnuwqqhgdtqjydc` or a spreadsheet pipeline), in which direction any sync runs, or whether any sync still runs. The token-based `lwl_migration_*` / `ingest_legacy_batch` functions on Brain are *consistent with* a push from another database, but the pusher is not in this repo.
- The definition of Brain's `brain-mcp` edge function, its tables, and Brain's RLS/policies beyond the five tables named in commits.
- Whether `attribution.*` / `revenue_brain.*` on `dgzx` and `brain_ops.*` on Brain are related in any way.

---

## 8. Conventions

- **`AGENTS.md`** (16 rules, Lovable banner): Brain-only integration in `src/lib/brain-*`; `getMyMembership` + all protected server fns call `brain_api_my_roles` with the caller's JWT; empty array = not approved, RPC error = pending, unknown roles deny; no privileged clients; member management only via caller-scoped admin RPCs that write `brain_ops.action_audit`; assignable roles must exist in both app role list and Brain's `member_roles` constraint; Team access UI requires per-member confirmation; Brain projections in `brain-data.ts` verified against target before use; profile names never source roles; redirects through `safeReturnPath`; consent UI one component at two aliases; security headers in `src/server.ts`; permissions defined once in `src/lib/permissions.ts`; signed-in pages under `_app`; generated Supabase compatibility modules only forward to Brain or reject.
- **`roadmap.md`**: Brain cutover checklist – 5 done, 3 blocked (verify real staff membership/RLS/reset/consent; enable admin/programme/revenue/evidence; switch editor-managed binding).
- **`README.md`**: Lovable boilerplate + "Brain cutover (2026-10-29)" section with configuration gates and post-deploy smoke checks (28 regression tests passed at that time).
- **`skills-lock.json`** / `.agents/skills/`: pinned Supabase agent skills (hashes recorded).
- **`CLAUDE.md`**: none in this repo. Commit e202c40 also tracks a second copy of the two skills under `.claude/skills/supabase/` and `.claude/skills/supabase-postgres-best-practices/` (regular files, mode 100644), but in this local clone those paths are **deleted in the working tree** (`git status` shows ` D .claude/skills/...`; the deletion pre-dates this documentation pass). `.agents/skills/` is the intact copy.
- **`.lovable/plan/*.md`**: three planning docs (10-26 identity portal, 10-27 role-based dashboard, 10-29 team access).
- Code style: dense single-file routes, zod validators inline, `exactOptionalPropertyTypes` (spread-conditional object building), `titleCase()` for every enum label, CampaignCode/ContactCode are the only identifiers shown (UUIDs are plumbing).
- Commit style since 11-05: imperative subject + explanatory body + `Co-Authored-By: Claude Sonnet 5`. Lovable commits are "Changes"/"Work in progress"/"Lovable update" with `X-Lovable-Edit-ID`.

---

## 9. Tests (`src/test/`, vitest + jsdom)

| File | Covers |
|---|---|
| `admin-access.test.ts` | `listMembers`/`setMemberAccess` handlers with a fake `db.rpc`: non-admins denied before admin RPC; `[]` vs RPC error distinguished; malformed lists are errors; `cannot_remove_own_admin` preserved. |
| `auth-rules.test.ts` | `safeReturnPath` (consent path kept, external/`//`/backslash rejected), `isValidAuthorizationId`, `parseScopes`, `decideMembership`. |
| `brain-access.test.ts` | `parseBrainRoles`/`decideBrainAccess` (objects, inactive/approved:false, unknown roles), storage key not `sb-txml...`. |
| `brain-cutover.test.ts` | `authenticateBrain` uses only Brain URL; `authorizeBrain` calls `brain_api_my_roles` with no args and never `.from()`; empty roles = none; invalid JWT rejected before role lookup; RPC error = "verification unavailable"; researcher blocked from members/outreach; `mapContact` keeps "validated" wording. |
| `brain-research.test.ts` | `parseMyDashboard` / `parseCoverage` (denied vs unavailable, drops canonical/verification/email fields), 25-per-page args, and a **source-text assertion** that research files never mention `txmlngnuwqqhgdtqjydc`, `supabaseAdmin` or `client.server`. |
| `permissions.test.ts` | `can()` matrix for researcher/lead/admin/unknown. |
| `post-login.test.ts` | `postLoginPath` only honours consent paths. |
| `app-routing.test.tsx` | `/` matches a real route. |
| `onboarding-guide.test.tsx` | counts from member list, 5 steps, copy MCP URL, members route admin-only. |
| `team-access.test.tsx` | confirmation-gated grant/revoke, no duplicate submits, busy state, failed change stays visible, `readMemberList` validation, route guard matrix. |
| `setup.ts` | jest-dom, `scrollTo`/`matchMedia` stubs. |

Not covered: campaign server functions, CSV parser, routes under `_app.campaigns*`, Unlayer editor, consent page UI.

---

## 10. Timeline (from `git log`)

| Date | Commits | Milestone |
|---|---|---|
| 2026-10-16 | 1 | Lovable template `tanstack_start_ts_current`. |
| 2026-10-26 | 23 | "Team Research" page -> pivot to Identity & Access Portal; connect to `txmlngnuwqqhgdtqjydc`; Supabase sign-in, user credentials, profile role management (plan: `research_members` table on Master Research). |
| 2026-10-27 | 13 | Role-based dashboard plan; sidebar + Overview/Schools/Contacts/Research/Outreach/Programs/Revenue/Members/Profile; "Launched Overview on login". |
| 2026-10-28 | 5 | OAuth consent flow fixes, login redirect deps. |
| 2026-10-29 | 53 | **Brain cutover**: "Added Brain-only auth mode", reconnect `config.toml` to `<BRAIN_REF>`, committed migration (admin RPCs), "Added admin brain & members", members page, approvals UI, `brain-research` RPC parsers, onboarding wizard, Team Access in nav. Lovable prompt commits also show staff accounts being created in Brain Auth (details intentionally omitted here). |
| 2026-10-30 / 11-03 | 4 | Lovable account-creation prompts; types regenerated. |
| 2026-11-05 | 19 | **Phase 1** campaigns dashboard (6 reporting RPCs; 70 campaigns / 48 NPF jobs / 44 890 recipient reports at the time); **Phase 2** three-UTM links + source mapping + resolver; campaign creation workflow (+ preflight/export); Sources page; Group bird logo; Campaigns on dashboard; pastel accent system -> density pass -> **dark violet reskin**; NPF column labels; pre-send links by Contact ID; CampaignCode/ContactCode only; nav cleanup; NPF report upload + Instantly linking; sidebar footer identity + `titleCase`; live clock/notifications/Settings; NPF vs Instantly segments, templates, contact dedup, and the **`workstream='campaign'` check-constraint fix** (11 write RPCs regenerated). |
| 2026-11-06 | 6 | RLS-blocked reads fixed via RPCs (schools/contacts/research, then tracking events); Instantly integration settings; unified `map_campaign_job`; Unlayer template builder (+`design_json`); suppression sync turned on + hourly cron + deliverability on dashboard; `.mcp.json` + Supabase agent skills. |

---

## 11. Gotchas (all evidenced in this repo)

1. **RLS enabled + zero policies looks like empty data, not an error.** `.from()` under the user's JWT silently returned `[]` for `research_schools/contacts/allocations/claims/tracking_events` (commits 8c3f205, 4f50de0). Fix = SECURITY DEFINER `brain_api_*` RPC; app now has zero direct reads. Likewise a **missing `GRANT EXECUTE`** on an RPC surfaces as `42501 permission denied`, which `brainAccess()` reports as "verification unavailable" – not a policy bug.
2. **`brain_ops.action_audit.workstream` CHECK constraint**: every write RPC created on 11-05 used `'campaigns'` (plural) and silently rolled back; the constraint only allows the singular `'campaign'` (commit fb83e12). The committed migration uses `'admin'`. The full allowed list is not in the repo – see the pattern doc for how to look it up before writing a new RPC.
3. **Immutability trigger `guard_campaign_tracking_identity_v2`**: `attribution_source_slug` cannot change once status is `live`/`done` (`immutable_after_launch`). All 70 historical campaigns therefore stay "Source not mapped"; set the source on draft/paused campaigns only.
4. **Older actor-trusting RPCs** (`brain_api_create_draft_campaign`, `_add_draft_recipient`, and the `p_actor` family) accept a client-supplied UUID. New RPCs derive the actor from `auth.uid()`. Do not grant the old ones to `authenticated`.
5. **Lovable drift**: Lovable can push to `main` (98 bot commits). `AGENTS.md` forbids history rewrites; always fetch before local work. The Lovable editor's Supabase binding (`SUPABASE_*`/`VITE_SUPABASE_*`, generated `client.ts`/`types.ts`) is *not* the runtime source – runtime reads only `LWL_BRAIN_PUBLISHABLE_KEY`.
6. **bun vs npm**: `package.json` carries `overrides.rolldown` while depending on Vite 8 (which bundles rolldown). bun tolerates an overrides/dependencies conflict that npm rejects; use `bun install` (README's `npm i` is stale). `bunfig.toml` enforces a 24h minimum release age except for `@lovable.dev/*`.
7. **Realtime**: not used in this repo (no `.channel(`); nothing to collide. (The sibling-repo lesson about fixed-name Realtime channels and React double-invoke does not apply here.)
8. **Function overload / signature drift**: `brain_api_save_email_template` gained `p_design_json` on 11-06 and `map_npf_job` was replaced by `map_campaign_job(p_provider, ...)`. Supabase RPC calls are resolved by argument *names*; if two overloads exist, PostgREST returns an ambiguity error – drop the old signature when changing one.
9. **Lead role is not assignable from the UI** (`ASSIGNABLE_ROLES = ["admin","researcher"]`) and the committed `admin_set_member_role` RPC also rejects `'lead'`, although `lead` is a recognised workspace role with more sections than researcher.
10. **`exactOptionalPropertyTypes`** means `{ q: undefined }` is a type error – server-fn inputs are built with conditional spreads; copy that pattern.
11. `src/start.ts` must keep `createCsrfMiddleware` explicitly – defining the file disables the default.
12. The visual template route reads `existing` from the full `listEmailTemplates()` list (no single-template RPC) and hydrates state once; large template sets will be fetched entirely.

---

## 12. File map (key files, repo-relative)

| Path | Role |
|---|---|
| `AGENTS.md` | Lovable banner + 16 architecture rules |
| `README.md` | Lovable boilerplate + Brain cutover notes/gates |
| `roadmap.md` | Cutover checklist |
| `skills-lock.json`, `.agents/skills/**` | Pinned Supabase agent skills |
| `.mcp.json` | Supabase MCP for project `<BRAIN_REF>` |
| `.lovable/project.json`, `.lovable/plan/*.md` | Lovable template + 3 plan docs |
| `package.json`, `bun.lock`, `bunfig.toml` | bun workspace, release-age guard |
| `vite.config.ts` | `@lovable.dev/vite-tanstack-config`, server entry `src/server.ts` |
| `vitest.config.ts`, `tsconfig.json`, `eslint.config.js`, `components.json` | tooling |
| `supabase/config.toml` | `project_id = <BRAIN_REF>` |
| `supabase/migrations/20261029121253_*.sql` | Only committed SQL: admin list/set member role RPCs + grants |
| `src/server.ts` | SSR entry wrapper: h3 500 normalisation, security headers (CSP/XFO/HSTS/Referrer/nosniff) |
| `src/start.ts` | request middleware (error + CSRF), client middleware attaching Brain bearer token |
| `src/router.tsx`, `src/routeTree.gen.ts` | router + generated tree |
| `src/styles.css` | design tokens (dark violet), utilities |
| `src/lib/brain-config.ts` | `BRAIN_PROJECT_REF`, `BRAIN_URL`, `BRAIN_STORAGE_KEY`, `BRAIN_MCP_URL` |
| `src/lib/brain-config.functions.ts` | `getBrainConfig` (publishable key only) |
| `src/lib/brain-client.ts` | browser Brain client + `supabase.auth` proxy |
| `src/lib/brain-auth.ts` | `requireBrainAuth` middleware |
| `src/lib/brain.server.ts` | `authenticateBrain`, `brainAccess`, `authorizeBrain`, `brainKey` |
| `src/lib/brain-roles.ts` | `parseBrainRoles`, `decideBrainAccess`, `brainWorkspaceRole` |
| `src/lib/brain-data.ts` | verified column projections + mappers for schools/contacts |
| `src/lib/brain-research.ts` | zod input + parsers for caller-scoped research RPCs |
| `src/lib/membership.ts`, `membership.functions.ts`, `brain-membership.functions.ts` | membership types + server fns |
| `src/lib/permissions.ts`, `route-guard.ts` | `SECTIONS`, `can()`, `guard()`, `pageHead()` |
| `src/lib/safe-redirect.ts`, `session.ts`, `oauth-client.ts` | redirect safety, sign-out, OAuth 2.1 consent API shim |
| `src/lib/admin.functions.ts` | profile/timezone, members, Instantly settings |
| `src/lib/data.functions.ts` | overview/schools/contacts/research/outreach |
| `src/lib/campaigns.functions.ts` | 27 campaign/NPF/Instantly/template server fns |
| `src/lib/campaigns-data.ts` | RPC envelope types (`CampaignSummaryRow`, `DataHealth`, `TrackingResolution`, ...) |
| `src/lib/csv.ts` | RFC4180-ish parser + SHA-256 |
| `src/lib/team-members.ts` | zod validation of member list |
| `src/lib/error-capture.ts`, `error-page.ts`, `lovable-error-reporting.ts` | SSR error plumbing |
| `src/integrations/supabase/{client.ts,types.ts,previewAuthStorage.ts}` | Lovable-generated, inert at runtime |
| `src/integrations/supabase/{client.server.ts,auth-middleware.ts}` | compatibility shims: throw / forward to Brain |
| `src/routes/**` | 30 route files (see 3.1) |
| `src/components/{app-sidebar,topbar-widgets,data-view,list-controls,portal-shell,consent-page,team-access,onboarding-guide,unavailable-data}.tsx` | app components |
| `src/components/ui/*` | shadcn primitives |
| `src/test/*` | 11 vitest files |
| `public/lwl-bird.png`, `public/favicon.ico`, `public/robots.txt` | assets |

---

## 13. Page -> server function -> RPC matrix

| Page | Server functions (query key) | RPCs hit |
|---|---|---|
| `/dashboard` | `getOverview` (`overview`); lead+: `listCampaigns` (`dashboard-recent-campaigns`), `getDataHealth` (`dashboard-data-health`), `getDeliverabilityOverview` (`dashboard-deliverability`) | `status`, `research_metrics`, `campaigns_list`, `data_health`, `outreach_overview` |
| `/schools` | `listSchools` (`schools,q,page`) | `list_schools` |
| `/schools/:id` | `getSchool` (`school,id`) | `get_school` |
| `/contacts` | `listContacts` (`contacts,q,status,page`) | `list_contacts` |
| `/research` | `getMyResearchDashboard` (`brain-my-research`), `searchHistoricalCoverage` (`brain-coverage`), `listResearch` (`research`) | `my_research_dashboard`, `school_coverage_self`, `list_research_ops` |
| `/outreach` | `listOutreach` (`outreach`) | `list_tracking_events` |
| `/campaigns` | `listCampaigns` (`campaigns-segment-npf`), `getInstantlyCampaignsList` (`campaigns-segment-instantly`) | `campaigns_list`, `instantly_campaigns_list` |
| `/campaigns/npf` | `listCampaigns` (`campaigns,q,page,source`), `createCampaign` | `campaigns_list`, `create_campaign` |
| `/campaigns/:id` | `getCampaignDetail` (`campaign-detail,id`), `listCampaignRecipients` (`campaign-recipients,...`), `listCampaignContacts` (`campaign-contacts,id`), `listCampaignLinks` (`campaign-links,id`), `getInstantlyCampaignSummary` (`instantly-summary,id`); mutations `setCampaignSource`, `mapNpfJob`, `importNpfReport`, `generateTrackingLinksBulk`, `generateTrackingLinksForCodes`, `resolveTrackingUrl`, `getCampaignPreflight`, `getCampaignExportRows`, `mapInstantlyCampaign` | `campaign_detail`, `campaign_recipients`, `campaign_contacts`, `campaign_links_list`, `instantly_campaign_summary`, `set_campaign_source`, `map_campaign_job`, `import_npf_report`, `generate_tracking_links_bulk`, `generate_tracking_links_for_codes`, `resolve_three_utm`, `campaign_preflight_self`, `campaign_export_rows` |
| `/campaigns/instantly` | `getInstantlyCampaignsList` (`instantly-campaigns-list`), `listEmailTemplates` (`email-templates`), `saveEmailTemplate`, `deleteEmailTemplate`, `checkContactRows` | `instantly_campaigns_list`, `list_email_templates`, `save_email_template`, `delete_email_template`, `check_contact_rows` |
| `/campaigns/instantly/templates/:id` | `listEmailTemplates` (`email-templates`), `saveEmailTemplate` | `list_email_templates`, `save_email_template` |
| `/sources` | `getSourcesSummary` (`sources-summary`) | `sources_summary` |
| `/data-health` | `getDataHealth` (`data-health`), `getInstantlyUnmapped` (`instantly-unmapped`) | `data_health`, `instantly_unmapped` |
| `/settings` | `updateMyTimezone`; admin: `getInstantlyIntegration`, `rotateInstantlyWebhookToken`, `setInstantlyWorkspace`, `saveInstantlyApiKey` | Auth `PUT /auth/v1/user`; `get_instantly_integration`, `rotate_instantly_webhook_token`, `set_instantly_workspace`, `save_instantly_api_key` |
| `/profile` | `updateMyProfile` | Auth `PUT /auth/v1/user` |
| `/admin/members` | `listMembers` (`brain-members`), `setMemberAccess` | `admin_list_members`, `admin_set_member_role` |
| topbar `NotificationBell` (lead+) | `getDataHealth` (`notif-data-health`), `getInstantlyUnmapped` (`notif-instantly-unmapped`) | `data_health`, `instantly_unmapped` |
| `/oauth/consent` | `getMyBrainAccess` + client-side `auth.oauth.*` | `my_roles` |
| every `/_app/*` load | `getMyMembership` | `my_roles` |

Note the bell and the dashboard each call `data_health` with their own query keys, so a lead/admin page load can issue `data_health` up to three times (dashboard, bell, data-health page). Harmless but worth knowing when reading Brain logs.

---

## 14. RPC envelope reference (from `src/lib/campaigns-data.ts` and `brain-research.ts`)

### 14.1 `CampaignSummaryRow` (rows of `campaigns_list`, `campaign` of `campaign_detail`)
`campaign_uuid, campaign_code, name, programme, attribution_source_slug|null, status, historical_import, npf_job_count, npf_recipient_job_records, npf_unique_recipient_emails, delivered, opened, clicked, bounced, dropped, provisionally_attributed_applications, attributed_referring_contacts, recorded_enrollments, payment_currency|null, recorded_payment_events, recorded_net_payment_minor, send_date|null, email_subject|null, campaign_medium, campaign_owner_id|null, external_reference|null, created_at, npf_job_ids: string[]|null`.

UI-derived "tracking health" (`_app.campaigns_.npf.tsx`): `npf_job_count === 0` -> "No NPF data"; no source slug -> "Source not mapped"; `npf_recipient_job_records === npf_unique_recipient_emails > 0` -> "Operational"; else "Partial".

### 14.2 `CampaignDetail`
`{ campaign: CampaignSummaryRow, npf_jobs: [{job_id, template_name, message_type, first_seen_at, last_seen_at, mapping_method}], reporting: {processed, delivered, opened, clicked, bounced, dropped, invalid, failed, total_recipient_job_records, unique_emails, matched_contacts, unmatched_records, ambiguous_records}, latest_report_at }`.

### 14.3 `CampaignRecipientRow` (NPF recipient-job record)
`id, job_id, recipient_name, recipient_email, template_name, status, match_status, contact_id, contact_code, contact_name, designation, school_id, school_name, communication_at_local, processed_at_local, delivered_at_local, opened_at_local, clicked_at_local, bounced_at_local, dropped_at_local, invalid_at_local, failed_at_local, vendor_response, updated_at`. The `*_at_local` columns are NPF's local-time strings (the import accepts `p_source_timezone`).

### 14.4 `DataHealth` (18 counts)
`campaigns_total, campaigns_with_npf_jobs, campaigns_without_npf_jobs, campaigns_missing_source_slug, npf_jobs_total, npf_jobs_unmapped_to_campaign, npf_recipient_reports_total, npf_recipient_reports_matched, npf_recipient_reports_unmatched, npf_recipient_reports_ambiguous, campaign_recipients_native_total, campaign_links_total, applications_total, applications_without_attribution, enrollments_total, payment_events_total, schools_total, contacts_total, generated_at`.

### 14.5 `SourceSummaryRow`
`source_slug ("(not mapped)" for unmapped), campaigns, delivered, opened, clicked, unique_contacts_reached, applications, enrollments, revenue_by_currency: [{currency, net_minor}]`.

### 14.6 Tracking-link family
- `CampaignLinkRow`: `link_id, link_kind, destination_url, is_active, created_at, campaign_recipient_id, contact_id, medium, audience_status, contact_code, contact_name`.
- `GenerateLinkResult`: ok -> `campaign_recipient_id, campaign_link_id, tracking_url, campaign_code, contact_code, source_slug`; failures `access_denied | invalid_link_kind | invalid_channel | campaign_not_found | source_not_mapped | contact_not_found | invalid_base_url`.
- `GenerateLinksBulkResult`: ok -> `generated, failed, samples[], not_found?[]`; failures `access_denied | campaign_not_found | source_not_mapped | invalid_input`.
- `TrackingResolution.status`: `ok | access_denied | invalid_input | ambiguous_or_unrecognized_format | campaign_not_found | contact_not_found`; ok carries `resolver_version ("v2_legacy"|"v3")`, `campaign{campaign_uuid,campaign_code,name,programme,status,attribution_source_slug}`, `contact{contact_id,contact_code,contact_name,designation,researcher_names[]}`, `school{school_id,name,country,city}|null`, `source_slug`.
- `CampaignExportRow`: `email, recipient_name, school_name, campaign_uuid, campaign_code, contact_uuid, contact_code, source_slug, tracking_url, link_kind` (the CSV writer deliberately omits both UUIDs).
- `CampaignPreflight`: `status ok|access_denied|not_found`, `ready`, `blockers[]`, `shared_outreach_conflicts`, `npf_contact_risk{unsubscribed_recipients, prior_bounce_recipients, prior_dropped_recipients}` + passthrough keys.

### 14.7 Campaign/job mutations
- `CreateCampaignResult`: ok -> `campaign_id, campaign_code, attribution_source_slug|null, steps, sending_enabled:false`; `access_denied | invalid_input | invalid_source_slug`.
- `SetCampaignSourceResult`: ok -> `campaign_id, attribution_source_slug`; `access_denied | invalid_source_slug | not_found | immutable_after_launch`.
- `MapNpfJobResult` / `MapInstantlyCampaignResult`: ok -> `job_id|external_campaign_id, campaign_id`; `access_denied | invalid_input | npf_job_not_found | campaign_not_found | already_mapped_to_another_campaign (+existing_campaign_id)`.
- `ImportNpfReportResult`: `{result:"completed", import_id, total_rows, inserted, updated, unchanged, stale, file_sha256}` | `{result:"identical_file_already_imported", file_sha256, import_id}` | `{status:"access_denied"|"error", message?}` (note the `result` vs `status` key difference – the handler checks `"status" in e`).

### 14.8 Instantly
- `InstantlyCampaignSummary`: `status`, `linked_external_ids[]`, `events{sent, opened, clicked, bounced, replied, unsubscribed, meetings_booked, campaign_completed_events, unique_recipients, total_events}`.
- `InstantlyCampaignRow`: `external_campaign_id, sent, opened, replied, last_event_at, campaign_id|null, campaign_name|null, campaign_code|null`.
- `InstantlyUnmappedRow`: `external_campaign_id, events, last_event_at`.
- `DeliverabilityOverview` (`outreach_overview`): `window_days, provider_health[{provider, last_event_at, events_in_window}], totals[{provider, event_category, events, contacts}], suppressions{<reason>: count}`.
- `InstantlyIntegration` (settings): `configured, enabled, workspaceId, webhookUrl, lastEventAt, totalEvents, hasApiKey`.

### 14.9 Templates and contacts
- `EmailTemplate`: `id, name, subject, body_html, design_json|null, created_at, updated_at`; `SaveEmailTemplateResult`: ok -> `id`; `access_denied | invalid_input | not_found`.
- `ContactCheckRow.status`: `existing` (contact_id, contact_code, contact_name, school) | `new` | `invalid_email` | `skipped_missing_name` | `created` (same fields as existing); `ContactCheckResult`: `committed, existing_count, new_count, invalid_count, rows[]`.

### 14.10 Research (caller-scoped)
- `my_research_dashboard` -> `authenticated_user_id, own_total_submissions, own_pending_submissions, own_active_claims, own_recent_submissions[{id, submission_kind, submitted_school_name, submitted_country, submitted_city, review_status, created_at, source_url}]`.
- `school_coverage_self` -> `total, offset, has_more, warning, historical_schools[{source_record_id, name, city, country, ...}]` (parser keeps only the four named fields).
- Overview: `status` -> `schools, school_contacts`; `research_metrics` -> `verified_named_contacts, worker_jobs_pending, worker_jobs_running, submission_evidence_unreviewed`.

---

## 15. Three-UTM tracking scheme

| UTM | Value | Example shape | Validated by |
|---|---|---|---|
| `utm_campaign` | **CampaignCode** (human-safe campaign identifier; the UUID is never exposed) | shown as `Code <campaign_code>` in the header | `resolve_three_utm` campaign lookup |
| `utm_medium` | **source slug** = `attribution_source_slug` of the campaign (researcher/data segment, e.g. a researcher-plus-dataset slug) | `^[a-z0-9-]{3,96}$` (client zod `min(3).max(96)`, DB `invalid_source_slug`) | immutable after launch |
| `utm_source` | **ContactCode** | `^CT[A-Z0-9]{10}$` (textarea placeholder shows two such codes) | resolver detects v3 vs v2 by which parameter matches this pattern |

- Built server-side by `three_utm_url_v3` from a caller-supplied destination (`p_base_url`, zod `.url()`), a `link_kind` (`calendly | apply | payment | programme | other`, default `programme`; the by-codes UI uses `apply`) and a `channel` (`email | whatsapp | sms | call | social | in_person | other`, default `email`).
- Legacy links (`three_utm_url_v2`) had the medium/source mapping swapped; the resolver returns `resolver_version: "v2_legacy"` for those and `"v3"` for new ones, so no flag is needed in the URL.
- A generated link creates a `campaign_recipients` row (audience membership, `audience_status`) and a `campaign_links` row (`is_active`). Links can only be generated once the campaign has a source slug (`source_not_mapped`).
- Email templates reference the per-recipient link only as `{{tracking_url}}`; humans never type a UTM.

---

## 16. Middleware and security headers

`src/start.ts`:
- `errorMiddleware` – catches non-HTTP errors from any request handler, logs via `console.error` (which `src/lib/error-capture.ts` monkey-patches to expand Error causes/stacks), returns the static HTML error page.
- `createCsrfMiddleware({ filter: ctx => ctx.handlerType === "serverFn" })` – explicit because defining `start.ts` disables the default.
- `attachSupabaseAuth` (client-side function middleware) – reads the already-initialised Brain client's session and adds `Authorization: Bearer <access_token>` to every server-fn call; no token -> no header -> `requireBrainAuth` throws `Unauthorized`.

`src/server.ts` (custom SSR entry declared in `vite.config.ts`):
- Normalises h3's swallowed `{"unhandled":true,"message":"HTTPError"}` 500 JSON into the HTML error page and logs the captured original error.
- Headers on every response: `Content-Security-Policy: frame-ancestors 'self' https://*.lovable.app https://*.lovable.dev https://lovable.dev; base-uri 'self'; object-src 'none'` (consent paths: `frame-ancestors 'none'` + `X-Frame-Options: DENY`), `Referrer-Policy: no-referrer`, `X-Content-Type-Options: nosniff`, `Strict-Transport-Security: max-age=31536000; includeSubDomains`.
- All `pageHead()` routes add `robots: noindex`; `public/robots.txt` nevertheless allows all crawlers (only the marketing-free auth pages are reachable unauthenticated).

---

## 17. Local development and operations

```sh
bun install                 # NOT npm i (overrides/dependencies conflict; README is stale)
# .env: LWL_BRAIN_PUBLISHABLE_KEY=<publishable key of <BRAIN_REF>>  (server-side only)
bun run dev                 # vite dev via @lovable.dev/vite-tanstack-config
bun run test                # vitest run (11 files)
bun run lint && bun run format
bun run build               # nitro build (cloudflare default target)
```

- Sign-in requires an existing Brain Auth user **and** an active `brain_ops.member_roles` row; otherwise `/access-denied`. There is no self-signup; admins approve from `/admin/members`.
- Adding a page: create `src/routes/_app.<name>.tsx` with `beforeLoad: guard("<section>")`, add the section to `SECTIONS` in `src/lib/permissions.ts`, add a nav item in `app-sidebar.tsx`, and a title in `_app.tsx` `titles`. Add a server fn that calls `authorizeBrain(context, "<section>")` and an RPC gated identically in the DB.
- Adding a Brain RPC: follow `docs/01-supabase/brain-rpc-pattern.md`, then add its envelope type to `campaigns-data.ts` (or a parser in `brain-research.ts` for caller-scoped data) and a server fn in the matching `*.functions.ts`.
- Supabase MCP: `.mcp.json` targets Brain; authenticate per user via OAuth inside Claude Code (`/mcp`). Note the user's claude.ai Supabase connector used for this documentation pass had access to the org that owns `<DATA_REF>` and `<ESTATE_REF>` but **not** to `<BRAIN_REF>`; Brain is in a different Supabase organization or account.

---

## 18. Lovable planning history vs. what shipped

| Plan (`.lovable/plan/`) | Planned | Shipped / changed |
|---|---|---|
| 2026-10-26 identity portal | `/`, `/login`, `/reset-password`, `/auth/callback`, `/oauth/consent` (+ `/.lovable/...`), `/account`, `/access-denied`; membership from a `research_members` table on **Master Research** (`txml...`) via a server-only key; white/navy/teal design; custom domain `research-auth.example-group.com` | Routes shipped as planned; membership source moved to Brain `brain_ops.member_roles` via `brain_api_my_roles` on 10-29; privileged key removed; design later replaced by dark violet; domain not verifiable |
| 2026-10-27 role-based dashboard | `_app` layout, Overview/Schools/Contacts/Research/Outreach/Programs/Revenue/Members/Profile, `permissions.ts`, `data.functions.ts` using `supabaseAdmin` with limit-50 selects | Layout + pages shipped; `supabaseAdmin` disabled at cutover; Programs/Revenue remain stubs and are hidden from nav; Outreach exists but hidden |
| 2026-10-29 team access & approvals | rename to Team Access & Approvals, pending vs active members, confirm-gated approve/remove, admin grant separate | Shipped (`team-access.tsx`, tests), plus the ChatGPT onboarding wizard |
| (no plan file) 11-05/11-06 | – | Campaigns/Sources/Data Health, three-UTM, NPF import/export, Instantly, templates, settings, theme – all authored locally with Claude Code and pushed (25 commits by `nishanta-code`) |

Still-open roadmap gates (`roadmap.md`): verify real Brain staff membership/RLS/reset/consent in production; enable programme/revenue/evidence capabilities once Brain exposes verified RPCs; editor-managed Supabase binding switch (cosmetic – runtime is independent).

---

## 19. Status-string glossary (every `status`/`result` value the app handles)

`ok`, `access_denied`, `invalid_input`, `not_found`, `cannot_remove_own_admin`, `invalid_source_slug`, `immutable_after_launch`, `invalid_link_kind`, `invalid_channel`, `campaign_not_found`, `source_not_mapped`, `contact_not_found`, `invalid_base_url`, `npf_job_not_found`, `already_mapped_to_another_campaign`, `ambiguous_or_unrecognized_format`, `error` (NPF import), and NPF import `result` values `completed`, `identical_file_already_imported`. Role-check outcomes on the app side: `approved`, `none`, `unavailable`, `inactive` (legacy `decideMembership`), and research parser states `ok`, `denied`, `unavailable`.

---

## 20. Open items / could not determine from this vantage point

- Full SQL of every RPC created 11-05/11-06 (applied via MCP, never committed) – including the exact allowed values of the `action_audit.workstream` CHECK constraint beyond `'admin'` and `'campaign'`, the `member_roles` role CHECK list, `guard_campaign_tracking_identity_v2` body, and the pg_cron job name for suppression sync.
- Brain's RLS policies (if any) outside the five tables named in commits; Brain edge functions other than `brain-mcp`; any Brain storage buckets.
- Whether the Lovable project is still auto-deploying from `main` and whether a custom domain exists.
- The relationship between `<DATA_REF>` research tables and Brain's (section 7).
