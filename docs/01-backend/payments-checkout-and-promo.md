# Checkout, payments, promo codes, payment links and installments

Ground truth: the **Application Portal** repo. Every other Group platform's `/pay` page is a copy of this design (with the variations listed in §11). Paths below are relative to `D:\the Group\Vs Code Studio\Application Portal\` unless another repo is named.

The one sentence version: **the price is always computed on the server from the catalogue, the discount is always validated centrally, the PaymentIntent always carries `metadata.program_code`, and a payment that cannot be attributed goes to a human.**

---

## 1. The data plane

`supabase/functions/_shared/db.ts` creates three clients and is the model every sibling copies:

```ts
export const PLATFORM_SCHEMA = "portal";
centralAdmin()            // shared estate, schema portal — service role, RLS bypassed
centralShared(schema="public")  // shared estate, schema public — catalogue + promo_codes
localAuthAdmin()          // this project's auth.users only (getUser, createUser)
localUserClient(authHdr)  // this project, bound to the caller's JWT, to identify them
```
Secret names: `EXT_SUPABASE_URL`, `EXT_SUPABASE_SERVICE_KEY` (shared estate), `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_ANON_KEY` (local project), `STRIPE_SECRET_KEY`, `LWL_STRIPE_WEBHOOK_SECRET` / `STRIPE_WEBHOOK_SECRET`.

Because the service role bypasses RLS, **every edge function enforces its own authorisation** — see §9.

A second, thinner accessor exists for pure REST reads against the shared backend: `_shared/centralDb.ts` exposes `central.select(table, query)` used by the promo engine.

## 2. Where the money numbers live

| Source | Table / field | Notes |
|---|---|---|
| Catalogue price | `public.all_programs.sticker_price`, `early_bird_price`, `price_currency` | whole units; price is per programme, no per-cohort override |
| Form price plan | `portal.form_templates.settings` | `{ currency, full_price:{label,amount,stripe_price_id?}, installments_enabled, installments:[{label,amount}] }` — `stripe_price_id` is legacy, read only as a fallback when a row has no local `amount` |
| Alt currency | `settings.alt_currency`, `all_programs.alt_currency`, `payment_links.alt_currency` | a fixed admin-typed secondary-currency price (e.g. an IDR option), never live FX. Honoured only when the payer explicitly asks for that exact currency |
| Custom offer price | `portal.payment_links.full_amount`, `currency`, installments | overrides the catalogue entirely |
| Negotiated camp fee | saved on the application (`_shared/applicantChanges.ts` → `savedCampPrice`) | one exact full payment, not a catalogue plan |
| School-collected | `_shared/schoolPayment.ts` | when the school pays, online checkout is refused with `school_collected: true` |

Money handling rules in `_shared/pricing.ts`:
- Minor units everywhere internally: `toMinor/toMajor` with a zero-decimal currency set (`bif, clp, djf, gnf, jpy, kmf, krw, mga, pyg, rwf, ugx, vnd, vuv, xaf, xof, xpf`).
- `applyRequestedCurrency(pricing, requested)` trusts only the *name* of the requested currency; the amount always comes from the configured `altCurrency`.
- **Early-bird resolution (the cross-platform reference rule):** `amount = early_bird_price` when populated and `> 0`, else `sticker_price`. There is **no date-based expiry anywhere**; an admin clears `early_bird_price` to end the offer. The Portal shows no sticker-vs-early-bird strikethrough at all — strike-through is reserved for promo/school discounts. Any "was $X now $Y" display on a sibling platform is that platform's own invention.

## 3. Checkout entry points (frontend)

| Route | File | Purpose |
|---|---|---|
| `/apply/:slug` | `src/pages/Apply.tsx`, `src/pages/apply/` | the form built in Form Builder for that `pay_slug`; writes `public.applications` |
| `/pay/:slug` | `src/pages/PayRoute.tsx` → `PayDirect.tsx` | pay-then-apply, or a custom `link_slug` offer |
| `/pay?token=` | `PayApplication.tsx` | apply-then-pay: the token is the application's `submission_token` |
| legacy | `LegacyCheckoutRedirect.tsx` | old checkout URLs → the canonical ones |
| `/payment-success` | `PaymentSuccess.tsx` | post-payment confirmation |
| custom forms | `CustomForm.tsx` | standalone forms outside the programme catalogue |
| masterclass | `MasterclassRegister.tsx` | registration into `public.masterclass_regs` |
| consent | `MediaConsentForm.tsx` | signature capture (`src/components/consent/SignaturePad.tsx`) |

Both checkout pages carry a sticky mobile CTA bar with safe-area insets, 16px input font (iOS zoom), and scroll-into-view on panel unlock.

## 4. `create-payment-intent` — the whole flow

`supabase/functions/_shared/createPaymentIntent.ts` (shared by `create-payment-intent` and `portal-create-payment-intent`). Input `{ token, promo_code?, plan_index?, installment_count?, pay_mode?, utm?, autopay_consent?, add_on_link_id?, add_on_link_slug?, pay_currency? }`.

1. **CORS preflight**, then two rate limits: per IP (`pi:ip:<ip>`, 30 / 5 min) and per token (`pi:token:<token>`, 15 / 5 min), each in-memory **and** durable (`rateLimit` + `rateLimitDurable` in `_shared/rateLimit.ts`, backed by `portal.rate_limit_hits`). 429 with `Retry-After`.
2. **Token check** — must be a UUID. No bearer auth: the caller is an anonymous applicant, so the unguessable `submission_token` *is* the credential.
3. **Load the application**: `applications` by `submission_token` **or** `id` (migrated platforms mailed links keyed on the row id; `id` is a globally unique PK so the fallback can never pick the wrong applicant).
4. **Refusals**: school collects payment → 409 `school_collected`; `registration_status === 'withdrawn'` → 400; missing/invalid billing email → 400.
5. **Add-on charge path** — if the application is already `paid` and an `add_on_link_id`/`add_on_link_slug` is given, resolve the link, require it to be the same programme (`form_template_id` or `program_code` match), reuse an existing intent carrying the same `payment_link_id`, and create an extra charge whose webhook *adds* to `amount` and appends to `payment_intent_id` (`_shared/addOnPayment.ts`, `_shared/priorPayments.ts`, `portal.app_settings` key `extra_charge_payment_links`).
6. **Cohort lock** — a payment link may pin a `cohort_code`; the application's `program_code` must match it.
7. **Capacity guard** — when the form declares overall and/or per-cohort seat limits, they are checked here.
8. **Resolve pricing**: `resolvePaymentTemplate` → `readPricing` → `applyCataloguePricing` / `applySchoolPricing` / `applySavedCampPricing` / `applyRequestedCurrency`, with `hydrateFromStripe` only as a legacy fallback.
9. **Installments**: `plan_index` defaults to the first pending installment; multiple installments can be paid in one intent (`installment_count`), and rows are upserted in `portal.application_installments` *before* the intent is created so the webhook can mark them paid.
10. **Promo**: `validateCentralPromo()` against the central table (§5); legacy `_shared/promo.ts` (Stripe-native coupons) only when the central path is unreachable or `isLocalPromoDisabled`.
11. **Final money guard**:
```ts
if (!Number.isInteger(amount) || amount <= 0 || amount > baseAmount) { /* 400, generic message */ }
```
    The charged amount is always derived server-side, can never exceed the list price and can never be discounted to zero or below.
12. **Stripe customer** reused from `applications.stripe_customer_id`, recreated if deleted/missing.
13. **Metadata contract** — every intent carries:
    `row_id`, `application_id`, `student_id`, `program_code`, `program_name`, optional `cohort_code` + `billing_code_mode` (finance rollup when consecutive camp batches bill as one, `_shared/financeBillingCode.ts`), `student_name`, `student_email`, `school_name`, `parent_email`, the three acquisition UTMs, the `pay_*` UTMs of the payment click itself, `catalog_source: "supabase_local_catalog"`, `list_amount_minor`, `list_currency`, optional `paid_in_alt_currency`, `pay_mode`, legacy `price_id`, and when a link was used `payment_link_id`/`payment_link_slug`/`payment_link_name`, plus installment fields (`installment_index`, `installment_count`, `installment_total`, `installment_row_id(s)`) and `promo_code`.
14. Returns `{ clientSecret, amount, currency, customerId, paymentIntentId, promo?, plan_index, plan_total, installment_label }`.

**Attribution separation** (`_shared/attributionSeparation.ts`) is a design rule, not a detail: acquisition UTMs are frozen on first save (`snapshotApplicationUtm`, optimistic-concurrency retry loop) and never overwritten; the payment click is a *separate touch* stored as `pay_*` keys; each intent is immutable evidence recorded under its own id in `form_data.payment_attributions`; a field-by-field blend of two campaigns is explicitly forbidden. Missing values are honest `"unknown"`, never blank.

## 5. Promo codes (central, `public.promo_codes`)

Schema (`docs/promo-codes.sql`, applied to the shared estate's `public`):

| Column | Rule |
|---|---|
| `code` | `UNIQUE`, `CHECK (code = upper(code) AND code ~ '^[A-Z0-9_-]{3,32}$')` |
| `program_id` | FK `public.all_programs(id)` — one code belongs to exactly ONE programme and is valid for every cohort of it |
| `discount_type` | `fixed` \| `percent`; `CHECK` percent ≤ 100; `fixed` requires `currency` |
| `max_redemptions`, `redemption_count` | counter kept live by trigger `promo_sync_redemption_count()` |
| `starts_at`, `expires_at`, `is_active` | validity window |
| `created_by_platform`, `created_by_email`, `note` | the only attribution that exists (no partner-owner column) |

`public.promo_redemptions` keys on `payment_intent_id UNIQUE` — the idempotency guarantee. Both tables: `GRANT ALL TO service_role`, RLS enabled with no policies; browsers never touch them.

Engine: `_shared/centralPromo.ts`.
- **Families.** Because `code` is globally unique, one code text offered on several programmes is stored as a numbered family `SAVE15`, `SAVE15-2`, `SAVE15-3`. The customer always types the base; `selectPromoFamily()` loads base + numeric siblings only (`SAVE15-EARLY` is never swept in), and validation picks the row whose `program_id` matches the programme being paid.
- **Programme resolution.** Checkout sends the dated cohort code; `resolveProgramId()` resolves `all_cohorts.program_code` → `program_id`, falling back to the code family (`CATEGORY-ORG-PROGRAM`) and then the programme name.
- **`applyDiscount()`** clamps: percent capped at 100, fixed converted by zero-decimal rule, discount clamped to `[0, amountMinor]`.
- **Cap checked twice**: at creation (`central-admin-v1` action `promo_upsert` rejects a fixed amount or computed percent above the programme's live price) and again at checkout (format, family match, active, window, max redemptions, currency), then the final money guard in §4.11.
- **Anti-enumeration**: every rejection collapses to one public message ("This promo code is not valid for this programme"); the real reason is logged server-side only.
- **Redemption** is written by the webhook *after* real payment success via `recordCentralRedemption()`, which first checks `payment_intent_id` (returns `already: true`), then counts usage on the family row for the programme actually paid. Reporting failures never block a paid checkout.
- **Never on a payment link.** Promo codes are disallowed on any `/pay/{link_slug}` checkout — no stacking.

## 6. Payment links (`portal.payment_links`)

Custom priced offers: B2B cohorts, school deals, add-on charges (accommodation, extras). Columns grown over time (`docs/payment-links-catalogue.sql`, `payment-links-b2b-cohort.sql`, `payment-links-usage-cap-fix.sql`): `slug`, `name`, `program_id` → `public.all_programs`, `partner_id` → `public.all_partners`, `apply_url`, optional `form_template_id` (now nullable — most partner programmes apply through their own external `apply_url`), `deal_type` (`b2c`|`b2b`, default `b2c`), optional `cohort_code` lock, `full_amount`, `currency`, `alt_currency`, installments, `max_uses`/`uses_count`, `is_active`, `expires_at`.

Guards and functions (all `security definer`, `search_path = portal, public`):
- `portal.payment_links_guard()` — BEFORE INSERT/UPDATE OF slug, program_id, partner_id: slug ≥ 3 chars, `^[a-z0-9][a-z0-9-]*[a-z0-9]$`, and rejects reserved slugs (`success, apply, admin, pay, checkout, login`).
- `portal.payment_links_cohort_guard()` + `cohort_code_start(_code)` — validates the cohort lock against the code's embedded start date.
- `portal.get_public_payment_link(_slug)` — the **only** way other platforms may read a link, so the price shown is always live.
- `portal.payment_link_base_url(_link_id)` — resolves the publishing domain from the catalogue partner (the old `public_base_url` override column was dropped: links always resolve to the owning partner's domain).
- `portal.claim_payment_link_use(_link_id)` — atomic usage-cap claim, called **once per application** from the webhook only on the first successful payment. It is atomic against concurrent callers but *not* idempotent, so the one-time guarantee must come from the caller; if the cap was already reached the payment still stands and the overflow is logged for manual review.

**Known gap:** there is no price-floor enforcement. `admin-api`'s `create_payment_link` / `update_payment_link` only check `amount > 0`. The Growth Partner spec's claim that links are "never below the floor we set" is an aspiration. There is also no partner-owner column on either promo codes or payment links; attribution for affiliate-created rows is `created_by_platform`/`created_by_email` plus a human-readable `note`, with `affiliate.promo_code_links` / `affiliate.payment_link_links` as the durable join.

## 7. The Stripe webhook

`supabase/functions/stripe-webhook/index.ts` (1,107 lines; `portal-stripe-webhook` is the prefixed twin).

- Signature verified with `stripe.webhooks.constructEventAsync(body, sig, secret)`; missing signature → 400.
- `HANDLED_EVENTS` allow-list: anything else is acknowledged immediately with `{received:true, ignored}` — no DB client, no Stripe calls, no log noise.
- `SEEN_EVENTS` in-isolate guard drops duplicate deliveries of an already-processed `event.id`.
- **Two flows**: (1) custom checkout — the intent carries `metadata.row_id`, so the application is marked paid directly; (2) `reconcile()` — no `row_id`: the flagship camp Payment Links are matched and recorded against `camp_leads`; everything else is an old Stripe buy link or direct charge and is **logged for audit only, with no matching, no auto-create and no email**. The old `untracked_payments` table and email/name auto-matching were deliberately retired (the Founders Platform misattribution incident).

On `payment_intent.succeeded`:
1. Mark the application paid (`payment_status`, `payment_mode`, amount, intent id), merging split payments (`_shared/splitPayments.ts`) and applying the scholarship patch (`_shared/scholarshipCode.ts`).
2. Installments: mark the paid rows (`application_installments`), create the **next** unpaid row first with its due date (`INSTALLMENT_GAP_DAYS = 20`, `_shared/installmentSchedule.ts`) so the confirmation email can state it, arm autopay when consent was given, and mark the application paid only when installment 1 lands. Each installment gets its own confirmation keyed to the immutable installment row, so retries cannot duplicate and later installments are not suppressed.
3. `claimPaymentLinkUse()` once, on first success only.
4. `recordCentralRedemption()` for the promo.
5. `persistPaymentAttribution()` — optimistic JSON merge that never erases concurrent answers.
6. `trackPaymentConversion()` (`_shared/trackConversion.ts`) reports the settled payment to the shared attribution platform — backend-only, best-effort, logs on failure, never affects the payment.
7. Fire confirmation email (`send-application-confirmation` / `send-payment-confirmation`), registration-pending email when the application form is still missing, or `send-installment-autopay-failed` on a failed automatic charge.
8. Refunds: `deliverRefundNotice` / `notifyReconciledRefund` (`_shared/refundDelivery.ts`, `refundNotice.ts`, `refundPolicyDates.ts`, `refundProcessingFee.ts`).

## 8. Scheduled money jobs (pg_cron → edge function with `x-cron-secret`)

| Function | Does |
|---|---|
| `cron-payment-reminders` / `send-payment-reminder` | weekly reminders until the cohort starts |
| `cron-installment-reminders` / `send-installment-reminder` | the 20-day ladder (`_shared/nonCampReminderLadder.ts`, `whatsappPaymentReminder.ts`) |
| `cron-installment-autopay` / `charge-installment-autopay` | charges the next installment on its due date for consenting payers |
| `cron-internal-installment-digest` / `send-internal-installment-digest` | staff digest |
| `cron-application-confirmations` | catch-up confirmations |
| `cron-pending-registrations` / `send-registration-pending` | paid-but-no-form chasers |
| `cron-letter-of-acceptance` / `send-letter-of-acceptance` | LoA issuance |
| `cron-prune-expired-cohorts` | removes links whose cohort start date has passed and auto-drafts forms left with no upcoming intake |
| `cron-lead-welcome`, `cron-whatsapp-drip`, `camp-drip` | lead nurture (WATI WhatsApp, `_shared/wati.ts`, `watiBlocklist*.ts`) |

Auth for these: `_shared/cronAuth.ts`.

## 9. Edge-function auth models used here

| Model | Functions |
|---|---|
| Bearer + role lookup on the shared backend | `admin-api`, `portal-admin-api`, `central-admin-v1`, `portal-central-admin-v1`, `data-api`, `portal-data-api`, `secret-transfer`, `storage-transfer` — read `Authorization: Bearer`, identify via the local project's `auth.getUser()`, then look up the caller's role in `portal.user_profiles` with a service-role client and check an allow-list, else 403 |
| Anonymous + unguessable token + rate limit | `create-payment-intent`, `get-application-by-token`, `complete-application-details`, `custom-form-public`, `check-existing-application`, `capture-lead`, `capture-one-tap-lead`, `masterclass-regs-api` |
| Webhook signature | `stripe-webhook` |
| Cron secret | every `cron-*` function |

The prefixed duplicates are the consolidation of function names onto one prefix; both names exist during the transition.

## 10. The applications table itself

`docs/applications-public-schema.sql` (2026-09-04): the table was moved `portal.applications → public.applications`, and `portal.applications` is now an auto-updatable **compatibility view** (`security_invoker = true`) so every edge function, RPC and PostgREST call bound to `portal` keeps working. Grants moved with it: `SELECT/INSERT/UPDATE/DELETE` to `authenticated` and `service_role`, **`INSERT` only to `anon`** (public form submission). Triggers, RLS policies and the `application_installments` FK moved too. Finish with `NOTIFY pgrst, 'reload schema'`.

This pattern — move the table, leave a `security_invoker` view behind, re-grant, notify PostgREST — is the safe way to relocate a table that dozens of deployed functions address by schema.

## 11. How the sibling platforms differ

| Platform | Checkout shape | Notes |
|---|---|---|
| **Fellowship Platform** | `supabase/functions/_shared/cataloguePricing.ts` | the best sibling implementation: a dedicated, unit-tested pricing module, proper null-when-no-discount sticker field, correct dual-branch strike logic. 7/7 tests pass. Copy this one. |
| **Financial Literacy Platform** | edge functions + Flywire | had the severe bug of never fetching `early_bird_price` (fixed 2026-11-05); also has a Flywire webhook path (`finedu.flywire_webhook_events`), voucher redemptions and `finedu.unmatched_payments` |
| **Pre-Law Platform** | **TanStack Start** `src/lib/*.server.ts` | no edge functions for checkout — any fix must follow that shape, never copy edge-function code; `prelaw.promo_slot_claims` adds slot-limited promos |
| **Pre-Med Platform** | edge functions | early-bird was gated on a hardcoded date constant (`COMPETITION_EARLY_BIRD_ENDS`), removed 2026-11-05 to be purely DB-driven — meaning someone must now clear `early_bird_price` manually; also has a price-override block for a stale catalogue row, and `premed.stripe_charge_cache` / `stripe_reconcile_state` |
| **Research Platform** | edge functions | backend charged early-bird correctly but had no `early > sticker` guard, a decoy always-truthy `sticker_major`, and no frontend strike; all fixed |
| **Challenge Platform, Builders Lab, Sales Command** | none | hand off to the Application Portal with UTMs |
| **LMS** | `lms.partner_checkout_config` + vault vouchers | redirect config for pending-payment enrollments; the Vault supplies credits at checkout (`docs/vault-checkout-contract.md` in that repo) |
| **Growth Partner Portal** | creates rows, never charges | writes real rows into central `public.promo_codes` and `portal.payment_links` so the Portal's checkout can actually see them |

Razorpay (`public.razorpay_payments`, 1,836 rows) is the India-facing provider; its webhook verifies an HMAC signature and the data project holds `migration.razorpay_programme_rules` mapping a payment's `notes.app_name` + date to a programme, plus `migration.razorpay_true_local` correcting dates derived from the payment id (an earlier correction table was 5h30 off and is retained only for history).

## 12. Non-negotiables (quote these in any review)

1. `metadata.program_code` on every PaymentIntent. A payment without it is unattributable.
2. Route webhooks on `program_code` first. Keep tolerant matching only as a fallback on a shared Stripe account, and never remove it until every platform on that account has migrated. Where loose matching already caused misattribution, do not fall back — send to manual review.
3. Never auto-match on amount or timing.
4. All money math server-side. Client-sent amounts are hints; the final guard rejects anything above the server-computed base.
5. Promo codes validated centrally; the client never calculates a discount; one neutral rejection message.
6. Redemption recorded only after real payment success, idempotent on `payment_intent_id`.
7. Payment-link price fully overrides the catalogue, and promos never stack on it.
8. Usage caps claimed atomically, once, on first success.
9. Installment cadence lives in exactly one module (`installmentSchedule.ts`); pages and reminders read it.
10. Platforms with a second payment product (Purpose Platform's subscriptions) must pass non-Academy events through untouched.
