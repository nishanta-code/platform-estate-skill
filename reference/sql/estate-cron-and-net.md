# Shared estate: pg_cron jobs, pg_net, extensions, storage buckets

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only. Literal bearer tokens / API keys / cron secrets in job commands are replaced with `<redacted>`; the staff e-mail in one job body is replaced with `<staff-email>`.

## cron.job (29 jobs; all run as `postgres` on `localhost`)

Three authentication patterns are in use:

- **premed-*** jobs: `net.http_post` to an edge function with `Authorization: Bearer ` || `(select decrypted_secret from vault.decrypted_secrets where name = 'DOCTORS_SERVICE_ROLE_KEY')` - secret pulled from Vault at run time (good).
- **lms-*** jobs: `net.http_post` with a **literal** `apikey` and `Authorization: Bearer <JWT>` (the project anon/service JWT embedded in the job text - redacted) plus `x-service-key` pulled from Vault secret `ext_service_key`.
- **group-portal-*** jobs: `net.http_post` with a **literal** `x-cron-secret: <48-hex>` header embedded in the job command (redacted) and no timeout override.

Anyone who can `SELECT` from `cron.job` (postgres / dashboard) can read the literal values in the lms-* and group-portal-* jobs; consider moving them to Vault like the premed-* jobs.

| jobid | jobname | schedule | active | command (summarised) |
|---|---|---|---|---|
| 3 | premed-intake-worker-every-minute | `* * * * *` | yes | POST `/functions/v1/intake-worker`, Bearer from Vault `DOCTORS_SERVICE_ROLE_KEY`, body `{scheduled_at: now()}`, timeout 120000 |
| 4 | premed-check-pending-payments-hourly | `0 * * * *` | **no** | POST `/functions/v1/check-pending-payments`, Vault key, body `{source:'cron'}`, timeout 300000 |
| 6 | premed-nurture-captured-leads-daily | `0 9 * * *` | yes | POST `/functions/v1/nurture-captured-leads`, Vault key, timeout 300000 |
| 7 | premed-mmi-daily-reminder | `0 8 * * *` | yes | POST `/functions/v1/mmi-daily-reminder`, Vault key, timeout 300000 |
| 8 | premed-reconcile-stripe-payments-6h | `7 */6 * * *` | **no** | POST `/functions/v1/reconcile-stripe-payments`, Vault key, timeout 300000 |
| 9 | premed-concierge-daily-digest | `30 4 * * *` | yes | POST `/functions/v1/concierge-daily-digest`, Vault key, body `{source:'cron', to:'<staff-email>'}`, timeout 300000 |
| 14 | lms-finalize-join-attendance | `*/5 * * * *` | yes | `select lms.finalize_join_attendance()` (pure SQL, no HTTP) |
| 15 | lms-community-ai-review-daily | `30 0 * * *` | yes | POST `/functions/v1/lms_community-ai-review`, headers apikey `<redacted>`, Bearer `<redacted>`, x-service-key from Vault `ext_service_key`, body `{}`, timeout 150000 |
| 16 | lms-send-feedback-reminders-daily | `30 4 * * *` | yes | POST `/functions/v1/lms_send-feedback-reminders` (same header pattern as 15) |
| 17 | lms-send-feedback-emails-hourly | `15 * * * *` | yes | POST `/functions/v1/lms_send-feedback-emails` |
| 18 | lms-send-session-reminders-5min | `*/5 * * * *` | yes | POST `/functions/v1/lms_send-session-reminders` |
| 19 | lms-reconcile-enrollments-15min | `*/15 * * * *` | yes | POST `/functions/v1/lms_reconcile-enrollments` |
| 20 | lms-login-rollout-3min | `*/3 * * * *` | yes | POST `/functions/v1/lms_login-rollout`, body `{"action":"tick"}` |
| 21 | lms-grade-reminder-yearly | `5 0 1 1 *` | yes | POST `/functions/v1/lms_grade-yearly`, body `{"action":"reminder"}` |
| 22 | lms-grade-promotion-yearly | `5 0 1 4 *` | yes | POST `/functions/v1/lms_grade-yearly`, body `{"action":"promotion"}` |
| 23 | lms-vault-daily | `30 0 * * *` | yes | POST `/functions/v1/lms_vault-daily` |
| 24 | lms-birthday-run | `1,16,31,46 * * * *` | yes | POST `/functions/v1/lms_birthday-run` |
| 25 | lms-referral-followups-hourly | `20 * * * *` | yes | POST `/functions/v1/lms_referral-followups` |
| 26 | group-portal-cron-payment-reminders | `*/15 * * * *` | yes | POST `/functions/v1/group-cron-payment-reminders`, headers `Content-Type`, `x-cron-secret: <redacted>`, body `{}` |
| 27 | group-portal-cron-letter-of-acceptance | `*/15 * * * *` | yes | POST `/functions/v1/group-cron-letter-of-acceptance` (x-cron-secret) |
| 28 | group-portal-cron-installment-reminders-hourly | `0 * * * *` | yes | POST `/functions/v1/group-cron-installment-reminders` (x-cron-secret) |
| 29 | group-portal-prune-expired-cohorts-daily | `15 3 * * *` | yes | POST `/functions/v1/group-cron-prune-expired-cohorts` (x-cron-secret) |
| 30 | group-portal-cron-whatsapp-drip-hourly | `7 * * * *` | yes | POST `/functions/v1/group-cron-whatsapp-drip` (x-cron-secret) |
| 31 | group-portal-cron-pending-registrations-daily | `30 4 * * *` | yes | POST `/functions/v1/group-cron-pending-registrations` (x-cron-secret) |
| 32 | group-portal-cron-camp-drip-15min | `*/15 * * * *` | yes | POST `/functions/v1/group-camp-drip`, body `{"action":"run"}` (x-cron-secret) |
| 33 | group-portal-cron-lead-welcome-hourly | `5 * * * *` | yes | POST `/functions/v1/group-cron-lead-welcome`, body `{"action":"run"}` (x-cron-secret) |
| 34 | group-portal-cron-application-confirmations | `*/5 * * * *` | yes | POST `/functions/v1/group-cron-application-confirmations` (x-cron-secret) |
| 35 | group-portal-cron-installment-autopay-daily | `20 6 * * *` | yes | POST `/functions/v1/group-cron-installment-autopay` (x-cron-secret) |
| 36 | group-portal-cron-internal-installment-digest-daily | `30 4 * * *` | yes | POST `/functions/v1/group-cron-internal-installment-digest` (x-cron-secret) |

All HTTP targets are `https://<ESTATE_REF>.supabase.co/functions/v1/<name>`. Job ids 1, 2, 5, 10-13 are gone (deleted jobs; their history still sits in `cron.job_run_details` with `jobname = NULL`, see below). Schedules are UTC. Edge functions used by cron: 6 premed, 11 lms (`lms_*`), 11 group-portal (`group-*`).

## cron.job_run_details summary (185,603 rows, 186 MB, oldest 2026-08-18, never pruned)

| jobname | status | runs | first | last |
|---|---|---|---|---|
| premed-check-pending-payments-hourly | succeeded | 1971 | 2026-08-29 | 2026-11-19 |
| premed-concierge-daily-digest | succeeded | 63 | 2026-09-23 | 2026-11-24 |
| premed-intake-worker-every-minute | failed | 2 | 2026-11-07 | 2026-11-07 |
| premed-intake-worker-every-minute | succeeded | 125467 | 2026-08-29 | 2026-11-24 |
| premed-mmi-daily-reminder | succeeded | 87 | 2026-08-30 | 2026-11-24 |
| premed-nurture-captured-leads-daily | succeeded | 88 | 2026-08-29 | 2026-11-24 |
| premed-reconcile-stripe-payments-6h | succeeded | 337 | 2026-08-31 | 2026-11-23 |
| lms-birthday-run | succeeded | 1723 | 2026-11-06 | 2026-11-24 |
| lms-community-ai-review-daily | succeeded | 18 | 2026-11-07 | 2026-11-24 |
| lms-finalize-join-attendance | failed | 1 | 2026-11-07 | 2026-11-07 |
| lms-finalize-join-attendance | succeeded | 7457 | 2026-10-29 | 2026-11-24 |
| lms-login-rollout-3min | succeeded | 9178 | 2026-11-06 | 2026-11-24 |
| lms-reconcile-enrollments-15min | succeeded | 1723 | 2026-11-06 | 2026-11-24 |
| lms-referral-followups-hourly | succeeded | 431 | 2026-11-06 | 2026-11-24 |
| lms-send-feedback-emails-hourly | succeeded | 431 | 2026-11-06 | 2026-11-24 |
| lms-send-feedback-reminders-daily | succeeded | 18 | 2026-11-07 | 2026-11-24 |
| lms-send-session-reminders-5min | failed | 1 | 2026-11-07 | 2026-11-07 |
| lms-send-session-reminders-5min | succeeded | 5168 | 2026-11-06 | 2026-11-24 |
| lms-vault-daily | succeeded | 18 | 2026-11-07 | 2026-11-24 |
| group-portal-cron-application-confirmations | failed | 1 | 2026-11-07 | 2026-11-07 |
| group-portal-cron-application-confirmations | succeeded | 5152 | 2026-11-06 | 2026-11-24 |
| group-portal-cron-camp-drip-15min | succeeded | 1718 | 2026-11-06 | 2026-11-24 |
| group-portal-cron-installment-autopay-daily | succeeded | 18 | 2026-11-07 | 2026-11-24 |
| group-portal-cron-installment-reminders-hourly | succeeded | 429 | 2026-11-06 | 2026-11-24 |
| group-portal-cron-internal-installment-digest-daily | succeeded | 18 | 2026-11-07 | 2026-11-24 |
| group-portal-cron-lead-welcome-hourly | succeeded | 429 | 2026-11-06 | 2026-11-24 |
| group-portal-cron-letter-of-acceptance | succeeded | 1718 | 2026-11-06 | 2026-11-24 |
| group-portal-cron-payment-reminders | succeeded | 1718 | 2026-11-06 | 2026-11-24 |
| group-portal-cron-pending-registrations-daily | succeeded | 18 | 2026-11-07 | 2026-11-24 |
| group-portal-cron-whatsapp-drip-hourly | succeeded | 430 | 2026-11-06 | 2026-11-24 |
| group-portal-prune-expired-cohorts-daily | succeeded | 18 | 2026-11-07 | 2026-11-24 |
| (deleted jobs, jobname NULL) | failed | 367 | 2026-08-18 | 2026-11-10 |
| (deleted jobs, jobname NULL) | succeeded | 19387 | 2026-08-18 | 2026-11-21 |

Reading: the lms-* and group-portal-* jobs were all (re)created on 2026-11-06 (the consolidation onto this shared project); the one-off failures on 2026-11-07 are a single incident. `premed-check-pending-payments-hourly` and `premed-reconcile-stripe-payments-6h` were deactivated on 2026-11-19 / 11-23. Note "succeeded" only means `net.http_post` was queued - HTTP failures show up in `net._http_response`, not here.

## pg_net

| metric | value |
|---|---|
| net.http_request_queue rows (pending) | 0 |
| net._http_response rows | 1031 (oldest 2026-11-24 05:34 UTC - pg_net's 6 h TTL purge is working) |
| net._http_response size | 51 MB (heap 49 MB for ~1k live rows = heavy bloat left by continuous delete/insert; `VACUUM FULL net._http_response` would reclaim it) |
| supabase_functions.hooks (webhook log) | 115,089 rows, 16 MB (never purged) |

## Extensions (pg_extension)

| extension | version | schema |
|---|---|---|
| pg_cron | 1.6.4 | pg_catalog |
| pg_net | 0.20.0 | extensions |
| pg_stat_statements | 1.11 | extensions |
| pg_trgm | 1.6 | extensions |
| pgcrypto | 1.3 | extensions |
| plpgsql | 1.0 | pg_catalog |
| supabase_vault | 0.3.1 | vault |
| unaccent | 1.1 | extensions |
| uuid-ossp | 1.1 | extensions |

Not installed: pg_graphql (schema `graphql_public` absent from the API), postgres_fdw/dblink, pgvector, pgmq, pg_jsonschema. `premed.match_knowledge` therefore uses tsvector full-text search, not embeddings.

## Storage buckets (storage.buckets)

| bucket | public | size limit | allowed mime types | created |
|---|---|---|---|---|
| announcement-media | yes | 8 MB | image/jpeg, image/png, image/webp, image/gif | 2026-08-21 |
| avatars | yes | - | - | 2026-08-18 |
| backups | no | - | - | 2026-08-17 |
| camp-letters | no | - | - | 2026-08-27 |
| campaign-public-assets | no | - | - | 2026-10-27 |
| certificates | no | - | - | 2026-08-18 |
| finedu-email-attachments | no | - | - | 2026-08-21 |
| ops-campaign-assets | no | - | - | 2026-11-10 |
| ops-public-assets | yes | - | - | 2026-11-10 |
| lms-submissions | no | 10 MB | - | 2026-09-19 |
| group-site-custom-form-uploads | no | - | - | 2026-11-06 |
| mentor-photos | yes | - | - | 2026-08-18 |
| program-attachments | no | - | - | 2026-08-18 |
| public-assets | yes | - | - | 2026-10-16 |
| public-site-assets | yes | - | - | 2026-09-02 |
| sl-submissions | no | 10 MB | png, jpeg, webp, heic, pdf, doc, docx | 2026-09-07 |
| ticket-attachments | **yes** | - | - | 2026-08-21 |
| videos | yes | 200 MB | video/mp4 | 2026-09-05 |

18 buckets; 9 public. `ticket-attachments` being public means support-ticket uploads are readable by URL without auth - worth confirming that is intended. Object-level RLS policies exist only for announcement-media, program-attachments, lms-submissions, mentor-photos and ticket-attachments (see `shared-estate-rls-policies.md`).
