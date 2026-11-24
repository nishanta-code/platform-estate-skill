# Shared estate: size and health

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6 on aarch64). Metadata only.

- Database size (`pg_database_size`): **582 MB**
- Business schemas (13 with tables): ~280 MB; system/log schemas `cron` + `net` + `supabase_functions` + `auth`: ~270 MB.

## Top 30 tables by total size (table + indexes + TOAST)

`n_live/n_dead` from `pg_stat_all_tables`; `seq_scan` / `idx_scan` are cumulative since stats reset. `last autovac/analyze` NULL = never (stats for those tables are unreliable; `n_live_tup = 0` on a 19 MB table means "never analysed", not empty).

| # | table | total | heap | indexes | toast | n_live | n_dead | seq_scan | seq_tup_read | idx_scan | last autovac | last autoanalyze |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | cron.job_run_details | 186 MB | 182 MB | 4.5 MB | 8 kB | (185,603 by count) | - | 12 | 1,209,581 | 0 | never | never |
| 2 | net._http_response | 51 MB | 49 MB | 704 kB | 1.5 MB | (1,031 by count) | - | 12 | 16,747 | 12 | never | never |
| 3 | research.analytics_events | 33 MB | 24 MB | 9.0 MB | 8 kB | 2,725 | 0 | 0 | 0 | 0 | never | never |
| 4 | lms.notification_log | 30 MB | 4.1 MB | 632 kB | **25 MB** | 11,784 | 78 | 342 | 3,551,708 | 6 | never | 2026-11-24 |
| 5 | public.master_data | 19 MB | 17 MB | 1.3 MB | 312 kB | 0 (stale) | 0 | 2 | 30,160 | 15 | never | never |
| 6 | supabase_functions.hooks | 16 MB | 9.7 MB | 6.4 MB | 8 kB | 4,106 (115,089 by count) | 0 | 0 | 0 | 1 | never | never |
| 7 | testprep.lead_events | 15 MB | 11 MB | 4.1 MB | 24 kB | 20 | 0 | 0 | 0 | 8 | never | never |
| 8 | portal.email_send_log | 13 MB | 7.4 MB | 6.0 MB | 8 kB | 28,809 | 609 | 2,488 | 68,152,623 | 376,249 | never | 2026-11-23 |
| 9 | public.applications | 11 MB | 7.1 MB | 2.3 MB | 1.5 MB | 5,110 | 996 | **24,022** | **88,962,988** | 144,422 | 2026-11-16 | 2026-11-22 |
| 10 | public.stripe_payments | 10 MB | 8.3 MB | 2.1 MB | 8 kB | 13,930 | 1,198 | 457 | 5,542,629 | 111,222 | 2026-11-23 | 2026-11-23 |
| 11 | finedu.page_visits | 9.8 MB | 6.4 MB | 3.3 MB | 8 kB | 3,168 | 0 | 0 | 0 | 0 | never | never |
| 12 | auth.refresh_tokens | 9.0 MB | 3.5 MB | 5.5 MB | 8 kB | 22,238 | 2,155 | 1 | 22,238 | 13,897 | 2026-11-20 | 2026-11-22 |
| 13 | lms.enrollment_events | 8.6 MB | 7.3 MB | 1.2 MB | 24 kB | 5,940 | 10 | 2 | 10,359 | 1,023 | never | 2026-11-20 |
| 14 | public.premed_masterclass_regs | 6.3 MB | 5.3 MB | 1.0 MB | 8 kB | 133 | 0 | 5 | 18,313 | 324 | never | never |
| 15 | lms.enrollment_sync_items | 6.3 MB | 4.9 MB | 1.4 MB | 8 kB | 1,602 | 0 | 0 | 0 | 0 | never | never |
| 16 | premed.intake_raw | 6.0 MB | 5.2 MB | 720 kB | 104 kB | 133 | 133 | 2 | 5,892 | 271 | never | never |
| 17 | public.form_responses | 6.0 MB | 2.1 MB | 360 kB | 3.6 MB | 1,767 | 242 | 3,086 | 5,265,590 | 12,088 | never | 2026-11-20 |
| 18 | portal.whatsapp_undeliverable_numbers | 5.5 MB | 4.0 MB | 1.4 MB | 8 kB | 269 | 1 | 13 | 65,583 | 39,608 | never | never |
| 19 | finance.wallex_transactions | 4.3 MB | 3.4 MB | 864 kB | 8 kB | 1,025 | 193 | 205 | 187,648 | 363,532 | never | 2026-11-21 |
| 20 | lms.notification_recipients | 3.9 MB | 1.7 MB | 2.1 MB | 0 | 20,542 | 604 | 0 | 0 | 48,865 | never | 2026-11-21 |
| 21 | prelaw.whatsapp_messages | 3.2 MB | 2.6 MB | 528 kB | 8 kB | 0 | 0 | 2 | 3,220 | 2 | never | never |
| 22 | testprep.session_analytics | 3.2 MB | 1.7 MB | 1.4 MB | 8 kB | 8 | 9 | 0 | 0 | 0 | never | never |
| 23 | lms.login_rollout_queue | 3.1 MB | 2.2 MB | 904 kB | 8 kB | 4,224 | 119 | 3,932 | 11,108,516 | 24,009 | 2026-11-21 | 2026-11-20 |
| 24 | auth.users | 3.1 MB | 2.0 MB | 1.1 MB | 8 kB | 4,228 | 683 | 511 | 247,139 | 47,006 | never | 2026-11-23 |
| 25 | lms.notifications | 2.8 MB | 1.7 MB | 1.0 MB | 8 kB | 781 | 45 | 4 | 35,361 | 68,474 | never | never |
| 26 | premed.sent_emails | 2.7 MB | 1.4 MB | 1.2 MB | 8 kB | 142 | 0 | 1 | 5,616 | 8 | never | never |
| 27 | lms.vault_lots | 2.4 MB | 1.0 MB | 1.3 MB | 8 kB | 5,826 | 423 | 377 | 2,117,636 | 4,587 | never | 2026-11-22 |
| 28 | public.research_masterclass_regs | 2.3 MB | 1.8 MB | 464 kB | 8 kB | 18 | 9 | 4 | 9,448 | 83 | never | never |
| 29 | premed.side_effects_queue | 2.3 MB | 2.0 MB | 328 kB | 8 kB | 3,306 | 410 | 5 | 16,307 | 23,832 | never | 2026-11-22 |
| 30 | lms.session_reminder_log | 2.3 MB | 1.1 MB | 1.1 MB | 8 kB | 9,013 | 1 | 0 | 0 | 2,138 | never | 2026-11-21 |

## Notable findings

### Log-table bloat (system schemas, ~250 MB of the 582 MB)

1. **`cron.job_run_details` - 186 MB, 185,603 rows, oldest 2026-08-18, never purged.** pg_cron does not clean this table on Supabase by default. The `premed-intake-worker-every-minute` job alone adds ~1,440 rows/day (125k so far). Standard fix is a daily cron job: `DELETE FROM cron.job_run_details WHERE end_time < now() - interval '7 days'` (Supabase docs recommend exactly this), then one `VACUUM FULL cron.job_run_details` (needs a short lock) to give the 180 MB back.
2. **`net._http_response` - 51 MB holding only 1,031 live rows.** pg_net's TTL purge is working (oldest row is from today), but the heap never shrinks after the constant delete/insert churn and autovacuum has never run on it. `VACUUM FULL net._http_response` (or `pg_repack`) would reclaim ~48 MB.
3. **`supabase_functions.hooks` - 115,089 rows, 16 MB, never purged.** This is the Database-Webhook request log from the 14 `script sync` triggers; it grows on every row change to `public.applications` etc. Safe to delete rows older than ~30 days.

### Business-table bloat / stale statistics

4. Tables whose on-disk size is far out of proportion to their live rows (heavy delete/update churn, never autovacuumed): `research.analytics_events` (24 MB heap for 2,725 rows), `testprep.lead_events` (11 MB for 20 rows), `finedu.page_visits` (6.4 MB for 3,168 rows), `public.premed_masterclass_regs` (5.3 MB for 133 rows), `premed.intake_raw` (5.2 MB, 133 live / 133 dead), `lms.enrollment_sync_items` (4.9 MB for 1,602 rows), `prelaw.whatsapp_messages` (2.6 MB, 0 live rows), `testprep.session_analytics`, `public.research_masterclass_regs`, `premed.sent_emails`. These look like tables that were bulk-loaded then truncated-by-DELETE during the migration into this shared project. A one-off `VACUUM (FULL, ANALYZE)` per table (each is small, lock is seconds) or `pg_repack` cleans them.
5. **Autovacuum/analyze has never run on most tables** (`last_autovacuum`/`last_autoanalyze` NULL on 22 of the top 30). The default autovacuum thresholds (50 rows + 20%) are rarely hit on small tables, so planner statistics are stale and `n_live_tup` is misleading (e.g. `public.master_data` shows 0 live rows at 19 MB, `public.all_mentors`/`all_partners` show 0). Running `ANALYZE` database-wide (cheap, no locks) is worthwhile after the November consolidation; consider `ALTER TABLE ... SET (autovacuum_vacuum_scale_factor = 0.05)` on the churny log tables.
6. `lms.notification_log` is 25 MB of TOAST for 11.8k rows - it stores full rendered e-mail/WhatsApp bodies per send. Fine today, but it is the fastest-growing business table (30 MB in ~3 weeks); consider trimming `body`/payload columns after 90 days.

### Query patterns worth a look (seq_scan vs idx_scan)

7. **`public.applications`: 24,022 sequential scans reading 89 M tuples** (vs 144k index scans). With 5,110 rows a seq scan is cheap individually, but ~17k tuples read per seq scan means some frequent query filters on a column without an index (candidates: `payment_date`, `registration_status` non-partial, `student_id` with mixed case, `parent_email`, `form_data` keys). Check `pg_stat_statements` for the top statements on this table.
8. `portal.email_send_log`: 2,488 seq scans / 68 M tuples read - likely `WHERE template_slug = ... AND sent_at > ...` or per-student lookups by `recipient_email` with `lower()`; existing indexes are on `(student_id, sent_at)`, `(recipient_email, template_slug)`, `(sent_at)`.
9. `lms.login_rollout_queue`: 3,932 seq scans / 11 M tuples - the 3-minute rollout tick appears to scan the whole queue; an index on `(status, next_attempt_at)` (or whatever `claim_login_rollout_item()` filters on) would help.
10. `lms.notification_log` (342 seq scans / 3.5 M), `public.form_responses` (3,086 / 5.3 M), `lms.vault_lots` (377 / 2.1 M), `public.stripe_payments` (457 / 5.5 M) - moderate; revisit once stats are fresh.

### Dead-tuple ratios to watch

`public.applications` 996 dead / 5,110 live (16%), `public.stripe_payments` 1,198 / 13,930, `auth.users` 683 / 4,228, `auth.refresh_tokens` 2,155 / 22,238, `finance.wallex_transactions` 193 / 1,025, `lms.vault_lots` 423 / 5,826 - all normal for actively-updated tables and will be handled once autovacuum triggers; no action needed beyond item 5.

### Other observations

- No materialized views exist anywhere, so there is nothing to refresh.
- `pg_stat_statements` 1.11 is installed - use it (via the Supabase Dashboard "Query Performance" page or `select * from extensions.pg_stat_statements order by total_exec_time desc limit 20`) to confirm items 7-10.
- Index count is high relative to data volume on several small tables (e.g. 13 indexes on `public.applications`, 9 on `research.school_leads`, 10 on `research.school_teacher_contacts`); harmless at this size but each adds write cost.
- Supabase advisors (`get_advisors`) were not run as part of this snapshot; the security advisor will flag the `public` tables with RLS and no policies, the SECURITY DEFINER views (`premed.programme_applications_read` etc.) and functions lacking a fixed `search_path` (only a handful here - most functions set one).
