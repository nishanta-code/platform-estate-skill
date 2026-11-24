# Brain project: schemas and tables

Live snapshot taken 2026-11-24 from <DATA_REF> (Supabase project "the Group", ap-northeast-2, PostgreSQL 17.6, database size 10 GB). Metadata only; no business rows were read.

## Schema size summary (sum of pg_total_relation_size over tables, views and matviews)

| schema | tables (incl. partitions) | views | matviews | size |
|---|---|---|---|---|
| public | 192 | 13 | 0 | 5659 MB |
| group | 101 | 32 | 1 | 3713 MB |
| quarantine | 14 | 0 | 0 | 820 MB |
| cron (system) | 2 | 0 | 0 | 155 MB |
| staging | 49 | 2 | 0 | 41 MB |
| revenue_brain | 51 | 4 | 0 | 34 MB |
| migration | 7 | 18 | 0 | 30 MB |
| leads | 39 | 1 | 0 | 26 MB |
| net (system) | 2 | 0 | 0 | 6296 kB |
| auth (system) | 27 | 0 | 0 | 2064 kB |
| supabase_migrations (system) | 1 | 0 | 0 | 904 kB |
| brain | 9 | 0 | 0 | 424 kB |
| storage (system) | 10 | 0 | 0 | 368 kB |
| attribution | 7 | 4 | 0 | 320 kB |
| realtime (system) | 10 | 0 | 0 | 264 kB |
| vault (system) | 1 | 1 | 0 | 48 kB |
| extensions (system) | 0 | 2 | 0 | - |
| core_ro | 0 | 3 | 0 | - (views over public.v_puller_*) |
| legacy_ro, legacy_sat | 0 | 0 | 0 | foreign-table schemas (postgres_fdw to the legacy Lovable projects); no local tables |
| graphql, graphql_public, pgbouncer | 0 | 0 | 0 | - |

User-schema table total: 469 (public 192 incl. 4 activities partitions + parent, group 101, staging 49, revenue_brain 51, leads 39, quarantine 14, brain 9, attribution 7, migration 7). Non-system total incl. cron/net/auth/storage/realtime/vault: 522.

## RLS disabled: 42 user-schema tables (all in staging.* and migration.*)

- **migration (5)**: baseline_20260902, payments_backup_20260902, payments_quarantine_20260902, razorpay_date_corrections, razorpay_programme_rules
- **staging (37)**: akbar_huso_rzp, akbar_offline_sheet, akbar_payments_snapshot, akbar_prog_code, akbar_prog_evidence, akbar_prog_map, akbar_sheet_emails, akbar_verification, akbar_verification_v1, applications_removed_20261003, code_lead, derived_codes, desc_code_evidence, desc_family, edge_claims, edge_household_of, edge_pairs, edge_people, edge_rows, edition_candidates, family_backtest, family_lead, huso_household_of, huso_norm, huso_pairs, huso_people, link_derived, payer_household_of, payer_pairs, payer_people, payments_raw, plink_code_windows, plink_codes, plink_evidence, prospects, prospects_raw, uncoded

Mitigation in place: neither staging nor migration is in the PostgREST exposed schema list and neither has USAGE granted to anon/authenticated (see brain-project-grants.md), so these tables are reachable only by postgres/service_role. Every other user-schema table has RLS enabled. Note that "RLS enabled" with **0 policies** (all of attribution.*, quarantine.*, revenue_brain.*, migration.id_map/run_log, and roughly 70 public.* tables) means deny-all for anon/authenticated and full access only for service_role/owner.

System schemas with RLS off (normal for Supabase): auth.custom_oauth_providers, mfa_recovery_code*, oauth_*, scim_*, webauthn_*; realtime.*, net.*, vault.secrets, supabase_migrations.schema_migrations.

## Row-count caveat

pg_stat_user_tables.n_live_tup reads 0 for most large tables: the postmaster restarted 2026-11-23 11:32 UTC (cron.job_run_details carries "server restarted" failures at that minute) and cumulative statistics did not survive it, so n_live_tup only counts rows touched since. The reltuples column (planner estimate from the last ANALYZE/VACUUM) is listed alongside and is the better approximation (e.g. public.research_contacts reltuples 599,126 vs n_live_tup 0; quarantine.research_contacts 647,650; public.research_raw_rows 764,682).

## Legend

table | n_live_tup (since restart) | reltuples (planner estimate) | total size incl. indexes+toast | RLS enabled | number of policies | table comment verbatim (these comments encode design decisions such as D-006, D-007, D-011, D-014, D-018, D-063, non-negotiables, and numbered notes like 025/028/030/036/038).

## Schema: attribution

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| campaigns | 0 | -1 | 48 kB | yes | 0 |  |
| circulation_overrides | 0 | -1 | 16 kB | yes | 0 |  |
| contacts | 0 | -1 | 96 kB | yes | 0 |  |
| outreach | 0 | -1 | 48 kB | yes | 0 |  |
| schools | 0 | -1 | 48 kB | yes | 0 |  |
| settings | 0 | -1 | 32 kB | yes | 0 |  |
| suppressions | 0 | -1 | 32 kB | yes | 0 |  |

## Schema: brain

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| knowledge_assets | 0 | -1 | 32 kB | yes | 1 | Brochures and links the concierge can attach. EMPTY in the source as of 6 August 2026, so the type CHECK list is untested against real data. |
| knowledge_base | 0 | -1 | 88 kB | yes | 1 | Key/value facts an agent may state. Text primary key on purpose: `key` IS the identity. Read-only to authenticated by design — a wrong value here is quoted to a parent (non-negotiable #5). |
| knowledge_cadence | 0 | -1 | 48 kB | yes | 1 | Sequence timing as data. Says when a step is DUE, never that it may be sent — the gate's frequency check overrides this table. |
| knowledge_faq | 0 | -1 | 48 kB | yes | 1 |  |
| knowledge_intents | 0 | -1 | 48 kB | yes | 1 | What a reply means, how hot it is, what to do and how long you have. Twelve rows distilled from 27,236 real emails in the meetings platform. Ported unchanged — rebuilding it means re-reading those emails. |
| knowledge_org | 0 | -1 | 32 kB | yes | 1 |  |
| knowledge_programs | 0 | -1 | 48 kB | yes | 1 | What an agent may SAY about a programme. Not public.programmes, which is the sellable catalogue with prices and seats. Same subject, different job. |
| knowledge_scenarios | 0 | -1 | 48 kB | yes | 1 |  |
| knowledge_voice_rules | 0 | -1 | 32 kB | yes | 1 | always / never / positioning / hard_rule. The kind decides the response to a violation: rewrite silently, or escalate to a human. |

## Schema: leads

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| agent_decisions | 0 | 915 | 400 kB | yes | 5 |  |
| agent_jobs | 0 | -1 | 80 kB | yes | 3 |  |
| agent_rate_limits | 0 | -1 | 16 kB | yes | 2 |  |
| deal_events | 0 | 255 | 152 kB | yes | 2 |  |
| deals | 0 | 242 | 376 kB | yes | 4 | SOP v1.0 section 6. One record per lead per programme. leads.meetings.deal_id links the activity log to it. |
| email_events | 0 | 735 | 376 kB | yes | 5 |  |
| email_sends | 0 | 711 | 4144 kB | yes | 5 |  |
| email_suppression | 0 | 50 | 48 kB | yes | 7 |  |
| email_templates | 0 | -1 | 80 kB | yes | 7 |  |
| invitations | 0 | -1 | 24 kB | yes | 2 |  |
| knowledge_assets | 0 | -1 | 24 kB | yes | 7 |  |
| knowledge_base | 0 | -1 | 16 kB | yes | 7 |  |
| knowledge_cadence | 0 | -1 | 24 kB | yes | 7 |  |
| knowledge_faq | 0 | -1 | 16 kB | yes | 7 |  |
| knowledge_intents | 0 | -1 | 24 kB | yes | 7 |  |
| knowledge_org | 0 | -1 | 16 kB | yes | 7 |  |
| knowledge_programs | 0 | -1 | 24 kB | yes | 7 |  |
| knowledge_scenarios | 0 | -1 | 24 kB | yes | 7 |  |
| knowledge_voice_rules | 0 | -1 | 16 kB | yes | 7 |  |
| lead_ai_calls | 0 | 205 | 152 kB | yes | 5 |  |
| applications | 0 | 721 | 536 kB | yes | 5 |  |
| meeting_assignees | 0 | -1 | 64 kB | yes | 5 |  |
| meeting_sync_queue | 0 | 4155 | 656 kB | yes | 2 |  |
| meetings | 0 | 390 | 4840 kB | yes | 7 |  |
| notifications | 0 | -1 | 48 kB | yes | 6 |  |
| program_cohorts | 0 | -1 | 48 kB | yes | 5 |  |
| scheduled_emails | 0 | 167 | 80 kB | yes | 5 |  |
| school_aliases | 0 | 335 | 128 kB | yes | 7 |  |
| school_merge_log | 0 | -1 | 16 kB | yes | 2 |  |
| school_registry | 0 | 400 | 224 kB | yes | 7 |  |
| sop_academic_calendar | 0 | 144 | 32 kB | yes | 2 | SOP Appendix B. No outbound campaign launches into a Break month; parked leads wake in an Active one. |
| sop_lists | 0 | 98 | 64 kB | yes | 2 | SOP v1.0 extensible lists (programme, region, sla, cadence, labels). Fixed lists are CHECKs on leads.deals. |
| sop_pricing_floors | 0 | -1 | 32 kB | yes | 2 | SOP 4.2. Below floor needs written approval before the offer is made, not after. |
| sop_targets | 0 | -1 | 32 kB | yes | 2 | SOP 8.4. Paid-revenue targets and the meeting load they imply. |
| sync_state | 0 | -1 | 48 kB | yes | 5 |  |
| tasks | 0 | 2316 | 1512 kB | yes | 5 |  |
| user_profiles | 0 | -1 | 32 kB | yes | 6 |  |
| webhook_logs | 0 | 11064 | 12 MB | yes | 2 |  |
| weekly_review_runs | 0 | -1 | 64 kB | yes | 2 |  |

## Schema: group

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| _wa_resend_audit_20260918 | 0 | -1 | 80 kB | yes | 1 |  |
| agent_decisions | 10 | 13481 | 6824 kB | yes | 2 |  |
| agent_email_examples | 0 | -1 | 144 kB | yes | 0 |  |
| agent_gmail_drafts | 0 | 128 | 544 kB | yes | 1 |  |
| agent_jobs | 0 | -1 | 80 kB | yes | 2 |  |
| agent_rate_limits | 0 | -1 | 16 kB | yes | 1 |  |
| ai_settings | 0 | -1 | 32 kB | yes | 2 |  |
| application_school_matches | 1 | 3310 | 1528 kB | yes | 1 |  |
| attention_state | 0 | -1 | 16 kB | yes | 4 |  |
| automations | 0 | 18 | 64 kB | yes | 2 |  |
| bf_assignee | 0 | 585 | 104 kB | yes | 1 |  |
| bf_insert | 0 | 304 | 160 kB | yes | 1 |  |
| bf_update | 0 | 893 | 160 kB | yes | 1 |  |
| calendly_question_registry | 13 | 13 | 104 kB | yes | 2 |  |
| campaign_registry | 0 | 416 | 136 kB | yes | 2 |  |
| core_applications | 3 | 13753 | 14 MB | yes | 1 |  |
| core_applications_removed_20261003 | 0 | 52 | 96 kB | yes | 0 |  |
| core_bank_refunds | 13 | 13 | 32 kB | yes | 1 |  |
| core_payments | 15872 | 15872 | 24 MB | yes | 1 |  |
| core_pull_state | 0 | 2 | 64 kB | yes | 1 |  |
| deal_events | 0 | 2612 | 968 kB | yes | 2 |  |
| deal_merge_log | 0 | 342 | 152 kB | yes | 1 |  |
| deals | 0 | 2123 | 2504 kB | yes | 4 |  |
| email_body_repair_attempts | 0 | 2801 | 592 kB | yes | 0 |  |
| email_events | 24 | 4611 | 2496 kB | yes | 2 |  |
| email_sends | 27 | 48190 | 3185 MB | yes | 2 |  |
| email_settings | 0 | -1 | 32 kB | yes | 2 |  |
| email_suppression | 1 | 191 | 136 kB | yes | 4 |  |
| email_templates | 0 | -1 | 136 kB | yes | 4 |  |
| followup_plans | 0 | -1 | 48 kB | yes | 3 |  |
| followup_report_runs | 0 | 19 | 48 kB | yes | 1 |  |
| followup_report_settings | 0 | -1 | 32 kB | yes | 2 |  |
| followup_touches | 0 | -1 | 80 kB | yes | 4 |  |
| gmail_account_requests | 0 | -1 | 24 kB | yes | 1 |  |
| invitations | 0 | -1 | 24 kB | yes | 1 |  |
| knowledge_assets | 0 | -1 | 24 kB | yes | 4 |  |
| knowledge_base | 0 | -1 | 128 kB | yes | 4 |  |
| knowledge_cadence | 0 | -1 | 48 kB | yes | 4 |  |
| knowledge_faq | 0 | -1 | 32 kB | yes | 4 |  |
| knowledge_intents | 0 | 12 | 80 kB | yes | 4 |  |
| knowledge_org | 0 | -1 | 32 kB | yes | 4 |  |
| knowledge_programs | 0 | 27 | 80 kB | yes | 4 |  |
| knowledge_scenarios | 0 | 12 | 80 kB | yes | 4 |  |
| knowledge_voice_rules | 0 | 21 | 64 kB | yes | 4 |  |
| lead_ai_calls | 0 | 1520 | 864 kB | yes | 2 |  |
| applications | 0 | 721 | 640 kB | yes | 2 |  |
| meeting_assignees | 30 | 5810 | 2296 kB | yes | 5 |  |
| meeting_edit_log | 0 | -1 | 48 kB | yes | 1 |  |
| meeting_followups | 0 | 445 | 3408 kB | yes | 1 |  |
| meeting_participants | 120 | 9236 | 4024 kB | yes | 1 |  |
| meeting_programme_interest | 0 | -1 | 40 kB | yes | 4 |  |
| meeting_report_runs | 0 | 28 | 96 kB | yes | 1 |  |
| meeting_report_settings | 0 | -1 | 32 kB | yes | 2 |  |
| meeting_sync_queue | 366 | 17250 | 4728 kB | yes | 1 |  |
| meeting_welcome_jobs | 953 | 953 | 488 kB | yes | 2 |  |
| meeting_zoom_links | 0 | -1 | 48 kB | yes | 2 |  |
| meetings | 7028 | 7027 | 112 MB | yes | 4 |  |
| meetings_sheet_sync_state | 1 | 1 | 464 kB | yes | 1 |  |
| member_daily_email_runs | 11 | -1 | 48 kB | yes | 1 |  |
| member_daily_email_settings | 0 | -1 | 32 kB | yes | 1 |  |
| member_email_agent | 0 | 24 | 72 kB | yes | 1 |  |
| member_report_days | 575 | 575 | 3032 kB | yes | 1 |  |
| member_sheet_reports | 24 | 24 | 112 kB | yes | 1 |  |
| notifications | 102001 | 101393 | 109 MB | yes | 2 |  |
| program_cohorts | 0 | 31 | 88 kB | yes | 2 |  |
| review_commitments | 0 | -1 | 24 kB | yes | 4 |  |
| scheduled_emails | 17 | 2553 | 1208 kB | yes | 2 |  |
| school_aliases | 3 | 3605 | 1088 kB | yes | 4 |  |
| school_domains | 0 | 579 | 264 kB | yes | 2 |  |
| school_match_log | 0 | 935 | 256 kB | yes | 0 |  |
| school_memory | 0 | -1 | 80 kB | yes | 4 |  |
| school_merge_log | 0 | -1 | 32 kB | yes | 1 |  |
| school_ownership | 0 | -1 | 32 kB | yes | 4 |  |
| school_registry | 0 | 3517 | 1552 kB | yes | 4 |  |
| school_research | 0 | 496 | 2080 kB | yes | 1 |  |
| sheet_import_stage | 0 | 3403 | 6968 kB | yes | 2 |  |
| sop_academic_calendar | 0 | 144 | 64 kB | yes | 2 |  |
| sop_lists | 0 | 107 | 72 kB | yes | 2 |  |
| sop_pricing_floors | 0 | -1 | 32 kB | yes | 2 |  |
| sop_targets | 0 | -1 | 32 kB | yes | 2 |  |
| staging_utm_restore_20261021 | 0 | 186 | 64 kB | yes | 0 |  |
| sync_runs | 2980 | 178355 | 128 MB | yes | 1 |  |
| sync_state | 0 | -1 | 48 kB | yes | 2 |  |
| system_health | 9 | 9 | 64 kB | yes | 1 |  |
| tasks | 10 | 12340 | 14 MB | yes | 3 |  |
| team_mailbox_import | 0 | -1 | 32 kB | yes | 1 |  |
| user_gmail_accounts | 24 | 24 | 624 kB | yes | 1 |  |
| user_profiles | 0 | 29 | 88 kB | yes | 4 |  |
| view_def_backup | 0 | -1 | 16 kB | yes | 0 |  |
| wa_agent_runs | 0 | 21 | 104 kB | yes | 1 |  |
| wa_claim_taps | 0 | -1 | 32 kB | yes | 0 |  |
| wa_inbound | 0 | 27 | 96 kB | yes | 1 |  |
| wa_limits | 0 | -1 | 32 kB | yes | 2 |  |
| wa_poll_state | 4 | 4 | 64 kB | yes | 1 |  |
| wa_sends | 43 | 2777 | 15 MB | yes | 1 |  |
| wa_session_window | 0 | 21 | 64 kB | yes | 1 |  |
| wa_settings | 0 | -1 | 32 kB | yes | 2 |  |
| wa_templates | 0 | -1 | 64 kB | yes | 2 |  |
| webhook_logs | 459 | 34847 | 59 MB | yes | 1 |  |
| weekly_commitments | 0 | -1 | 24 kB | yes | 4 |  |
| weekly_review_runs | 0 | -1 | 96 kB | yes | 1 |  |

## Schema: migration

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| baseline_20260902 | 0 | -1 | 16 kB | NO | 0 | Money baseline per provider/currency/status as at 2026-09-02, before repair. Any later divergence must be explainable by a documented phase. |
| id_map | 0 | 47629 | 21 MB | yes | 0 | Every migrated row maps back to its source. Required, not optional (D-018). A transform that does not write here cannot be safely re-run or undone. |
| payments_backup_20260902 | 0 | 10678 | 5160 kB | NO | 0 | Full copy of public.payments taken 2026-09-02 before the cut-over repair (duplicate removal, provider relabel, unique key). Restore source for rollback. |
| payments_quarantine_20260902 | 0 | 73 | 72 kB | NO | 0 | The 73 mislabelled Stripe duplicates identified in public.payments on 2026-09-02. Full original rows plus the id of the row that should survive. Restore source. |
| razorpay_date_corrections | 0 | 300 | 136 kB | NO | 0 | SUPERSEDED 3 Oct 2026 by migration.razorpay_true_local (dates from the payment id). These corrections fixed day/month but were 5h30 off. Kept for history; nothing reads it. |
| razorpay_programme_rules | 0 | -1 | 32 kB | NO | 0 | Which programme a Razorpay payment belongs to, by the app_name in its notes and its date. Read by migration.sync_razorpay_payments. Add a row per new edition. |
| run_log | 124 | 8447 | 2872 kB | yes | 0 | One row per wave execution. An open row (finished_at IS NULL) means a wave started and did not report completion — check it before starting another. |

## Schema: public

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| activities (partitioned) | 0 | -1 | 0 bytes | yes | 1 |  |
| activities_2026_08 (partition) | 0 | 1157 | 800 kB | yes | 1 |  |
| activities_2026_09 (partition) | 0 | -1 | 112 kB | yes | 1 |  |
| activities_2026_10 (partition) | 0 | -1 | 80 kB | yes | 1 |  |
| activities_default (partition) | 0 | 5815 | 3664 kB | yes | 1 |  |
| affiliations | 0 | 1464 | 528 kB | yes | 1 |  |
| agent_runs | 0 | 888 | 592 kB | yes | 1 |  |
| agent_setting_history | 0 | -1 | 24 kB | yes | 0 |  |
| agent_settings | 0 | -1 | 32 kB | yes | 0 | Operating parameters for the background agents — floors, batch sizes, caps. Every row carries why it is that value. Sending autonomy is NOT here: non-negotiable 3 keeps that at 0 and out of reach. |
| allocations | 0 | -1 | 64 kB | yes | 0 |  |
| applications | 2 | 12940 | 37 MB | yes | 0 | Unified programme applications, one row per row of the eleven *_applications tables in the legacy project. Loaded and kept current through legacy_ro. raw holds the complete original row; unique(source_table, source_row_id) makes the hourly sync safe to re-run. |
| approved_facts | 0 | -1 | 32 kB | yes | 1 |  |
| autonomy_thresholds | 0 | -1 | 48 kB | yes | 1 | Per agent, per segment. Every row starts at 0 (D-006). The CHECK makes "autonomy is earned" a database guarantee, not only a code path. |
| bank_refunds | 0 | -1 | 32 kB | yes | 0 | Refunds paid by bank transfer (Wallex). payment_id is set only when exactly one payment matches beneficiary name, currency and programme; otherwise the refund still counts against its programme and is flagged. |
| campaigns | 0 | -1 | 48 kB | yes | 1 |  |
| cohorts | 0 | 51 | 96 kB | yes | 1 |  |
| consents | 0 | 483 | 264 kB | yes | 1 |  |
| frequency_counters | 0 | -1 | 16 kB | yes | 0 |  |
| household_members | 0 | 21096 | 3672 kB | yes | 1 |  |
| households | 0 | 10256 | 1504 kB | yes | 1 |  |
| inbound_records | 0 | 7493 | 19 MB | yes | 1 | Every inbound write from every app lands here and nowhere else. A row is a claim, not a fact. 030. |
| jobs | 0 | -1 | 80 kB | yes | 0 |  |
| li_connections | 0 | -1 | 32 kB | yes | 1 | Manually reviewed professional networking prospects; no automatic invitation dispatch. |
| li_content_posts | 0 | -1 | 32 kB | yes | 1 | LinkedIn post drafts and publication state; publishing requires a separate authorized integration. |
| li_daily_runs | 0 | -1 | 24 kB | yes | 1 | Run history for research and weekly editorial workflows. |
| li_linkedin_connections | 0 | -1 | 16 kB | yes | 1 |  |
| li_linkedin_oauth_states | 0 | -1 | 24 kB | yes | 0 |  |
| li_research_items | 0 | -1 | 32 kB | yes | 1 | Source-linked public research and editorial intelligence. |
| li_work_updates | 0 | -1 | 24 kB | yes | 1 | LinkedIn editorial inputs; avoid confidential business details. |
| meetings | 0 | 374 | 2624 kB | yes | 0 |  |
| memberships | 0 | -1 | 128 kB | yes | 1 |  |
| merge_lineage | 0 | -1 | 64 kB | yes | 1 |  |
| messages | 0 | 881 | 2944 kB | yes | 1 |  |
| org_aliases | 0 | 55797 | 23 MB | yes | 1 |  |
| org_group_domains | 0 | 6666 | 1624 kB | yes | 0 | Every domain a group reaches its schools under. A district commonly has two or three. The domain is the primary key because one domain must belong to exactly one group — otherwise contact_ownership has two answers for one address. |
| org_groups | 0 | 5960 | 3704 kB | yes | 0 | A school group, district or university system: one relationship covering many organisations. Exists so a country campaign mails the relationship once instead of mailing 188 GEMS schools separately, which is the complaint on record. |
| organizations | 0 | 58362 | 67 MB | yes | 1 |  |
| payments | 19 | 15715 | 26 MB | yes | 0 | What customers actually paid. Eight currencies, stored as minor units plus a currency code — nothing is converted at rest (D-007, D-063). Refunds are separate rows in `refunds`, never a status string (D-014). |
| people | 0 | 32728 | 30 MB | yes | 1 |  |
| person_identities | 0 | 42908 | 18 MB | yes | 1 |  |
| programme_aliases | 0 | -1 | 80 kB | yes | 0 |  |
| programme_codes | 0 | 293 | 136 kB | yes | 0 | Finance catalogue of cohort codes (legacy finance.program_codes), each segment named from finance.codes_dropdown. Kept current by migration.sync_programme_codes(). payments.programme_code and applications.programme_code join here. |
| programmes | 0 | -1 | 96 kB | yes | 1 |  |
| refunds | 0 | 372 | 240 kB | yes | 0 | A refund is an event with its own amount (D-014). The legacy database put these in prose inside stripe_payments.status across 14 distinct strings. |
| research_activity | 0 | 81746 | 25 MB | yes | 1 |  |
| research_agent_runs | 0 | -1 | 96 kB | yes | 1 |  |
| research_agent_settings | 0 | -1 | 32 kB | yes | 1 |  |
| research_allocations | 0 | -1 | 48 kB | yes | 1 |  |
| research_alumni | 0 | 707 | 960 kB | yes | 2 |  |
| research_applications | 0 | 946 | 3208 kB | yes | 1 |  |
| research_audit_blank_city | 0 | 132017 | 7200 kB | yes | 0 |  |
| research_audit_city_x | 0 | 29341 | 2832 kB | yes | 0 |  |
| research_audit_cityref | 0 | 134377 | 19 MB | yes | 0 |  |
| research_audit_cs | 0 | 11962 | 944 kB | yes | 0 |  |
| research_audit_csc | 0 | 29341 | 3472 kB | yes | 0 |  |
| research_audit_school_city | 0 | 44606 | 4496 kB | yes | 0 |  |
| research_autopilot | 0 | 22 | 64 kB | yes | 1 |  |
| research_b2c_contacts | 0 | -1 | 32 kB | yes | 1 |  |
| research_b2c_people | 0 | 0 | 224 kB | yes | 1 |  |
| research_backfill_cursor | 0 | -1 | 32 kB | yes | 0 |  |
| research_campaign_contacts | 0 | -1 | 64 kB | yes | 1 |  |
| research_campaign_reports | 0 | -1 | 24 kB | yes | 1 |  |
| research_campaigns | 0 | -1 | 32 kB | yes | 1 |  |
| research_cell_flags | 0 | 67 | 168 kB | yes | 1 |  |
| research_city_targets | 0 | 57166 | 22 MB | yes | 0 |  |
| research_claims | 0 | -1 | 32 kB | yes | 1 |  |
| research_cleaning_runs | 1 | 55 | 128 kB | yes | 0 |  |
| research_contact_facets | 0 | 600767 | 301 MB | yes | 0 |  |
| research_contact_health | 600772 | 600772 | 131 MB | yes | 0 |  |
| research_contact_segments | 0 | -1 | 32 kB | yes | 1 |  |
| research_contacts | 0 | 599126 | 1654 MB | yes | 1 |  |
| research_contrib_cursor | 1 | 1 | 56 kB | yes | 0 |  |
| research_data_health | 0 | -1 | 32 kB | yes | 0 |  |
| research_datasets | 0 | -1 | 48 kB | yes | 1 |  |
| research_derived_map | 0 | 24443 | 4008 kB | yes | 0 |  |
| research_designation_proposals | 0 | -1 | 56 kB | yes | 1 |  |
| research_district_directories | 0 | 661 | 296 kB | yes | 1 |  |
| research_district_scans | 0 | 30 | 32 kB | yes | 0 |  |
| research_domain_checks | 0 | -1 | 32 kB | yes | 1 |  |
| research_domain_known | 0 | 100722 | 12 MB | yes | 0 |  |
| research_domain_names | 0 | 200787 | 48 MB | yes | 0 |  |
| research_domain_sites | 0 | 20391 | 4608 kB | yes | 1 |  |
| research_duplicate_log | 0 | 96817 | 73 MB | yes | 1 |  |
| research_email_check_requests | 0 | 1336 | 240 kB | yes | 0 |  |
| research_email_check_runs | 0 | 294 | 144 kB | yes | 1 |  |
| research_email_check_state | 0 | 1 | 64 kB | yes | 1 |  |
| research_email_checks | 0 | 5595 | 4040 kB | yes | 1 |  |
| research_email_health | 0 | 0 | 104 kB | yes | 1 |  |
| research_email_templates | 0 | -1 | 32 kB | yes | 1 |  |
| research_events | 0 | -1 | 24 kB | yes | 1 |  |
| research_exports | 0 | -1 | 16 kB | yes | 0 |  |
| research_fact_fill_cols | 0 | -1 | 32 kB | yes | 0 |  |
| research_fact_fill_plan | 0 | 6983 | 2936 kB | yes | 0 |  |
| research_fact_fill_schools | 0 | 5625 | 472 kB | yes | 0 |  |
| research_field_changes | 0 | 709001 | 260 MB | yes | 1 |  |
| research_field_defs | 0 | -1 | 24 kB | yes | 1 |  |
| research_fix_cityst | 0 | 31067 | 3184 kB | yes | 0 |  |
| research_fix_g22 | 0 | 360 | 40 kB | yes | 0 |  |
| research_fix_g45 | 0 | 213 | 40 kB | yes | 0 |  |
| research_fix_g45src | 0 | 29635 | 2152 kB | yes | 0 |  |
| research_fix_g6 | 0 | 379 | 72 kB | yes | 0 |  |
| research_fix_plan | 0 | 146552 | 41 MB | yes | 0 |  |
| research_fix_r1 | 0 | 4635 | 552 kB | yes | 0 |  |
| research_fix_r6 | 0 | -1 | 16 kB | yes | 0 |  |
| research_fix_typo | 0 | 520 | 72 kB | yes | 0 |  |
| research_fix_web | 0 | 110663 | 12 MB | yes | 0 |  |
| research_fix_web_a | 0 | 31830 | 12 MB | yes | 0 |  |
| research_folder_members | 0 | 138632 | 29 MB | yes | 0 |  |
| research_folders | 8 | 8 | 80 kB | yes | 0 |  |
| research_generated_docs | 0 | -1 | 32 kB | yes | 1 |  |
| research_groups | 0 | -1 | 48 kB | yes | 1 |  |
| research_identity_matches | 0 | 371 | 176 kB | yes | 1 |  |
| research_intel_chats | 0 | -1 | 48 kB | yes | 1 |  |
| research_intel_messages | 0 | -1 | 64 kB | yes | 1 |  |
| research_job_locks | 0 | 2 | 64 kB | yes | 1 |  |
| research_jobs | 0 | 11972 | 4232 kB | yes | 1 |  |
| research_leads | 0 | -1 | 48 kB | yes | 1 |  |
| research_legacy_source_map | 0 | -1 | 32 kB | yes | 0 |  |
| research_library_cursor | 1 | 1 | 56 kB | yes | 0 |  |
| research_locations | 0 | -1 | 48 kB | yes | 1 |  |
| research_optouts | 0 | -1 | 32 kB | yes | 1 |  |
| research_outreach_claims | 0 | -1 | 32 kB | yes | 1 |  |
| research_pages | 0 | 41476 | 201 MB | yes | 1 |  |
| research_people | 996 | 382179 | 203 MB | yes | 1 |  |
| research_person_emails | 996 | 432134 | 147 MB | yes | 1 |  |
| research_person_events | 996 | 967832 | 355 MB | yes | 1 |  |
| research_person_links | 0 | 1932 | 640 kB | yes | 1 |  |
| research_person_phones | 0 | 5576 | 1560 kB | yes | 1 |  |
| research_person_schools | 996 | 475128 | 143 MB | yes | 1 |  |
| research_pipeline_links | 0 | 9005 | 2672 kB | yes | 1 |  |
| research_place_fill_plan | 0 | 37038 | 9760 kB | yes | 0 |  |
| research_proposals | 0 | 697 | 448 kB | yes | 1 |  |
| research_raw_rows | 0 | 764682 | 785 MB | yes | 1 |  |
| research_reach_cache | 0 | 55 | 4480 kB | yes | 0 |  |
| research_record_actions | 0 | -1 | 80 kB | yes | 3 |  |
| research_ref_cities | 0 | 148038 | 29 MB | yes | 0 |  |
| research_ref_countries | 0 | 250 | 64 kB | yes | 0 |  |
| research_ref_states | 0 | 4961 | 1216 kB | yes | 0 |  |
| research_registry_dry | 0 | -1 | 16 kB | yes | 0 |  |
| research_registry_schools | 0 | 121494 | 74 MB | yes | 1 |  |
| research_resolve_items | 0 | 262174 | 301 MB | yes | 1 |  |
| research_resolve_runs | 0 | 5719 | 1984 kB | yes | 1 |  |
| research_review_domain_school | 0 | 11097 | 2616 kB | yes | 0 |  |
| research_review_outcomes | 0 | 119221 | 103 MB | yes | 1 |  |
| research_review_rows | 0 | 0 | 12 MB | yes | 1 |  |
| research_saved_views | 0 | -1 | 32 kB | yes | 1 |  |
| research_schema_export_test | 0 | -1 | 56 kB | yes | 0 |  |
| research_school_facts | 0 | 157858 | 59 MB | yes | 1 |  |
| research_school_findings | 0 | 43431 | 19 MB | yes | 1 |  |
| research_school_intel | 0 | -1 | 24 kB | yes | 1 |  |
| research_school_link_plan | 0 | 242693 | 62 MB | yes | 0 |  |
| research_school_links | 0 | 223398 | 40 MB | yes | 0 |  |
| research_school_notes | 0 | -1 | 24 kB | yes | 1 |  |
| research_school_profiles | 0 | -1 | 64 kB | yes | 1 |  |
| research_school_web | 0 | -1 | 32 kB | yes | 1 |  |
| research_schools | 16 | 211722 | 86 MB | yes | 1 |  |
| research_segments | 0 | -1 | 24 kB | yes | 1 |  |
| research_send_exports | 0 | -1 | 48 kB | yes | 1 |  |
| research_settings | 0 | 18 | 272 kB | yes | 1 |  |
| research_short_links | 0 | -1 | 24 kB | yes | 1 |  |
| research_sop_acknowledgements | 0 | -1 | 40 kB | yes | 1 |  |
| research_sop_versions | 0 | -1 | 48 kB | yes | 1 |  |
| research_source_tools | 0 | -1 | 32 kB | yes | 0 |  |
| research_spend | 0 | 994 | 760 kB | yes | 1 |  |
| research_staff_profiles | 0 | -1 | 64 kB | yes | 1 |  |
| research_stats_contributions | 2667 | 2667 | 840 kB | yes | 0 |  |
| research_stats_contributions_meta | 0 | 1 | 56 kB | yes | 0 |  |
| research_stats_email_checks | 0 | 1 | 56 kB | yes | 0 |  |
| research_tasks | 0 | 140494 | 76 MB | yes | 1 |  |
| research_territory_briefs | 0 | -1 | 24 kB | yes | 1 |  |
| research_tracking_events | 0 | -1 | 32 kB | yes | 1 |  |
| research_tracking_tokens | 0 | -1 | 24 kB | yes | 1 |  |
| research_trust_totals_cache | 0 | 18 | 64 kB | yes | 0 |  |
| research_upload_tabs | 0 | 10 | 208 kB | yes | 1 |  |
| research_uploads | 0 | 2 | 64 kB | yes | 1 |  |
| research_us_states | 0 | 52 | 32 kB | yes | 0 |  |
| research_view_snapshots | 0 | -1 | 16 kB | yes | 1 |  |
| schema_migrations | 0 | 95 | 64 kB | yes | 0 |  |
| send_attempts | 0 | -1 | 80 kB | yes | 1 |  |
| sensitive_access_log | 0 | -1 | 80 kB | yes | 0 | Who looked at what, under which grant. Append-only. The log outlives the staff row on purpose — offboarding somebody must not erase what they read. |
| sequence_steps | 0 | -1 | 24 kB | yes | 1 |  |
| sequences | 0 | -1 | 16 kB | yes | 1 |  |
| source_codes | 0 | -1 | 80 kB | yes | 1 | Registry of minted source codes and who answers for each. The gate reads owner for R-C5's fourth check; a code with no row here cannot be messaged. |
| source_systems | 0 | -1 | 32 kB | yes | 1 | The systems permitted to appear in the SYSTEM segment of a source_code. Declares intent; connected_at records observed arrival. 028. |
| staff | 0 | 3 | 144 kB | yes | 0 | Who works here. The absence of a row is what separates a parent or student from an admin console — they share the same Supabase Auth pool. Rows are created by the invitation trigger or an owner, never by self-service. |
| staff_grants | 0 | -1 | 80 kB | yes | 0 | Elevated access, per person, with a written reason and an expiry. Separate from roles on purpose: a role says what you do, a grant says what you may see. has_grant() enforces expires_on, so lapsed access dies on the date. |
| staff_invitations | 0 | 0 | 64 kB | yes | 0 |  |
| staff_role_assignments | 0 | -1 | 56 kB | yes | 0 |  |
| staff_scopes | 0 | -1 | 48 kB | yes | 0 |  |
| staff_targets | 0 | -1 | 24 kB | yes | 0 | One row per staff member. 100 rows a day and 700 a week are the numbers named on the call; 20 meetings a month full-time, 10 part-time. All editable — they are a target, not a law. |
| suppressions | 0 | 130 | 96 kB | yes | 1 |  |
| tasks | 0 | 385 | 392 kB | yes | 1 |  |

## Schema: quarantine

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| dataset_rows | 0 | 0 | 224 kB | yes | 0 |  |
| datasets | 0 | -1 | 64 kB | yes | 0 | One version of one location's list. Credit for a meeting is derived from this and never typed: see meeting_dataset_pairs(). |
| email_domain_evidence | 0 | 211132 | 60 MB | yes | 0 | One row per domain per method. 106,728 domains carry 647,599 addresses, so a domain fact is stored once and joined, never copied onto every address that shares it. |
| email_evidence | 0 | 46456 | 18 MB | yes | 0 | Append-only. One row per check ever performed on an address. The verdict is derived from these by email_sendability() and is never stored — a stored flag is what let an April MX check still read as "valid" in August. |
| geo_aliases | 0 | 11125 | 7752 kB | yes | 0 | One row per distinct state_raw value, with what the machine could work out and what it had to ask. Ordered by rows_affected because 200 of these cover 79% of the data and 8,823 of them cover 3.5%. |
| import_batches | 0 | -1 | 32 kB | yes | 0 |  |
| import_header_maps | 0 | -1 | 32 kB | yes | 0 |  |
| import_rows | 0 | 3078 | 5808 kB | yes | 0 | One spreadsheet row, stored verbatim under the file's own headers. The mapping is applied when read, never on the way in, so a wrong guess is a correction rather than a re-upload. |
| org_candidates | 0 | 107674 | 68 MB | yes | 0 | Email domains where two or more distinct people work, with the employer name they state. NOT organisations. A candidate becomes one only by promotion, and the CHECK refuses to promote a name the people disagree on or a domain we already hold. There is no merge path for organisations, which is why this gate exists. 1,646 landed by w13_org_identity: 1,345 unanimous, 301 conflicted, 1,326 promotable, 21 linkable. |
| rejected_identities | 0 | 206 | 88 kB | yes | 0 | Identities that cannot produce a canonical key — malformed emails, national phone numbers with no country. Kept rather than deleted: several are recoverable by a human, and the raw value is the only evidence of what arrived. 038. |
| research_contact_edits | 0 | -1 | 64 kB | yes | 0 | One row per field corrected by hand, with the reason. Read by research_review_queue() to surface repeated corrections — the same fix applied forty times is a patch waiting to be written, not a habit to be continued (SOP house rule 7). |
| research_contacts | 0 | 647650 | 508 MB | yes | 0 | 647,650 scraped contacts, 98.9% never validated. NOT people. A row becomes a person only by promotion, and the CHECK refuses to promote one that has not validated. Never an identity source (D-011). |
| research_schools | 0 | 175136 | 152 MB | yes | 0 | One row per distinct school name in research_contacts. A rollup, not a source: rebuilt by refresh_research_schools(). Exists because searching 647,650 wide rows for a school name cost 5.5s while searching 175,136 narrow ones is a lookup. 036. |
| validator_runs | 0 | 168 | 456 kB | yes | 0 |  |

## Schema: revenue_brain

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| account_program_fit | 0 | -1 | 32 kB | yes | 0 |  |
| approval_audit | 0 | 1189 | 344 kB | yes | 0 |  |
| autonomous_test_cases | 0 | 42 | 200 kB | yes | 0 |  |
| autonomous_test_runs | 0 | 24 | 88 kB | yes | 0 |  |
| brain_decisions | 0 | 1208 | 1128 kB | yes | 0 |  |
| campaign_routes | 0 | 16 | 80 kB | yes | 0 |  |
| claims_registry | 0 | 87 | 96 kB | yes | 0 |  |
| conversion_events | 0 | 55 | 112 kB | yes | 0 |  |
| conversion_test_cases | 0 | 12 | 80 kB | yes | 0 |  |
| conversion_test_runs | 0 | -1 | 48 kB | yes | 0 |  |
| eb_conversations | 0 | -1 | 40 kB | yes | 0 |  |
| eb_do_not_contact | 0 | -1 | 32 kB | yes | 0 |  |
| eb_events | 0 | -1 | 32 kB | yes | 0 |  |
| eb_leads | 0 | -1 | 32 kB | yes | 0 |  |
| eb_messages | 0 | -1 | 32 kB | yes | 0 |  |
| eb_notifications | 0 | -1 | 24 kB | yes | 0 |  |
| eb_reply_drafts | 0 | -1 | 24 kB | yes | 0 |  |
| eb_sync_state | 0 | -1 | 32 kB | yes | 0 |  |
| import_tokens | 0 | -1 | 32 kB | yes | 0 |  |
| orchestration_runs | 0 | 1392 | 2568 kB | yes | 0 |  |
| outbound_queue | 0 | 1235 | 1136 kB | yes | 0 |  |
| outreach_events | 0 | 169 | 472 kB | yes | 0 |  |
| pilot_copy_review_cases | 0 | -1 | 96 kB | yes | 0 |  |
| pilot_copy_review_runs | 0 | -1 | 48 kB | yes | 0 |  |
| pilot_preflight_cases | 0 | -1 | 96 kB | yes | 0 |  |
| pilot_preflight_runs | 0 | -1 | 48 kB | yes | 0 |  |
| programme_batch_leads | 0 | -1 | 24 kB | yes | 0 |  |
| programme_batches | 0 | -1 | 16 kB | yes | 0 |  |
| programme_profiles | 0 | -1 | 72 kB | yes | 0 |  |
| reply_handoffs | 0 | 46 | 328 kB | yes | 0 |  |
| revenue_attribution | 0 | -1 | 48 kB | yes | 0 |  |
| runtime_controls | 0 | -1 | 32 kB | yes | 0 |  |
| safety_test_cases | 0 | -1 | 80 kB | yes | 0 |  |
| safety_test_runs | 0 | -1 | 48 kB | yes | 0 |  |
| school_program_state | 0 | 650 | 216 kB | yes | 0 |  |
| sender_accounts | 0 | -1 | 32 kB | yes | 0 |  |
| sequence_steps | 0 | -1 | 160 kB | yes | 0 |  |
| sequence_templates | 0 | -1 | 48 kB | yes | 0 |  |
| shadow_cases | 0 | 45 | 192 kB | yes | 0 |  |
| shadow_runs | 0 | -1 | 80 kB | yes | 0 |  |
| source_evidence | 0 | 3099 | 1648 kB | yes | 0 |  |
| test_recipient_allowlist | 0 | -1 | 32 kB | yes | 0 |  |
| verification_events | 0 | 1154 | 488 kB | yes | 0 |  |
| webhook_inbox | 0 | 188 | 504 kB | yes | 0 |  |
| wow_internal_cert_cases | 0 | -1 | 48 kB | yes | 0 |  |
| wow_internal_cert_runs | 0 | -1 | 48 kB | yes | 0 |  |
| wow_pilot_case_release | 0 | -1 | 24 kB | yes | 0 |  |
| wow_pilot_release | 0 | -1 | 48 kB | yes | 0 |  |
| yale_contacts | 0 | 1441 | 22 MB | yes | 0 |  |
| yale_pilot_leads | 0 | 1147 | 1152 kB | yes | 0 |  |
| yale_school_codes | 0 | 651 | 272 kB | yes | 0 |  |

## Schema: staging

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| akbar_huso_rzp | 0 | 1275 | 184 kB | NO | 0 |  |
| akbar_offline_sheet | 0 | 13575 | 5232 kB | NO | 0 | Akbar's offline-programme master (13,575 rows, received 30 Sep 2026). Verbatim copy of the workbook; the *_norm and *_digits columns are derived. No amounts and no dates exist in the source. |
| akbar_payments_snapshot | 0 | 13220 | 3152 kB | NO | 0 | Point-in-time copy of public.payments taken for reconciling Akbar's offline sheet, 30 Sep 2026. Read-only reference; public.payments stays the source of truth. |
| akbar_prog_code | 0 | 139 | 40 kB | NO | 0 |  |
| akbar_prog_evidence | 0 | 524 | 136 kB | NO | 0 |  |
| akbar_prog_map | 0 | 136 | 40 kB | NO | 0 | Which payment descriptions belong to which programme in Akbar's sheet. Derived from shared emails, then every pair read by hand on 30 Sep 2026: 9 rejected as a different product at the same school, 1 held as probable pending Akbar. |
| akbar_sheet_emails | 0 | 24854 | 2768 kB | NO | 0 |  |
| akbar_verification | 0 | 13575 | 2272 kB | NO | 0 | Every row of Akbar's sheet checked against real payments, 30 Sep 2026. Each payment verifies at most one row. money is per currency in minor units, net of public.refunds. |
| akbar_verification_v1 | 0 | 13575 | 1904 kB | NO | 0 |  |
| app_household_of | 0 | 1204 | 232 kB | yes | 0 |  |
| app_pairs | 0 | 1273 | 112 kB | yes | 0 |  |
| applications | 0 | 1484 | 608 kB | yes | 0 |  |
| applications_removed_20261003 | 0 | 52 | 168 kB | NO | 0 |  |
| code_lead | 0 | 135 | 24 kB | NO | 0 |  |
| derived_codes | 0 | 1770 | 280 kB | NO | 0 |  |
| desc_code_evidence | 0 | 6642 | 1072 kB | NO | 0 |  |
| desc_family | 0 | 156 | 40 kB | NO | 0 |  |
| edge_claims | 0 | 111 | 24 kB | NO | 0 |  |
| edge_household_of | 0 | -1 | 0 bytes | NO | 0 |  |
| edge_pairs | 0 | -1 | 8192 bytes | NO | 0 |  |
| edge_people | 0 | -1 | 8192 bytes | NO | 0 |  |
| edge_rows | 0 | 81 | 24 kB | NO | 0 |  |
| edition_candidates | 0 | 309 | 40 kB | NO | 0 |  |
| email_suppression_raw | 0 | -1 | 16 kB | yes | 0 |  |
| family_backtest | 0 | 98 | 16 kB | NO | 0 |  |
| family_lead | 0 | 100 | 16 kB | NO | 0 |  |
| household_of | 0 | 1483 | 240 kB | yes | 0 |  |
| household_pairs | 0 | 1551 | 128 kB | yes | 0 |  |
| huso_household_of | 0 | 449 | 32 kB | NO | 0 |  |
| huso_norm | 0 | 574 | 416 kB | NO | 0 |  |
| huso_pairs | 0 | 483 | 32 kB | NO | 0 |  |
| huso_people | 0 | 1197 | 336 kB | NO | 0 |  |
| inbound_people | 0 | 2779 | 984 kB | yes | 0 |  |
| link_derived | 0 | 217 | 40 kB | NO | 0 |  |
| payer_household_of | 0 | 7811 | 576 kB | NO | 0 |  |
| payer_pairs | 0 | 8170 | 544 kB | NO | 0 |  |
| payer_people | 0 | 12404 | 2136 kB | NO | 0 |  |
| payments_raw | 0 | 10386 | 3552 kB | NO | 0 |  |
| phone_claims | 0 | 6324 | 584 kB | yes | 0 |  |
| phone_owner | 0 | 5436 | 552 kB | yes | 0 |  |
| plink_code_windows | 0 | 364 | 72 kB | NO | 0 |  |
| plink_codes | 0 | 271 | 72 kB | NO | 0 |  |
| plink_evidence | 0 | 2348 | 320 kB | NO | 0 |  |
| prospects | 0 | 12282 | 5784 kB | NO | 0 |  |
| prospects_raw | 0 | 14581 | 6184 kB | NO | 0 |  |
| school_aliases_raw | 0 | 573 | 104 kB | yes | 0 |  |
| school_registry_raw | 0 | 350 | 104 kB | yes | 0 |  |
| uncoded | 0 | 2445 | 424 kB | NO | 0 |  |
| w5b_linkedin_suppressions_hashes | 0 | 87 | 48 kB | yes | 0 |  |

## System schema: auth

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| audit_log_entries | 0 | -1 | 24 kB | yes | 0 | Auth: Audit trail for user actions. |
| custom_oauth_providers | 0 | -1 | 56 kB | NO | 0 |  |
| flow_state | 0 | -1 | 80 kB | yes | 0 | Stores metadata for all OAuth/SSO login flows |
| identities | 0 | 15 | 112 kB | yes | 0 | Auth: Stores identities associated to a user. |
| instances | 0 | -1 | 16 kB | yes | 0 | Auth: Manages users across multiple sites. |
| mfa_amr_claims | 0 | 7 | 120 kB | yes | 0 | auth: stores authenticator method reference claims for multi factor authentication |
| mfa_challenges | 0 | -1 | 24 kB | yes | 0 | auth: stores metadata about challenge requests made |
| mfa_factors | 0 | -1 | 56 kB | yes | 0 | auth: stores metadata about factors |
| mfa_recovery_code_sets | 0 | -1 | 24 kB | NO | 0 |  |
| mfa_recovery_codes | 0 | -1 | 24 kB | NO | 0 |  |
| oauth_authorizations | 0 | -1 | 40 kB | NO | 0 |  |
| oauth_client_states | 0 | -1 | 24 kB | NO | 0 | Stores OAuth states for third-party provider authentication flows where Supabase acts as the OAuth client. |
| oauth_clients | 0 | -1 | 24 kB | NO | 0 |  |
| oauth_consents | 0 | -1 | 48 kB | NO | 0 |  |
| one_time_tokens | 0 | -1 | 144 kB | yes | 0 |  |
| refresh_tokens | 18 | 142 | 440 kB | yes | 0 | Auth: Store of tokens used to refresh JWT tokens once they expire. |
| saml_providers | 0 | -1 | 32 kB | yes | 0 | Auth: Manages SAML Identity Provider connections. |
| saml_relay_states | 0 | -1 | 40 kB | yes | 0 | Auth: Contains SAML Relay State information for each Service Provider initiated login. |
| schema_migrations | 0 | 77 | 24 kB | yes | 0 | Auth: Manages updates to the auth system. |
| scim_tokens | 0 | -1 | 48 kB | NO | 0 |  |
| scim_users | 0 | -1 | 88 kB | NO | 0 |  |
| sessions | 0 | 14 | 144 kB | yes | 0 | Auth: Stores session data associated to a user. |
| sso_domains | 0 | -1 | 32 kB | yes | 0 | Auth: Manages SSO email address domain mapping to an SSO Identity Provider. |
| sso_providers | 0 | -1 | 32 kB | yes | 0 | Auth: Manages SSO identity provider information; see saml_providers for SAML. |
| users | 0 | 32 | 304 kB | yes | 0 | Auth: Stores user login data within a secure schema. |
| webauthn_challenges | 0 | -1 | 32 kB | NO | 0 |  |
| webauthn_credentials | 0 | -1 | 32 kB | NO | 0 |  |

## System schema: cron

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| job | 0 | 98 | 152 kB | yes | 1 |  |
| job_run_details | 0 | 47984 | 155 MB | yes | 1 |  |

## System schema: net

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| _http_response | 2785 | 2785 | 6240 kB | NO | 0 |  |
| http_request_queue | 26 | 19 | 80 kB | NO | 0 |  |

## System schema: realtime

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| messages (partitioned) | 0 | -1 | 0 bytes | yes | 0 |  |
| messages_2026_11_21 (partition) | 0 | -1 | 24 kB | NO | 0 |  |
| messages_2026_11_22 (partition) | 0 | -1 | 24 kB | NO | 0 |  |
| messages_2026_11_23 (partition) | 0 | -1 | 24 kB | NO | 0 |  |
| messages_2026_11_24 (partition) | 0 | -1 | 24 kB | NO | 0 |  |
| messages_2026_11_25 (partition) | 0 | -1 | 24 kB | NO | 0 |  |
| messages_2026_11_26 (partition) | 0 | -1 | 24 kB | NO | 0 |  |
| messages_2026_11_27 (partition) | 0 | -1 | 24 kB | NO | 0 |  |
| schema_migrations | 0 | 81 | 24 kB | NO | 0 |  |
| subscription | 0 | 0 | 72 kB | NO | 0 |  |

## System schema: storage

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| bucket_lifecycle_states | 0 | -1 | 24 kB | yes | 0 |  |
| buckets | 0 | -1 | 48 kB | yes | 0 |  |
| buckets_analytics | 0 | -1 | 24 kB | yes | 0 |  |
| buckets_vectors | 0 | -1 | 16 kB | yes | 0 |  |
| lifecycle_abandoned_artifacts | 0 | -1 | 24 kB | yes | 0 |  |
| migrations | 0 | 61 | 72 kB | yes | 0 |  |
| objects | 0 | -1 | 88 kB | yes | 12 |  |
| s3_multipart_uploads | 0 | -1 | 24 kB | yes | 0 |  |
| s3_multipart_uploads_parts | 0 | -1 | 24 kB | yes | 0 |  |
| vector_indexes | 0 | -1 | 24 kB | yes | 0 |  |

## System schema: supabase_migrations

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| schema_migrations | 1 | 345 | 904 kB | NO | 0 |  |

## System schema: vault

| table | n_live_tup | reltuples | total size | RLS | #policies | comment |
|---|---|---|---|---|---|---|
| secrets | 0 | -1 | 48 kB | NO | 0 | Table with encrypted `secret` column for storing sensitive information on disk. |

