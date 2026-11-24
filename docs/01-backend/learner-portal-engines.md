# Learner Portal: the enrollment, credit and referral engines

The Learner Portal (schema `lms` on the estate) is the shared post-purchase platform for
students of every brand. It is the most mature codebase in the estate: ~110 tables, ~82 edge
functions, a unit-tested state machine for enrollment, a double-entry credit ledger, and a
referral programme with real anti-abuse rules.

Paths are relative to `learner-portal/`.

---

## 0. Client and auth wiring

`supabase/functions/_shared/lms.ts` is the single factory:

```ts
export const LMS_URL = Deno.env.get("EXT_SUPABASE_URL")!;
export const LMS_SERVICE_KEY = Deno.env.get("EXT_SUPABASE_SERVICE_KEY")!;
lmsAdmin()      // service role, db.schema = "lms"
catalogAdmin()  // service role, public (shared catalogue)
requireAdmin(admin, token)    // getUser(token) then lms.user_roles has role='admin' → else 401/403
requireUser(token)            // any signed-in user → { userId, email }
requireCronOrAdmin(req, admin)// header x-service-key compared with safeEqual(), or an admin bearer
isAdminUser(admin, userId)
```

Three things to copy from this file: the **role check is a table lookup, not a JWT claim**; the
cron gate uses a **constant-time compare** (`_shared/safe-equal.ts`); and the browser client is
a *different* file (`src/integrations/lms/client.ts`) pinned to `db.schema: "lms"` — the
auto-generated `src/integrations/supabase/client.ts` points at a dead project and must not be
imported (one test still does).

### The edge-function namespace migration

`docs/lms-edge-functions.md` documents a careful rename of 40 endpoints to a `lms_<name>`
namespace on a shared project where other platforms also deploy functions:

1. Deploy the 40 canonical endpoints with their existing in-code auth.
2. Rewire only the existing cron jobs (one migration), creating no new jobs.
3. Publish the client with the updated transport, which prefixes logical names **exactly once**
   (double-prefix prevention is unit-tested); older cached clients keep working against the 40
   retained legacy endpoints.
4. Verify signed-in student/admin/mentor flows and watch legacy traffic before deleting
   anything. External callbacks may still use old names.

The rule that governs the whole cutover: **no retry/fallback is added, because an ambiguous
failure must never duplicate passwords, enrollment actions, emails, WhatsApp sends or wallet
writes.** Schedules keep their original ids, cadence and vault-key references.

---

## 1. The enrollment engine

### Shape

| Layer | File | Responsibility |
|---|---|---|
| Pure state machine | `_shared/enrollment-rules.ts` | `decide()` — mirrors the plpgsql authority, dependency-free, unit-tested |
| Orchestrator | `_shared/enrollment-engine.ts` | `applyEnrollmentEvent()` — cohort resolution, account resolution, post-commit messages; all I/O injected via `EngineDeps` |
| Injected I/O | `_shared/enrollment-deps.ts` | account creation, password generation, mailer, last-email status, portal-access email |
| Pure planning for the sweeper | `_shared/reconcile-rules.ts` | which application decides an (email, cohort) pair; safety guards |
| Sweeper | `_shared/reconcile.ts` | runs the plan against live data |
| Source adapters | `_shared/applications.ts` | `ADAPTERS` — the per-platform application tables to read, column maps, cohort resolution, team-email detection |
| DB authority | `lms.apply_enrollment_change` (plpgsql) | writes the state change **and** its audit row atomically |

**Every path goes through one function.** Provision, admin action, webhook and reconcile all
call `applyEnrollmentEvent()`; nothing writes `lms.enrollments` directly.

### The state machine

Event types: `paid`, `pending`, `refunded_full`, `refunded_partial`, `admin_enroll`,
`admin_unenroll`, `admin_release_override`.
Sources: `webhook`, `reconcile`, `admin`, `provision`.

Actions it can return — note how much of the design is in the vocabulary:
`enrolled`, `upgraded_from_pending`, `unchanged`, `pending_created`,
`pending_ignored_already_paid`, `refund_revoked`, `partial_refund_logged`, `admin_enrolled`,
`admin_unenrolled`, `override_released`, `skipped_pinned`, `stale_ignored`, `no_enrollment`,
`no_student`.

Invariants encoded there:
- **`manual_override` pins a row.** An admin decision is never overwritten by automation
  (`skipped_pinned`), until an explicit `admin_release_override`.
- **Events carry `occurred_at`**, and an event older than the row's `last_event_at` is
  `stale_ignored` — out-of-order webhook deliveries cannot regress state.
- **Idempotency** via `external_event_id`; a repeat returns `duplicate: true`, `written: false`.
- A partial refund is **logged**, not acted on.
- `pending` never downgrades a paid enrollment.

Enrollment events are append-only (`lms.tg_enrollment_events_immutable`).

### The reconcile sweeper (automatic enrollment)

`_shared/reconcile-rules.ts` holds the guards as constants, which is the point:

```ts
export const LMS_LAUNCH_DATE = "2026-09-12";   // paid applications only auto-enroll into
                                               // cohorts starting on/after portal launch
export const MAX_UNENROLL_SHARE = 0.05;        // one run may unenroll at most 5% of active rows
```

`pickApplication(rows)` chooses the deciding application for one (email, cohort): it normalises
payment status, and **if both a paid and a refunded row exist it returns `{ conflict }` rather
than guessing**. Otherwise it prefers paid, then refunded, then pending, newest `paid_at` first.

Operational controls live in `lms.enrollment_settings` and are edited only through
`enrollment-automation` (admin-gated):
`auto_enroll_enabled`, `auto_unenroll_enabled`, `dry_run`, `max_changes_per_run` (1–25),
`alert_email`. **Turning anything live requires the caller to type the literal word `LIVE`,
checked on the server** (`confirm !== "LIVE"` → 400 `confirm_required`). Run history and errors
are visible in `lms.enrollment_sync_runs`, `enrollment_sync_items`, `enrollment_errors`,
`enrollment_runs`, `enrollment_run_items`, `enrollment_events`.

### Login rollout — the throttled onboarding line

`login-rollout/index.ts` is the single sending line for portal logins:
- `tick` (scheduler every 2 minutes, or an admin) sends **at most one email**, and only when
  `lms.claim_login_rollout_item()` says so: switch on, minimum gap, 6-hour and 24-hour caps.
- `status / preview / build / start / pause / clear_waiting` are admin-only.
- Enrollments still go through the engine; this function never writes an enrollment.
- Supporting SQL: `claim_login_rollout_item`, `defer_login_rollout_item`,
  `cooldown_login_rollout`, `finish_login_rollout_item`; state in
  `lms.login_rollout_settings` / `login_rollout_queue` (thousands of rows).
- A `LOGIN_ONLY_TABLE` exists for a platform whose applicants get a portal login but never a
  programme — kept out of `ADAPTERS` deliberately.
- Addresses are auto-corrected for obvious mistakes (spaces, trailing dots, misspelt
  big-provider domains) before sending.
- Cohort holds (`_shared/cohort-holds.ts`) and unused-portal-cohort detection
  (`_shared/portal-cohorts.ts`) stop mail for cohorts that are not ready.
- Onboarding packets are enqueued in batches (`_shared/onboarding-batches.ts`).

### Self-registration and account claiming

`_shared/registration.ts` is the reference implementation for public signup anywhere:

```ts
export const NEUTRAL_MESSAGE = "We couldn't create an account with those details. …sign in instead or reset your password.";
export const LIMITS = { perIpPerHour: 5, perEmailPerHour: 3, minFormMs: 3000 };
```
- Honeypot field (`website`) — any value rejects with the neutral message.
- `form_loaded_at` must be at least 3 seconds ago.
- Name 2–100 chars, must contain a letter, rejects `< > { } @ \ http:`.
- Strict email regex, ≤ 254 chars, no `..`; **team-domain emails are refused** for student
  accounts (`isTeamEmail`).
- Password 8–72 chars and never equal to the email; never echoed back.
- Policy acceptance mandatory, with a recorded `policy_version`.
- Rate-limit keys are **salted HMAC-SHA256 hashes** so raw IPs and emails are never stored.
- Cloudflare Turnstile token verified server-side.
- Duplicate check uses `lms.auth_user_id_by_email` (indexed point lookup), never `listUsers`.
- Self-registered accounts are unprivileged; `claim-student-account` links an existing identity,
  and the engine later "reclaims" the account (forced reset + `lms.revoke_user_sessions`) before
  attaching it to a real programme.

### A real reconciliation audit (how to run one)

`docs/student-reconciliation-20261116.md` is a model artefact: a read-only SQL audit across
applications, auth, profiles, enrollments, catalogue and section membership, with four verified
repairs applied through the engine's atomic RPC and **no password changes or manual sends**. It
reports exact counts (4,578 applications → 2,722 paid / 1,723 pending / 133 refunded; 2,629
distinct paid accounts; 88 paid programme/cohort codes), proves "zero paid missing-access
pairs" after repair, and then lists **every exception that needs a human** — conflicting cohort
targets, a paid code with no catalogue cohort, typo'd email domains (`gmail.con`) that must not
be merged without verified ownership, legacy seats with no current application that are
retained rather than treated as refunds. The closing section names the limitation honestly
(shared adapters still read legacy tables; payment precedence unchanged).

The rule it establishes: **absence of an application is not evidence of a refund**, and identity
merges are never inferred.

---

## 2. The Vault (credits / loyalty)

A full double-entry credit system. Tables: `vault_settings` (+ `vault_settings_history`),
`vault_lots` (earned credit with expiry), `vault_ledger` (immutable entries),
`vault_vouchers`, `vault_fx_rates`, `vault_review_flags`, `vault_walkthrough_seen`,
`birthday_grants`.

### Earning
Trigger-driven, so earning is a consequence of real events, never an API call:
`tg_vault_cashback` (cashback on payment, rates per delivery mode: online / in-person / camp),
`tg_vault_earn_attendance`, `tg_vault_earn_feedback`, `tg_vault_earn_project`,
`tg_vault_cohort_completed`, `tg_vault_referral_reward`, `t_vault_welcome_on_profile`
(`vault_grant_welcome`), `vault_birthday_due`.
Settings expose the whole economy as data: `cashback_rate_online/in_person/camp`,
`effort_profile/session/project/completion`, `max_rewarded_projects_per_cohort`,
`level_thresholds`, `level_caps`, `loyalty_expiry_months`, `effort_inactivity_months`,
`voucher_valid_hours`, `referral_multiplier`, `referral_release_days`, and the kill switches
`vault_enabled`, `redemption_enabled`, `allow_free_programs`, `restore_credit_on_refund`.

### Spending
`vault-redeem` (student, `requireUser`) → SQL `vault_quote` / `vault_quote_program` /
`vault_create_voucher` / `vault_create_voucher_program`, all **under a wallet lock**, with the
amount re-quoted and capped in SQL. The caller is always the signed-in student; nothing about
identity or amount is trusted from the body. User-facing error strings are mapped from codes
(`redemption_off`, `not_found`, `already_paid`, `deadline_over`, `no_price`, …).

Lifecycle: `_vault_reserve` → `_vault_release_voucher` / `vault_cancel_voucher` /
`vault_expire_vouchers` → `vault_commit_voucher_from_redemption` once the payment page reports
a real redemption. `vault_reverse` handles refunds; `vault_release` releases held credit.

### The checkout contract (`docs/vault-checkout-contract.md`)
A coupon is a **single-use row in the central `public.promo_codes` table** (code `VAULT…`,
`max_redemptions = 1`, tied to one programme, `expires_at` set). The portal appends
`?promo=<CODE>`. Every `/pay` page must, before charging:
1. accept the code only if `is_active`, `redemption_count < max_redemptions`, now is inside
   `starts_at`/`expires_at`, and `program_id` matches;
2. apply exactly `discount_value` (fixed, USD);
3. `select … for update` the promo row while charging so two tabs cannot both use it;
4. after payment, insert `public.promo_redemptions` with `customer_email`, `amount_before`,
   `amount_after`, `payment_intent_id`.

The LMS then marks the coupon used automatically; if email or discount do not match, the credit
stays held and admins get a review flag. **`redemption_enabled` stays OFF until every payment
page implements steps 1–4** — a feature flag gating a cross-repo contract.

### Integrity
`vault-daily` (cron or admin) expires due credit, sends 30-day expiry warnings and runs
`vault_integrity_report`, emailing admins **only when balances do not reconcile**. Ledger and
lots are immutable (`tg_vault_ledger_immutable`). Admin adjustments go only through
`vault-admin-actions` (`vault_adjust`, `update_vault_settings`), which are `service_role`-only
SECURITY DEFINER functions — the edge function is "the only door, and it verifies the admin role
on every call".

---

## 3. The referral programme

Two independent rewards, and the distinction matters:

**a) Invitation reward** — a flat $5 credit for inviting a friend who verifies and joins.
`referral-invites` / `referral-join` / SQL `referral_create_invite`,
`referral_begin_verification`, `referral_check_verification`, `referral_finish_join`,
`referral_resend_invite`, `referral_email_identity`.
Anti-abuse (`_shared/referral-rules.ts`): a code is sent only to the friend's mailbox, **five
attempts, ten-minute validity, three-minute resend spacing**; verification codes are generated
by **rejection sampling** (`newVerificationCode`) to avoid modulo bias; and identity is
normalised for life — `normalizeReferralEmail()` strips Gmail dots and `+aliases` so aliases
share one identity. The documented stance is blunt: *a distinct mailbox cannot prove a distinct
human — retain limits and abuse monitoring.*

**b) Programme referral reward** — a percentage of a real paid enrollment.
Attribution is **UTM-only**: `utm_source` carries the referrer's account UUID,
`utm_medium=learner-portal`, `utm_campaign=lms-student-referal`; `buildProgramReferralUrl()`
refuses any non-HTTPS or credentialed URL. Tables: `referral_settings`, `referral_invites`,
`referral_verifications`, `referral_join_rewards`, `referral_program_integrations`,
`referral_program_links`, `referral_application_attribution`, `referral_first_payments`,
`referral_lookup_attempts`.

### The intake contract other platforms must implement (`docs/referral-application-contract.md`)

A model cross-repo contract, written as "pending until verified":

1. Capture `lms_ref` from the personal link; keep the most recent valid token before
   submission; never overwrite a submitted application.
2. Send the token in the **initial application INSERT** together with the real programme code
   and student email. *A student UUID is not a reward claim.*
3. A database trigger hashes the token and verifies source table, programme, expiry and the
   referrer's active role. Invalid or absent tokens leave attribution empty.
4. Existing applications keep their original attribution; later token/UTM updates cannot claim
   them.
5. UTMs are preserved for analytics only.
6. Tokens are never exposed in public reads or logs and grant no read/edit rights.

Payment side: only trusted payment handlers may set paid status, **cash actually collected after
discounts**, currency and date — *coupon/credit value is not cash collected*. The trigger
consumes the first positive verified payment across active tables; captured amount and currency
are immutable; later programmes and refunds never reopen eligibility; the reward uses fixed FX,
whole-dollar rounding **down**, a one-day verification delay and refund reversal.
`lms.referral_program_integrations` maps source tables to approved `program_ids` and is marked
verified only after end-to-end checks of: valid/tampered/expired tokens, no token, changed
programme, competing inviter vs application referrer, existing unpaid applications,
simultaneous/retried payments, later programmes, refunds, non-USD, amount corrections, and
public vs private access. **Do not activate if untrusted clients can mark payments paid.**

---

## 4. The rest of the platform (for context)

Sessions and attendance (`sessions`, `session_mentors`, `session_attendance`,
`session_submissions`, `session_feedback`, `session_join_clicks`,
`session_recording_watch`, `session_section_recordings`, `lms.record_session_join`,
`finalize_join_attendance` on a 5-minute cron, `check-recording-access`); cohort structure
(`cohort_pathways`, `cohort_groups`+members+settings, `cohort_sections`+members,
`session_group_links`, `session_section_links`, `cohort_completions`); notifications
(`notifications` + `notification_recipients` ~20k + `notification_log`, `fan-out-notification`);
community (`community_channels`, `community_messages`, `community_reads`,
`community_mentions`, `community_ai_reviews`, `community_moderation_log`,
`community-ai-review`, `can_view_channel`/`is_channel_member` RLS helpers);
support (`support_tickets`, `ticket_messages`, `help_messages`, `help-agent` grounded by
`_shared/help-knowledge.ts`); certificates (`certificate_templates`, `certificates`,
`issue_certificates`, `revoke_certificate`); WhatsApp (`whatsapp_reminder_dispatch`,
`whatsapp_blocklist_archive`, `claim_whatsapp_reminders`/`finish_whatsapp_reminders`,
`_shared/whatsapp*.ts`, `wati-bulk.ts`, `session-whatsapp.ts`, timezone-aware reminder
dispatch with per-student stops); grade promotion (`grade_yearly_runs`, `promote_grades`,
`grade_reminder`); masterclasses (`mc_reg_base` union + `mc_norm()` label matching across every
platform's own registration table).

Role helpers used in RLS throughout: `lms.has_role(uid, role)`, `is_cohort_student`,
`is_cohort_mentor`, `is_my_catalog_mentor`, `my_mentor_id`, `community_members`.

---

## 5. What to copy from this design

1. **One engine, many callers.** A single `applyEnrollmentEvent()` with injected I/O, mirrored
   by one plpgsql function that writes state and audit atomically.
2. **Pure rules in their own module** so the state machine can be unit-tested and dry-run for
   people who do not have an account yet.
3. **A named action for every outcome**, including the refusals (`skipped_pinned`,
   `stale_ignored`, `conflict`), so logs explain themselves.
4. **Guards as constants**, not conditionals buried in code: launch date, max unenroll share,
   max changes per run.
5. **A typed confirmation (`LIVE`) checked server-side** before automation can write.
6. **One slow sending line** with claim/defer/finish SQL and hard caps, instead of a loop that
   emails everyone.
7. **Immutable ledgers and audit tables**, with integrity reports that only alert on mismatch.
8. **Economy as settings rows**, with kill switches, so no rate is hardcoded.
9. **Cross-repo features ship behind a flag and a written contract** that lists the exact
   checks required before activation.
10. **Normalise identity for anti-abuse** (Gmail aliases), and never claim a mailbox proves a
    human.
