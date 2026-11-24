# Auth and identity across the estate

Sources: live `pg_trigger`/`pg_policies` queries on `<ESTATE_REF>` (2026-11-23/24), `growth-partner-portal/docs/shared_architecture_knowledge.md` ("Auth and security"), `learner-portal/mem/tech/auth-session-policy.md`, the 2026-11-23 shared-auth hardening work (Pre-Med Platform `premed-student-register`, Growth Partner Portal `affiliate-register-partner` / `affiliate-admin-signup`), Counselling CRM migrations, each repo's `src/lib/auth.tsx` / `src/lib/session.tsx`.

---

## 1. Which bucket is a login in?

| Bucket | Projects | Consequence |
|---|---|---|
| Shared estate `auth.users` (`<ESTATE_REF>`) | Ops Tool, Application Portal, Pre-Med Platform, Growth Partner Portal, Challenge Platform, LMS/Learner Portal | one email = one account across all six; a password set in Ops Tool works on the LMS login page |
| Own-project auth, estate data | Financial Literacy Platform, Pre-Law Platform, Fellowship Platform, Research Platform, Test-Prep Platform, Purpose Platform, Founders Platform | the browser authenticates against the platform's own small project; its edge functions / server functions reach the estate with a service-role key |
| Standalone | Builders Lab, Counselling CRM, Sales Command (original project), Research & Outreach (Brain) | nothing is shared |

Never assume "set once in Ops Tool, works everywhere". Check the repo's client file first.

---

## 2. Identity mechanics on the shared estate

- **No trigger on `auth.users`.** Verified via `pg_trigger` — zero triggers. Profile provisioning into each schema's own table (`ops.profiles`, `lms.student_profiles` + `lms.user_roles`, `premed.user_profiles`, `portal.user_profiles`, `affiliate.partners`/`affiliate.admins`) is 100% app-side and independent per platform. One shared login does not create a profile anywhere else.
- `auth.users.user_metadata` (e.g. `full_name` set at signup) is visible to every platform's session — read it to pre-fill a new profile on first login instead of building a shared profile table.
- A prior attempt at a shared `public.has_role()` / cross-platform registry **broke and was abandoned**. Do not re-propose it. Roles are schema-scoped: `lms.has_role(uid,'admin')` means admin of the Learning Portal only. Put `has_role` in a schema not exposed through the Data API and schema-qualify every call; pin `search_path = <schema>, public` on every function.
- RLS on the profile tables sampled (`premed.user_profiles`, `ops.profiles`, `lms.student_profiles`, `portal.user_profiles`) is correctly scoped to `auth.uid() = owner column`.
- Migration files are not proof a trigger exists. LMS and Pre-Med Platform migration history both show triggers that no longer exist live. Always check `pg_trigger`.

### Central auth config (verified for the LMS, applies project-wide)

`jwt_exp` 3600; refresh-token rotation on with 10s reuse interval; `sessions_timebox` 0; `sessions_inactivity_timeout` 0; `sessions_single_per_user` false. Do not enable timebox, inactivity timeout or single-session-per-user — the Learner Portal relies on indefinite multi-device sessions.

---

## 3. The safe account-creation convention (server-side, never bare `signUp()`)

The proven pattern, now standard across the estate:

1. **Point lookup before create** — a small `SECURITY DEFINER` RPC `<schema>.auth_user_id_by_email(text)` doing a case-insensitive indexed lookup on `auth.users`. Never page `auth.admin.listUsers()` (does not scale to public traffic). Exists today as `premed.auth_user_id_by_email`, `affiliate.auth_user_id_by_email`, the LMS equivalent used by `student-register`, Application Portal's `portal-admin-api` (`findAuthUserByEmail`), Ops Tool's `ops-admin-users`. EXECUTE is granted to `service_role` only (`has_function_privilege` verified: anon=false, authenticated=false).
2. If the email exists, **link** to the existing identity (provision the platform's profile row) instead of creating a duplicate.
3. Rate limiting by IP + email hash (`<schema>.registration_attempts` tables: `lms.registration_attempts`, `premed.registration_attempts`, `affiliate.registration_attempts`; Application Portal uses `portal.rate_limit_hits`).
4. Honeypot field + minimum fill time; Cloudflare Turnstile on the LMS.
5. **Neutral anti-enumeration messages** — every failure collapses to one public message; the real reason is logged server-side.
6. Rollback on partial failure (delete the auth user if the profile insert fails).
7. Self-registered accounts start unprivileged (`email_verified:false`), and the LMS enrollment engine "reclaims" them (forced password reset + session revocation) before attaching them to a real programme. Self-claimed identity is never trusted on its own.
8. Block staff-domain emails from becoming student accounts (LMS).
9. Bulk onboarding of pre-existing students goes through a throttled queue (`lms.login_rollout_settings` / `login_rollout_queue`, 4.2k rows) rather than all at once.

Reference implementations (read-only):
- `learner-portal/supabase/functions/student-register/` — the fullest version.
- `premed-platform/supabase/functions/premed-student-register/` and `create-user/` (admin provisioning aligned to lookup-before-create on 2026-11-23).
- `growth-partner-portal/supabase/functions/affiliate-register-partner/` and `affiliate-admin-signup/` (admin signup additionally gated on a pre-provisioned `affiliate.admins` row).
- `ops-tool/supabase/functions/ops-admin-users/` — staff accounts, no public self-signup.
- `counselling-crm/supabase/functions/manage-team/` + `src/lib/team.functions.ts` — invite/update_role/remove, role-checked server-side; public signup disabled project-wide via Management API `disable_signup: true`.

---

## 4. Browser session patterns

- Supabase JS client with `persistSession`; LMS chooses storage by "Keep me signed in": remembered = `localStorage`, tab-only = `sessionStorage` (`src/integrations/lms/client.ts`, storage key `lms-auth`).
- Every `signOut()` on the LMS passes `{ scope: "local" }` (global scope logs the user out of every device); the only exception is Admin Settings' explicit "Sign out everywhere".
- Never sign out on a transient error (failed `user_roles` lookup) — deny the route instead. Never sign out on `pagehide`/`beforeunload` (a refresh fires them).
- `App.tsx` on the LMS revalidates the session on `visibilitychange`, `focus`, `online`, cross-tab `storage` events, plus a 5-minute interval. `RequireRole` retries `getSession()` once after 800ms to survive token-rotation races.
- Route guards: `RequireRole`/`RoleLayout` (LMS), `ProtectedRoute` + `portal.user_profiles` role (Application Portal admin), `session.tsx` with `affiliate|school|admin` roles (Growth Partner Portal), server middleware `attachCrmAuth` + `requireSupabaseAuth` on TanStack Start (Counselling CRM), `permissions.ts` + member roles (Research & Outreach).
- Realtime subscriptions inside `useEffect` must use a unique channel topic per effect run (`notifications:${userId}:${random}`) — a fixed topic collides under React's dev double-invoke and `.on()` on an already-subscribed channel throws (crashed a whole route in Counselling CRM CRM).

---

## 5. Edge-function auth models (what "verify_jwt=false" really means)

All estate functions are deployed with `verify_jwt=false` and do their own auth in code. The four shapes:

| Model | Used by | How |
|---|---|---|
| Bearer + role lookup | admin functions (`portal-admin-api`, `central-admin-v1`, `ops-admin-users`, `manage-team`, `admin-manage-user`) | read `Authorization: Bearer`, `auth.getUser()` on the auth project, then look up the caller's role in the platform's `user_profiles`/`user_roles` via a service-role client; 403 unless on the allow-list. Role checks live in function code because service-role bypasses RLS. |
| Anonymous + unguessable token + rate limit | public checkout/intake (`create-payment-intent`, form submits, masterclass registration) | no bearer; a UUID `submission_token` plus per-IP/per-token rate limiting; all money math server-side |
| Webhook signature | `stripe-webhook` (`constructEvent`), Razorpay HMAC, Flywire, Calendly, Zoom, WhatsApp provider callbacks | verify signature with `timingSafeEqual`; idempotent on provider event id |
| Cron secret | scheduled jobs invoked by `pg_cron` + `pg_net` | `x-cron-secret` header must match a secret; the Challenge Platform reminder job failed every 15 minutes for lack of this header |

---

## 6. Hardening history (what has been fixed, what is open)

- 2026-11-23: `portal.masterclass_form_templates` had RLS **disabled** → enabled with 5 policies mirroring `form_templates`. Pre-Med Platform' bare client `signUp()` (dead code at the time) replaced by `premed-student-register`. Growth Partner Portal partner/admin signup moved server-side. Stale `affiliate.partners` comment fixed.
- 2026-11-21 (Counselling CRM): every custom function's EXECUTE revoked from PUBLIC (Postgres grants EXECUTE to PUBLIC by default — `notify_owners_and_admins` was callable via PostgREST RPC with arbitrary text); bare `using (true)` policies replaced; `user_roles` made the sole role authority with a `guard_role_change` trigger ("last owner can't be removed"); minimum password length raised 6→8.
- 2026-10-15 (Growth Partner Portal): Lovable's cleanup removed 22 unused packages and cleared stale storage policies.
- **Open**: leaked-password protection (HaveIBeenPwned) is a paid-tier Supabase feature (HTTP 402 on the free Counselling CRM project); on the estate it is a dashboard toggle (Authentication → Providers) still reported off by the master doc. Two storage buckets (`announcement-media`, `ticket-attachments`) public with listing allowed. A per-function `verify_jwt` review for the admin-named functions was recommended and not yet recorded as done.
