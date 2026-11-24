# Brain project: pg_cron, pg_net, storage, vault, extensions

Live snapshot taken 2026-11-24 from <DATA_REF>. Metadata only. **Every secret-looking literal has been replaced with `<redacted>`**: Authorization Bearer values, JWTs, the static `internal_token` value that most group-* jobs pass in the request body (one shared literal reused across ~40 jobs), and the `x-autopilot-secret` header of job 69. Header NAMES and function URLs are kept.

Finding worth acting on: cron.job command text stores those tokens in plain text (readable by any session that can read cron.job, which the cron_job_policy restricts to `username = current_user`, i.e. postgres). Rotating the shared internal_token means editing ~40 jobs; a vault secret + `vault.decrypted_secrets` lookup inside the command would centralise it (there is already one vault secret, `spine_anon_key`).

## cron.job (107 jobs: 63 active, 44 inactive)

All jobs run as `postgres` in database `postgres`. "fn X" = `net.http_post` to `https://<DATA_REF>.supabase.co/functions/v1/X` with header `Content-Type: application/json` (+ `Authorization: Bearer <redacted>` where present) and a JSON body carrying `internal_token: <redacted>` plus the listed parameters. Timeouts are `timeout_milliseconds`.

Three generations of jobs coexist: `lm_*` (jobids 43-63, the original lead-management set, all INACTIVE, body token placeholder `__SERVICE_KEY__`), `spine_*` / `sync-*` / `sop_*` / `research_*` (jobids 66-113, SQL-function jobs for the public spine, migration and research pipelines) and `group-*` (jobids 115-175, the current group platform set).

| jobid | jobname | schedule | active | target |
|---|---|---|---|---|
| 43 | lm_calendly_reconcile_30min | */30 * * * * | no | fn calendly-day-reconcile {days:60, insert_missing:true, backfill_days:14} |
| 44 | lm_zoom_sync_30min | */30 * * * * | no | fn zoom-sync {} |
| 45 | lm_zoom_sync_worker_every_minute | * * * * * | no | fn zoom-sync-worker {tick:now()} |
| 46 | lm_lwl_thread_sync | */10 * * * * | no | fn gmail-thread-sync |
| 47 | lm_meetings_finalize_sweeper_5min | */5 * * * * | no | fn meetings-finalize-sweeper |
| 48 | lm_lwl_finalize_past_due_15min | */15 * * * * | no | fn master-agent-daily {mode:finalize_past_due} |
| 49 | lm_dispatch_high_score_webhook_every_2min | */2 * * * * | no | fn dispatch-high-score-webhook {trigger:cron} |
| 50 | lm_lwl_core_sync_daily | 0 2 * * * | no | fn group-core-sync {mode:daily} timeout 300000 |
| 51 | lm_agent_runner_every_minute | * * * * * | no | fn agent-runner |
| 52 | lm_agent_sweeper_every_5min | */5 * * * * | no | fn agent-sweeper |
| 53 | lm_send_scheduled_emails_every_15min | */15 * * * * | no | fn send-scheduled-emails {source:cron, ts:now()} |
| 54 | lm_recheck_no_shows_hourly | 15 * * * * | no | fn master-agent-daily {mode:recheck_no_shows} |
| 55 | lm_lwl_daily_sweep | 0 7 * * * | no | fn master-agent-daily {} |
| 56 | lm_lwl_daily_post_conversion | 15 7 * * * | no | fn master-agent-daily {mode:post_conversion} |
| 57 | lm_lwl_daily_dead_detection | 30 7 * * * | no | fn master-agent-daily {mode:dead_detection} |
| 58 | lm_lwl_daily_reactivation | 0 8 * * 1 | no | fn master-agent-daily {mode:reactivation} |
| 59 | lm_school_reconciler_2h | 15 */2 * * * | no | fn school-reconciler |
| 60 | lm_sync_applications_morning | 30 0 * * * | no | fn sync-applications |
| 61 | lm_sync_applications_evening | 30 12 * * * | no | fn sync-applications |
| 62 | lm_sync_applications_every_2h | 7 */2 * * * | no | fn sync-applications |
| 63 | lm_weekly_review_sunday | 0 18 * * 0 | no | fn weekly-review {scheduled_at:now()} |
| 66 | spine_org_classifier | 30 0,8,16 * * * | no | SELECT public.org_classify_body('agent:org-classifier') |
| 67 | spine_link_by_domain | 40 */6 * * * | YES | SELECT public.research_link_by_domain(50000) |
| 69 | group-autopilot | */30 * * * * | no | POST https://project--b3fa42a7-0487-4c13-9bf5-1de7cd7703e3-dev.lovable.app/api/public/autopilot-tick, headers x-autopilot-secret: <redacted>, x-autopilot-source: pg_cron, timeout 55000 (external Lovable dev URL) |
| 71 | spine_org_promote | 50 */6 * * * | YES | SELECT public.org_promote_candidates() |
| 73 | spine_group_seed | 55 */6 * * * | YES | SELECT public.org_groups_seed() |
| 74 | spine_validator_provider | 30 21 * * * | no | SELECT public.provider_tick() |
| 75 | spine_validator_dns | */15 * * * * | YES | SELECT public.validator_tick('dns', 500) |
| 76 | spine_geo_propose | 15 9 * * * | no | SELECT public.geo_propose_body('agent:geo-hygiene') |
| 77 | spine_geo_apply | 10 * * * * | YES | SELECT public.geo_apply(500) |
| 78 | sync-applications-hourly | 5 * * * * | YES | select migration.sync_applications() |
| 79 | sync-payments-stripe-hourly | 20 * * * * | YES | select migration.sync_stripe_payments() |
| 80 | sync-payments-razorpay-hourly | 35 * * * * | YES | select migration.sync_razorpay_payments() |
| 82 | sop_partnerships_sweep_daily | 30 1 * * * | YES | select leads.sop_sweep() |
| 83 | sync-programme-codes-hourly | 25 * * * * | YES | select migration.sync_programme_codes() |
| 84 | sync-payments-dbs-hourly | 27 * * * * | YES | select migration.sync_dbs_payments() |
| 85 | sync-bank-refunds-hourly | 29 * * * * | YES | select migration.sync_bank_refunds() |
| 86 | group-identity-hygiene | 30 2 * * * | YES | select public.research_identity_hygiene_cron('identity-hygiene', 1000) |
| 87 | group-trust-totals | 25 2,10,18 * * * | no | select public.research_refresh_trust_totals() |
| 88 | group-domain-index | 35 6 * * * | no | select public.research_refresh_domain_index() |
| 91 | group-duplicate-merge | */30 * * * * | YES | select public.research_auto_merge_duplicates(3000, 'Auto-merge') |
| 92 | group-review-autoclean | 10 * * * * | YES | select public.research_review_autoclean() |
| 110 | group-email-check-stats | 17 4,12,20 * * * | no | select public.research_refresh_email_check_stats() |
| 111 | group-library-tick | */2 * * * * | YES | select public.research_library_tick(4) |
| 112 | group-contributions | */5 18-23 * * * | YES | select public.research_contrib_tick(3) |
| 113 | research_retry_tick_30m | 7,37 * * * * | YES | select public.research_retry_tick(5,30,360) |
| 115 | group-agent-runner-every-minute | * * * * * | no | fn agent-runner |
| 116 | group-agent-sweeper-every-5min | */5 * * * * | no | fn agent-sweeper |
| 117 | group-ai-backfill-every-5min | */5 * * * * | YES | fn ai-backfill {limit:10} |
| 118 | group-calendly-day-reconcile | */30 * * * * | YES | fn calendly-day-reconcile {days:60, insert_missing:true} timeout 60000 |
| 119 | group-calendly-field-backfill-daily | 20 1 * * * | YES | fn calendly-backfill {start:now-7d, end:now+30d, max:200} timeout 180000 |
| 120 | group-calendly-schema-watch-daily | 0 2 * * * | YES | fn calendly-schema-watch {days:120, limit:600} timeout 240000 (body token is a JWT, redacted) |
| 121 | group-extract-commitments-10min | */10 * * * * | YES | fn extract-commitments {limit:20} timeout 180000 |
| 122 | group-group-core-sync-daily | 0 2 * * * | YES | fn group-core-sync timeout 180000 |
| 123 | group-group-daily-dead-detection | 30 7 * * * | no | fn master-agent-daily {mode:dead_detection} |
| 124 | group-group-daily-post-conversion | 15 7 * * * | no | fn master-agent-daily {mode:post_conversion} |
| 125 | group-group-daily-reactivation | 0 8 * * 1 | no | fn master-agent-daily {mode:reactivation} |
| 126 | group-group-daily-sweep | 0 7 * * * | no | fn master-agent-daily (body literal has a trailing comma: `{"internal_token":"...",}` which is invalid JSON; job is inactive) |
| 127 | group-group-finalize-past-due-15min | */15 * * * * | YES | fn master-agent-daily {mode:finalize_past_due} timeout 120000 |
| 128 | group-group-followup-sync | */5 * * * * | YES | fn gmail-followup-sync timeout 150000 (no Authorization header; token in body only) |
| 129 | group-group-thread-sync | */10 * * * * | YES | fn gmail-thread-sync timeout 120000 |
| 130 | group-meeting-report-monthly | 0 4 1 * * | YES | fn send-meeting-report {kind:monthly, trigger:schedule} timeout 120000 |
| 131 | group-meeting-report-monthly-retry-1h | 0 5 1 * * | YES | fn send-meeting-report {kind:monthly} (retry) |
| 132 | group-meeting-report-monthly-retry-3h | 0 7 1 * * | YES | fn send-meeting-report {kind:monthly} (retry) |
| 133 | group-meeting-report-weekly | 30 4 * * 6 | YES | fn send-meeting-report {kind:weekly, trigger:schedule} |
| 134 | group-meeting-report-weekly-retry-1h | 30 5 * * 6 | YES | fn send-meeting-report {kind:weekly} (retry) |
| 135 | group-meeting-report-weekly-retry-3h | 30 7 * * 6 | YES | fn send-meeting-report {kind:weekly} (retry) |
| 136 | group-meetings-finalize-sweeper-5min | */5 * * * * | YES | fn meetings-finalize-sweeper timeout 60000 |
| 137 | group-notification-sweeper-10min | */10 * * * * | YES | fn notification-sweeper timeout 60000 |
| 138 | group-prep-upcoming-hourly | 25 * * * * | YES | fn prep-upcoming {limit:25, horizon_days:7} |
| 139 | group-pull-bank-refunds-hourly | 46 * * * * | YES | select group.pull_bank_refunds_from_core() |
| 140 | group-pull-from-core-hourly | 45 * * * * | YES | select group.pull_from_core() |
| 141 | group-recheck-no-shows-hourly | 15 * * * * | no | fn master-agent-daily {mode:recheck_no_shows} (Bearer and internal_token are EMPTY strings; broken, inactive) |
| 142 | group-recovery-backlog-drip-6h | 15 */6 * * * | YES | select group.release_recovery_backlog(10) |
| 143 | group-refresh-meeting-outcomes | 50 * * * * | YES | refresh materialized view group.mv_meeting_outcomes_all |
| 144 | group-resolve-school-codes-hourly | 48 * * * * | YES | select group.resolve_school_codes() |
| 145 | group-school-reconciler-2h | 15 */2 * * * | no | fn school-reconciler (body token is a JWT, redacted) |
| 146 | group-send-scheduled-emails-every-15min | */15 * * * * | YES | fn send-scheduled-emails {source:cron, ts:now()} timeout 120000 |
| 147 | group-sop_partnerships_sweep_daily | 30 1 * * * | YES | select group.sop_sweep_all() |
| 148 | group-summarize-meetings-15min | 4-59/15 * * * * | YES | fn summarize-meetings {limit:25} timeout 120000 |
| 149 | group-sync-applications-evening | 30 12 * * * | no | fn sync-applications (JWT body token, redacted) |
| 150 | group-sync-applications-every-2h | 7 */2 * * * | no | fn sync-applications (JWT body token, redacted) |
| 151 | group-sync-applications-morning | 30 0 * * * | no | fn sync-applications (JWT body token, redacted) |
| 152 | group-sync-meeting-attribution | 52 * * * * | YES | select group.sync_meeting_attribution() |
| 153 | group-system-health-15min | */15 * * * * | YES | select group.run_health_checks() |
| 154 | group-wa-day-start-15min | */15 * * * * | YES | fn wa-day-start timeout 60000 |
| 155 | group-wa-founder-brief-15min | 7-59/15 * * * * | YES | fn wa-founder-brief timeout 60000 |
| 156 | group-wa-inbound-poll-hourly | 0 0-18 * * * | YES | fn wa-inbound-poll timeout 60000, guarded by `WHERE (now() at time zone 'Asia/Kolkata')::time BETWEEN '06:00' AND '23:59:59'` |
| 157 | group-wa-meeting-reminder-2min | */2 * * * * | YES | fn wa-meeting-reminder timeout 30000 |
| 158 | group-wa-new-meeting-claim-every-5-min | */5 * * * * | no | fn wa-new-meeting-claim |
| 159 | group-wa-unassigned-nudge-15min | */15 * * * * | YES | fn wa-unassigned-nudge timeout 60000 |
| 160 | group-weekly-review-sunday | 0 18 * * 0 | no | fn weekly-review {scheduled_at:now()} (Authorization header value redacted) |
| 161 | group-welcome-bot-dispatch-minutely | * * * * * | YES | fn welcome-bot-dispatch {days:60, insert_missing:true} timeout 60000 |
| 162 | group-welcome-bot-expire | */10 * * * * | YES | select group.welcome_bot_expire_stale() |
| 163 | group-workspace-mailboxes | */5 * * * * | YES | fn workspace-mailboxes timeout 150000 |
| 164 | group-zoom-sync | */5 * * * * | YES | fn zoom-sync {limit:20} timeout 300000 |
| 165 | group-zoom-sync-worker-every-minute | * * * * * | YES | fn zoom-sync-worker {tick:now()} timeout 30000 |
| 166 | group-zoom-taker-backfill-daily | `40 1 * * *]` | YES | fn zoom-taker-backfill {limit:100} timeout 180000 (JWT body token, redacted). NOTE the schedule string carries a stray `]`; pg_cron still ran it at 01:40 daily (7 successful runs in the last 7 days) |
| 167 | group-meetings-sheet-sync | */5 * * * * | YES | fn meetings-sheet-sync timeout 150000 |
| 168 | group-member-sheet-report | */3 * * * * | YES | fn member-sheet-report timeout 150000 |
| 169 | group-repair-email-bodies | 17 * * * * | YES | fn repair-email-bodies timeout 150000 |
| 170 | group-member-email-agent | 25 * * * * | no | select group.member_email_agent_dispatch_all('{}'::jsonb) |
| 171 | group-member-email-examples | 30 2 * * 1 | YES | select group.member_email_agent_dispatch('{"refresh_examples":true}'::jsonb) |
| 172 | group-application-school-matcher | 35 * * * * | no | select group.application_school_matcher_dispatch('{"limit":40}'::jsonb) |
| 173 | group-wa-claim-poll-every-5-min | */5 * * * * | no | fn wa-claim-poll |
| 174 | group-auto-link-application-schools | 17 * * * * | YES | select group.auto_link_application_schools(500) |
| 175 | group-member-daily-email | 0 3 * * * | YES | select group.member_daily_email_dispatch_all() |

## cron.job_run_details, last 7 days (2026-11-17 11:38 to 2026-11-24 11:37 UTC), by job and status

Table has 156,190 rows in total going back to 2026-08-17 (155 MB heap, never vacuumed/analysed; pg_cron does not prune it unless a cleanup job exists, and none does).

| jobname | status | runs | first | last | avg secs | sample failure |
|---|---|---|---|---|---|---|
| group-ai-backfill-every-5min | failed | 2 | 11-18 09:50 | 11-18 09:55 | 22.4 | job startup timeout |
| group-ai-backfill-every-5min | succeeded | 2014 | 11-17 11:40 | 11-24 11:35 | 0.03 | |
| group-application-school-matcher | succeeded | 19 | 11-20 12:35 | 11-21 06:35 | 0.03 | (job since deactivated) |
| group-auto-link-application-schools | succeeded | 71 | 11-21 13:17 | 11-24 11:17 | 0.40 | |
| group-calendly-day-reconcile | succeeded | 336 | 11-17 12:00 | 11-24 11:30 | 0.04 | |
| group-calendly-field-backfill-daily | succeeded | 16 | 11-18 01:20 | 11-24 01:20 | 0.02 | |
| group-calendly-schema-watch-daily | succeeded | 7 | 11-18 02:00 | 11-24 02:00 | 0.04 | |
| group-contributions | succeeded | 504 | 11-17 18:00 | 11-23 23:55 | 36.2 | (heaviest SQL job: research_contrib_tick) |
| group-duplicate-merge | succeeded | 336 | 11-17 12:00 | 11-24 11:30 | 0.11 | |
| group-extract-commitments-10min | failed | 1 | 11-18 09:50 | 11-18 09:50 | 11.0 | job startup timeout |
| group-extract-commitments-10min | succeeded | 1007 | 11-17 11:40 | 11-24 11:30 | 0.03 | |
| group-identity-hygiene | succeeded | 7 | 11-18 02:30 | 11-24 02:30 | 22.1 | |
| group-library-tick | failed | 4 | 11-18 09:12 | 11-23 09:34 | 17.6 | job startup timeout |
| group-library-tick | succeeded | 5034 | 11-17 11:38 | 11-24 11:36 | 8.57 | (every 2 min, ~8.6 s each: the busiest SQL job) |
| group-group-core-sync-daily | succeeded | 7 | 11-18 02:00 | 11-24 02:00 | 0.03 | |
| group-group-finalize-past-due-15min | succeeded | 672 | 11-17 11:45 | 11-24 11:30 | 0.03 | |
| group-group-followup-sync | failed | 2 | 11-18 09:50 | 11-18 09:55 | 22.4 | job startup timeout |
| group-group-followup-sync | succeeded | 2014 | 11-17 11:40 | 11-24 11:35 | 0.03 | |
| group-group-thread-sync | failed | 1 | 11-18 09:50 | 11-18 09:50 | 11.1 | job startup timeout |
| group-group-thread-sync | succeeded | 1007 | 11-17 11:40 | 11-24 11:30 | 0.03 | |
| group-meeting-report-weekly | succeeded | 1 | 11-21 04:30 | 11-21 04:30 | 0.05 | |
| group-meeting-report-weekly-retry-1h | succeeded | 1 | 11-21 05:30 | | 0.02 | |
| group-meeting-report-weekly-retry-3h | succeeded | 1 | 11-21 07:30 | | 0.09 | |
| group-meetings-finalize-sweeper-5min | failed | 2 | 11-18 09:50 | 11-18 09:55 | 22.4 | job startup timeout |
| group-meetings-finalize-sweeper-5min | succeeded | 2014 | 11-17 11:40 | 11-24 11:35 | 0.03 | |
| group-meetings-sheet-sync | succeeded | 1429 | 11-19 12:00 | 11-24 11:35 | 0.02 | |
| group-member-daily-email | succeeded | 1 | 11-24 03:00 | | 0.09 | (new job) |
| group-member-email-agent | succeeded | 20 | 11-20 11:25 | 11-21 06:25 | 0.05 | (since deactivated) |
| group-member-email-examples | succeeded | 1 | 11-23 02:30 | | 0.04 | |
| group-member-sheet-report | failed | 2 | 11-23 06:57 | 11-23 09:33 | 12.8 | job startup timeout |
| group-member-sheet-report | succeeded | 1979 | 11-20 07:00 | 11-24 11:36 | 0.02 | |
| group-notification-sweeper-10min | failed | 1 | 11-18 09:50 | | 11.0 | job startup timeout |
| group-notification-sweeper-10min | succeeded | 1007 | 11-17 11:40 | 11-24 11:30 | 0.03 | |
| group-prep-upcoming-hourly | succeeded | 168 | 11-17 12:25 | 11-24 11:25 | 0.02 | |
| group-pull-bank-refunds-hourly | succeeded | 168 | 11-17 11:46 | 11-24 10:46 | 0.11 | |
| group-pull-from-core-hourly | succeeded | 168 | 11-17 11:45 | 11-24 10:45 | 1.18 | |
| group-recovery-backlog-drip-6h | succeeded | 28 | 11-17 12:15 | 11-24 06:15 | 0.18 | |
| group-refresh-meeting-outcomes | failed | 1 | 11-18 09:50 | | 11.1 | job startup timeout |
| group-refresh-meeting-outcomes | succeeded | 167 | 11-17 11:50 | 11-24 10:50 | 0.34 | |
| group-repair-email-bodies | succeeded | 98 | 11-20 10:17 | 11-24 11:17 | 0.01 | |
| group-resolve-school-codes-hourly | succeeded | 168 | 11-17 11:48 | 11-24 10:48 | 0.51 | |
| group-review-autoclean | succeeded | 168 | 11-17 12:10 | 11-24 11:10 | 0.17 | |
| group-send-scheduled-emails-every-15min | succeeded | 672 | 11-17 11:45 | 11-24 11:30 | 0.03 | |
| group-sop_partnerships_sweep_daily | succeeded | 7 | 11-18 01:30 | 11-24 01:30 | 0.28 | |
| group-summarize-meetings-15min | failed | 1 | 11-23 09:34 | | 37.7 | job startup timeout |
| group-summarize-meetings-15min | succeeded | 671 | 11-17 11:49 | 11-24 11:34 | 0.01 | |
| group-sync-meeting-attribution | succeeded | 168 | 11-17 11:52 | 11-24 10:52 | 0.49 | |
| group-system-health-15min | succeeded | 672 | 11-17 11:45 | 11-24 11:30 | 2.39 | |
| group-wa-day-start-15min | succeeded | 672 | 11-17 11:45 | 11-24 11:30 | 0.03 | |
| group-wa-founder-brief-15min | succeeded | 672 | 11-17 11:52 | 11-24 11:37 | 0.01 | |
| group-wa-inbound-poll-hourly | succeeded | 133 | 11-17 12:00 | 11-24 11:00 | 0.04 | |
| group-wa-meeting-reminder-2min | failed | 4 | 11-18 09:12 | 11-23 09:34 | 17.3 | job startup timeout |
| group-wa-meeting-reminder-2min | succeeded | 5034 | 11-17 11:38 | 11-24 11:36 | 0.02 | |
| group-wa-unassigned-nudge-15min | succeeded | 672 | 11-17 11:45 | 11-24 11:30 | 0.03 | |
| group-welcome-bot-dispatch-minutely | failed | 11 | 11-18 09:12 | 11-23 11:32 | 17.1 | server restarted |
| group-welcome-bot-dispatch-minutely | succeeded | 10068 | 11-17 11:38 | 11-24 11:37 | 0.01 | |
| group-welcome-bot-expire | failed | 1 | 11-18 09:50 | | 11.1 | job startup timeout |
| group-welcome-bot-expire | succeeded | 1007 | 11-17 11:40 | 11-24 11:30 | 0.04 | |
| group-workspace-mailboxes | failed | 2 | 11-18 09:50 | 11-18 09:55 | 22.4 | job startup timeout |
| group-workspace-mailboxes | succeeded | 2014 | 11-17 11:40 | 11-24 11:35 | 0.02 | |
| group-zoom-sync | failed | 2 | 11-18 09:50 | 11-18 09:55 | 22.4 | job startup timeout |
| group-zoom-sync | succeeded | 1810 | 11-17 12:00 | 11-24 11:35 | 0.03 | |
| group-zoom-sync-worker-every-minute | failed | 9 | 11-18 09:27 | 11-23 09:34 | 18.1 | job startup timeout |
| group-zoom-sync-worker-every-minute | succeeded | 10069 | 11-17 11:38 | 11-24 11:37 | 0.01 | |
| group-zoom-taker-backfill-daily | succeeded | 7 | 11-18 01:40 | 11-24 01:40 | 0.02 | |
| research_retry_tick_30m | succeeded | 336 | 11-17 12:07 | 11-24 11:37 | 0.39 | |
| sop_partnerships_sweep_daily | succeeded | 7 | 11-18 01:30 | 11-24 01:30 | 0.26 | |
| spine_geo_apply | succeeded | 168 | 11-17 12:10 | 11-24 11:10 | 0.12 | |
| spine_group_seed | succeeded | 28 | 11-17 12:55 | 11-24 06:55 | 0.36 | |
| spine_link_by_domain | succeeded | 28 | 11-17 12:40 | 11-24 06:40 | 1.04 | |
| spine_org_promote | succeeded | 28 | 11-17 12:50 | 11-24 06:50 | 3.80 | |
| spine_validator_dns | succeeded | 672 | 11-17 11:45 | 11-24 11:30 | 0.05 | |
| **sync-applications-hourly** | **failed** | **93** | 11-17 12:05 | **11-24 11:05** | 1.13 | `ERROR: relation "public.edge_applications" does not exist` (remote SQL via postgres_fdw) |
| sync-applications-hourly | succeeded | 75 | 11-20 12:05 | 11-23 14:05 | 21.9 | |
| sync-bank-refunds-hourly | succeeded | 168 | 11-17 12:29 | 11-24 11:29 | 1.39 | |
| sync-payments-dbs-hourly | failed | 1 | 11-18 09:27 | | 10.0 | job startup timeout |
| sync-payments-dbs-hourly | succeeded | 167 | 11-17 12:27 | 11-24 11:27 | 1.54 | |
| sync-payments-razorpay-hourly | succeeded | 168 | 11-17 12:35 | 11-24 11:35 | 12.0 | |
| **sync-payments-stripe-hourly** | failed | 1 | 11-23 09:20 | | 120.1 | `canceling statement due to statement timeout` in sync_refunds_from_payments (statement_timeout is 120 s; the job averages 77.7 s) |
| sync-payments-stripe-hourly | succeeded | 167 | 11-17 12:20 | 11-24 11:20 | 77.7 | |
| sync-programme-codes-hourly | succeeded | 168 | 11-17 12:25 | 11-24 11:25 | 2.09 | |

Reading of the failures: (1) `sync-applications-hourly` is the only job failing right now and has failed on 93 of the last 168 hourly runs, including the latest (11:05 today) -- the legacy `public.edge_applications` table was dropped upstream on ~2026-11-16 (migration.v_app_edge was stubbed to `WHERE false`, but migration.sync_applications() still probes the remote table); (2) the "job startup timeout" bursts on 2026-11-18 09:12-09:55 and 2026-11-23 09:33-09:34 and the "server restarted" entries at 2026-11-23 11:32 line up with the postmaster start time (2026-11-23 11:32:52 UTC), i.e. a platform restart, not job bugs; (3) `sync-payments-stripe-hourly` runs close to the 120 s statement_timeout.

## pg_net

| metric | value |
|---|---|
| net.http_request_queue rows (pending) | 25 |
| net._http_response rows (retained window 2026-11-24 05:35 to 11:34 UTC, ~6 h) | 2787 |
| responses by status | 200: 2613, 504: 23, 404: 1, null (timed_out, error_msg set): 150 |
| timed_out | 150 (5.4 %) |

The 150 timeouts over 6 hours correspond to edge-function calls that exceed the per-job `timeout_milliseconds`; the 504s are gateway timeouts from the functions platform. RLS is off on net.* (normal).

## Storage

| bucket | public | file_size_limit | allowed_mime_types | created | objects |
|---|---|---|---|---|---|
| task-attachments | false | none | none | 2026-11-13 | 0 |
| email-attachments | false | none | none | 2026-11-13 | 0 |

12 policies on storage.objects (listed in brain-project-rls-policies.md): per bucket, admin ALL, member SELECT scoped through group.tasks / group.email_sends attachments, and staff insert/update/delete/select on own objects (owner = auth.uid() and group.is_active_staff). No objects stored yet.

## Vault

1 secret: `spine_anon_key` (created 2026-08-18). Value not read.

## Extensions installed (9)

| extension | version | schema | note |
|---|---|---|---|
| plpgsql | 1.0 | pg_catalog | |
| pg_cron | 1.6.4 | pg_catalog | |
| pg_stat_statements | 1.11 | extensions | 583 tracked statements |
| uuid-ossp | 1.1 | extensions | |
| pgcrypto | 1.3 | extensions | |
| supabase_vault | 0.3.1 | vault | |
| pg_trgm | 1.6 | **public** | installed in public rather than extensions (Supabase advisor "extension in public") |
| postgres_fdw | 1.1 | **public** | used by legacy_ro / legacy_sat foreign schemas and migration.sync_* |
| pg_net | 0.20.4 | **public** | |

Notable NOT installed although available: pg_graphql (graphql_public schema exists but is empty), vector, pgmq, pg_jsonschema, hypopg, index_advisor, pgsodium, wrappers, pg_repack, pgaudit.

## supabase_migrations.schema_migrations

352 applied migrations; latest version 20261124110938 (applied today, 2026-11-24).
