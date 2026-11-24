# Brain project: size and health

Live snapshot taken 2026-11-24 from <DATA_REF> (PostgreSQL 17.6, ap-northeast-2).

## Headline numbers

| metric | value |
|---|---|
| database size | 10 GB |
| tables (all non-system schemas incl. cron/net/auth/storage/realtime/vault) | 522 |
| total index size (pg_stat_user_indexes) | 2662 MB |
| indexes never scanned (idx_scan = 0, excluding PK/unique) | 592 indexes, 1513 MB |
| buffer cache hit ratio (pg_stat_database) | 98.88 % |
| xact commit / rollback | 491,937 / 95,310 (16 % rollbacks -- high; the research pipeline and cron jobs use exception-driven control flow) |
| temp files / temp bytes | 17,590 / 77 GB (sorts and hashes spilling to disk; work_mem is 5 MB) |
| deadlocks / conflicts | 0 / 0 |
| connections | 27 (2 active, 18 idle, 7 background); max_connections 90 |
| postmaster start | 2026-11-23 11:32:52 UTC |
| pg_stat_database.stats_reset | null (counters nevertheless restarted with the postmaster; see caveat) |
| replication slots | 0 |
| settings | shared_buffers 512 MB (65536 x 8 kB), effective_cache_size 1.5 GB, work_mem 5 MB, maintenance_work_mem 128 MB, statement_timeout 120 s, autovacuum_vacuum_scale_factor 0.2, track_counts on |

**Statistics caveat**: the instance restarted 2026-11-23 11:32 UTC (cron.job_run_details shows "server restarted" at that minute). Cumulative counters (seq_scan, idx_scan, n_live_tup, n_tup_*) therefore cover only ~24 h, and n_live_tup is 0 for every table not touched since. Use reltuples for row estimates. last_autovacuum / last_autoanalyze are null for nearly every table -- those timestamps are also lost on restart, so "never analysed" below means "not since the restart", not "never".

## Top 30 tables by total size

| table | total | heap | indexes | toast | reltuples | n_live_tup | n_dead_tup | seq_scan | seq_tup_read | idx_scan | ins/upd/del since restart | last autovac / autoanalyze |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| group.email_sends | 3185 MB | 64 MB | 46 MB | **3074 MB** | 48,190 | 26 | 45 | 10,979 | 529,370,981 | 16,378,710 | 26/59/0 | - / - |
| public.research_contacts | 1654 MB | 842 MB | 812 MB | 288 kB | 599,126 | 0 | 1,078 | 1 | 38,515 | 3,044,406 | 0/1078/0 | - / - |
| public.research_raw_rows | 785 MB | 602 MB | 182 MB | 1336 kB | 764,682 | 0 | 0 | 0 | 0 | 1,080 | 0/0/0 | - / - |
| quarantine.research_contacts | 508 MB | 196 MB | 312 MB | 8 kB | 647,650 | 0 | 0 | 124 | 62,174,428 | 234,936 | 0/0/0 | - / - |
| public.research_person_events | 355 MB | 235 MB | 121 MB | 8 kB | 967,832 | 996 | 0 | 0 | 0 | 0 | 996/0/0 | - / - |
| public.research_contact_facets | 301 MB | 216 MB | 85 MB | 8 kB | 600,767 | 0 | 0 | 0 | 0 | 0 | 0/0/0 | - / - |
| public.research_resolve_items | 301 MB | 287 MB | 14 MB | 8 kB | 262,174 | 0 | 0 | 0 | 0 | 0 | 0/0/0 | - / - |
| public.research_field_changes | 260 MB | 164 MB | 95 MB | 8 kB | 709,001 | 0 | 0 | 0 | 0 | 0 | 0/0/0 | - / - |
| public.research_people | 203 MB | 91 MB | 112 MB | 24 kB | 382,179 | 996 | 0 | 0 | 0 | 4,980 | 996/0/0 | - / - |
| public.research_pages | 201 MB | 24 MB | 5 MB | 172 MB | 41,476 | 0 | 0 | 0 | 0 | 0 | 0/0/0 | - / - |
| cron.job_run_details | 155 MB | 150 MB | 5 MB | 8 kB | 47,984 (actual 156,190) | 0 | 0 | 202 | 29,938,110 | 0 | 0/0/0 | - / - |
| quarantine.research_schools | 152 MB | 76 MB | 76 MB | 8 kB | 175,136 | 0 | 0 | 0 | 0 | 0 | 0/0/0 | - / - |
| public.research_person_emails | 147 MB | 62 MB | 85 MB | 8 kB | 432,134 | 996 | 0 | 0 | 0 | 2,988 | 996/0/0 | - / - |
| public.research_person_schools | 143 MB | 73 MB | 70 MB | 8 kB | 475,128 | 996 | 0 | 0 | 0 | 1,992 | 996/0/0 | - / - |
| public.research_contact_health | 131 MB | 84 MB | 47 MB | 8 kB | 600,772 | 600,772 | **95,835** | 76 | 45,658,672 | 8,649,369 | 0/**6,777,947**/0 | - / 2026-11-24 |
| group.sync_runs | 128 MB | 94 MB | 34 MB | 8 kB | 178,355 | 2,969 | 0 | 0 | 0 | 18 | 2969/0/0 | - / - |
| group.meetings | 112 MB | 17 MB | 8 MB | 86 MB | 7,027 | 7,027 | 281 | 7,936 | 42,123,843 | 430,332 | 30/29,319/0 | 2026-11-24 / 2026-11-24 |
| group.notifications | 109 MB | 56 MB | 52 MB | 8 kB | 101,393 | 101,977 | 16,920 | 0 | 0 | 1,752 | 44,504/0/0 | 2026-11-24 / - |
| public.research_review_outcomes | 103 MB | 95 MB | 7 MB | 24 kB | 119,221 | 0 | 0 | 0 | 0 | 0 | 0/0/0 | - / - |
| public.research_schools | 86 MB | 41 MB | 45 MB | 8 kB | 211,722 | 16 | 0 | 0 | 0 | 4,388 | 16/0/0 | - / - |
| public.research_tasks | 76 MB | 48 MB | 28 MB | 72 kB | 140,494 | 0 | 0 | 0 | 0 | 294 | 0/0/0 | - / - |
| public.research_registry_schools | 74 MB | 48 MB | 26 MB | 8 kB | 121,494 | 0 | 0 | 0 | 0 | 0 | 0/0/0 | - / - |
| public.research_duplicate_log | 73 MB | 62 MB | 12 MB | 8 kB | 96,817 | 0 | 0 | 0 | 0 | 48 | 0/0/0 | - / - |
| quarantine.org_candidates | 68 MB | 40 MB | 28 MB | 8 kB | 107,674 | 0 | 0 | 8 | 861,392 | 28 | 0/0/0 | - / - |
| public.organizations | 67 MB | 40 MB | 26 MB | 8 kB | 58,362 | 0 | 0 | 8 | 236,144 | 14,064 | 0/0/0 | - / - |
| public.research_school_link_plan | 62 MB | 42 MB | 19 MB | 8 kB | 242,693 | 0 | 0 | 0 | 0 | 0 | 0/0/0 | - / - |
| quarantine.email_domain_evidence | 60 MB | 43 MB | 17 MB | 8 kB | 211,132 | 0 | 0 | 0 | 0 | 192 | 0/0/0 | - / - |
| public.research_school_facts | 59 MB | 49 MB | 10 MB | 8 kB | 157,858 | 0 | 0 | 0 | 0 | 0 | 0/0/0 | - / - |
| group.webhook_logs | 59 MB | 44 MB | 5 MB | 10 MB | 34,847 | 458 | 458 | 0 | 0 | 746 | 458/458/0 | - / - |
| public.research_domain_names | 48 MB | 29 MB | 19 MB | 8 kB | 200,787 | 0 | 0 | 0 | 0 | 0 | 0/0/0 | - / - |

## Observations

### group.email_sends is 3.1 GB of TOAST for ~48k rows
96 % of the table is TOAST: rendered_html / body_html / body_text / attachments for every synced Gmail message. It has had 10,979 sequential scans reading 529 M tuples in 24 h (something scans the whole table repeatedly -- candidates: the gmail-thread-sync / repair-email-bodies / member-sheet-report jobs and the statement-level triggers `email_sends_sheet_kick_*` and `member_report_email_*`, which fire on every insert/update). Together with 16.4 M index scans this is the hottest table in the database. Worth considering: moving bodies to Storage or a side table, or at least ensuring the sweeper queries hit indexes.

### research_contact_health churn
6.78 M updates since the restart on a 600k-row table (every row rewritten ~11 times per day) with 95,835 dead tuples (16 %); autoanalyze caught it today but autovacuum has not completed since restart. This is the validator/health scoring loop (spine_validator_dns every 15 min, group-library-tick every 2 min). Candidate for a lower autovacuum_vacuum_scale_factor on this table, or batching the updates.

### Unused indexes: 592 indexes, 1.5 GB never scanned since restart
The largest never-scanned indexes (idx_scan = 0 over ~24 h, so treat as "possibly unused", not proof):

| index | size |
|---|---|
| public.research_contacts.research_contacts_school_contact_trgm_live_idx | 121 MB |
| quarantine.research_contacts.research_contacts_source_uk (unique) | 88 MB |
| quarantine.research_contacts.research_contacts_school_trgm_ix | 63 MB |
| quarantine.research_contacts.research_contacts_school_trgm | 56 MB (duplicate of the above, same columns) |
| public.research_person_events.research_person_events_person_idx | 51 MB |
| public.research_contacts.research_contacts_school_realname_key (unique) | 51 MB |
| public.research_people.research_people_primary_email_trgm_live_idx | 46 MB |
| quarantine.research_schools.research_schools_trgm_ix | 41 MB |
| public.research_contacts.research_contacts_email_plain_idx | 39 MB |
| public.research_contacts.research_contacts_serial_live_idx | 36 MB |
| public.research_field_changes.research_field_changes_contact_idx | 36 MB |
| public.research_contacts.research_contacts_trust_school_idx | 32 MB |
| public.research_person_events.research_person_events_contact_idx | 29 MB |
| public.research_contacts.research_contacts_person_idx | 27 MB |
| public.research_raw_rows.research_raw_rows_pending_idx / _unprocessed_idx | 22 MB each (overlapping partial indexes) |

public.research_contacts alone carries 812 MB of indexes on an 842 MB heap, ~45 of them with zero scans in the window (facet/reach/trust/verify/completeness variants). public.organizations has both organizations_name_trgm and organizations_name_trgm_ix (8 MB each, same expression). Recommendation: capture pg_stat_user_indexes over at least a full week before dropping, then drop the clear duplicates first.

### Sequential-scan heavy small tables (hot loops)
| table | seq_scan | idx_scan | seq_tup_read | note |
|---|---|---|---|---|
| public.research_source_tools | 7,396,227 | 168,154 | 236 M | tiny lookup table read on every research_contacts insert/update by trigger research_contacts_tool_in; fine for a small table but it is the single most scanned relation |
| group.user_profiles | 3,421,730 | 1,055,315 | 98 M | hit by every RLS policy evaluation (is_admin_or_owner / is_active_staff); small table so seq scan is cheap, but the counts show how often policies fire |
| group.member_sheet_reports | 69,884 | 23 | 1.0 M | member-sheet-report job every 3 min |
| group.school_aliases | 8,415 | 6 | 33 M | resolve_school_codes / auto_link jobs scan aliases (3.5k rows) rather than using the lower(trim(raw_text)) unique index |
| group.scheduled_emails | 2,634 | 137 | 4.2 M | send-scheduled-emails every 15 min |
| cron.job_run_details | 202 | 0 | 30 M | 156k rows, no index use; add a retention job (e.g. delete rows older than 30 days) |

### Dead tuples
Only group.notifications (16,920 dead / 102k live, 44k inserts since restart, autovacuumed today) and research_contact_health exceed 10k dead tuples. No table crosses the 20 % and >10k threshold except research_contact_health (16 %, flagged above).

### Bloat notes
- group.email_sends TOAST (3 GB) is the dominant storage consumer; row count is small so VACUUM FULL / pg_repack would reclaim little unless bodies are deleted; the real fix is not storing full HTML twice (rendered_html and body_html).
- cron.job_run_details: 155 MB for ~156k rows, never pruned since 2026-08-17.
- group.sync_runs: 128 MB / 178k rows of sync telemetry (3k rows per day); also a candidate for retention.
- public.research_* audit/fix/plan tables (research_fix_plan 41 MB, research_school_link_plan 62 MB, research_audit_cityref 19 MB, research_fix_web(_a) 12 MB each, research_place_fill_plan 10 MB) are one-off working tables kept after their batch ran; ~200 MB reclaimable if archived.
- 77 GB of temp files written since restart with work_mem = 5 MB indicates large sorts/hashes in the research pipeline (research_contrib_tick averages 36 s per run, research_library_tick 8.6 s every 2 minutes); raising work_mem for the cron role (ALTER ROLE postgres SET work_mem) or for those functions (SET work_mem in proconfig) would cut disk I/O.

### Tables never analysed since restart (>10 MB)
cron.job_run_details, leads.webhook_logs, group.core_applications, group.email_sends, group.notifications, group.sync_runs, group.tasks, group.wa_sends, group.webhook_logs, migration.id_map, public.applications, public.inbound_records, public.org_aliases, public.organizations, public.payments, public.people, public.person_identities, 31 public.research_* tables, quarantine.email_domain_evidence, quarantine.email_evidence, quarantine.org_candidates, quarantine.research_contacts, quarantine.research_schools, revenue_brain.yale_contacts. Mostly static tables; planner stats (reltuples) are present from before the restart, so this is informational.
