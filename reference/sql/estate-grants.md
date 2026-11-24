# Shared estate grants and roles

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only.

Sources: `pg_roles`, `has_schema_privilege()`, `information_schema.role_table_grants`.

## API-exposed schemas

`current_setting('pgrst.db_schemas', true)` and `pgrst.db_extra_search_path` both return NULL in this session, so the PostgREST exposed-schema list cannot be read from SQL here; it is set in the Supabase Dashboard (Project Settings -> API -> Exposed schemas). What the privilege data shows: the `authenticator` login role (which PostgREST connects as) has USAGE on only `public`, `leaders` and `net`, while `anon` / `authenticated` have USAGE on every platform schema except `affiliate`, `research`, `automation`, `legacy_compat`, `cron`, `vault`. Because PostgREST switches to anon/authenticated/service_role per request, it is the anon/authenticated/service_role USAGE that governs API access, together with the Dashboard exposed-schema list.

Practical reading: `affiliate` and `research` are **not** reachable by anon/authenticated through the API at all (no schema USAGE); their platforms work through SECURITY DEFINER functions in `public`/their schema called via RPC, or through service_role from edge functions.

## Schema USAGE by role

| schema | anon | authenticated | service_role | authenticator |
|---|---|---|---|---|
| affiliate | no | no | **yes** | no |
| automation | no | no | no | no |
| auth | yes | yes | yes | no |
| cron | no | no | no | no |
| premed | yes | yes | yes | no |
| extensions | yes | yes | yes | no |
| finance | yes | yes | yes | no |
| finedu | yes | yes | yes | no |
| ops | yes | yes | yes | no |
| prelaw | yes | yes | yes | no |
| legacy_compat | no | no | no | no |
| lms | yes | yes | yes | no |
| portal | yes | yes | yes | no |
| net | yes | yes | yes | yes |
| fellowship | yes | yes | yes | no |
| public | yes | yes | yes | yes |
| testprep | yes | yes | yes | no |
| research | no | no | **yes** | no |
| storage | yes | yes | yes | no |
| leaders | yes | yes | yes | yes |
| vault | no | no | yes | no |

No role has CREATE on any of these schemas (except the owner roles).

## Table-level privilege summary (information_schema.role_table_grants, tables + views)

Privilege letters: D=DELETE I=INSERT R=REFERENCES S=SELECT T=TRIGGER/TRUNCATE U=UPDATE. "objects" counts tables and views with at least one grant to that role.

| schema | grantee | objects with grants | privilege types |
|---|---|---|---|
| affiliate | service_role | 1 | all (DELETE,INSERT,REFERENCES,SELECT,TRIGGER,TRUNCATE,UPDATE) |
| premed | anon | 10 | SELECT |
| premed | authenticated | 47 | DELETE,INSERT,SELECT,UPDATE |
| premed | service_role | 50 | all |
| finance | anon | 3 | SELECT |
| finance | authenticated | 9 | SELECT |
| finance | service_role | 33 | all |
| finedu | anon | 9 | INSERT,SELECT |
| finedu | authenticated | 25 | DELETE,INSERT,SELECT,UPDATE |
| finedu | service_role | 30 | all |
| ops | authenticated | 2 | INSERT,SELECT,UPDATE |
| ops | service_role | 8 | all |
| prelaw | anon | 5 | SELECT |
| prelaw | authenticated | 11 | DELETE,INSERT,SELECT,UPDATE |
| prelaw | service_role | 13 | all |
| lms | anon | 5 | SELECT |
| lms | authenticated | 75 | DELETE,INSERT,SELECT,UPDATE |
| lms | service_role | 87 | all |
| portal | anon | 4 | INSERT |
| portal | authenticated | 18 | DELETE,INSERT,SELECT,UPDATE |
| portal | service_role | 26 | all |
| fellowship | anon | 1 | SELECT |
| fellowship | authenticated | 1 | SELECT |
| fellowship | service_role | 24 | all |
| public | anon | 20 | all (legacy default grants) |
| public | authenticated | 23 | all (legacy default grants) |
| public | service_role | 30 | all |
| testprep | anon | 13 | INSERT,SELECT |
| testprep | authenticated | 13 | INSERT,SELECT |
| testprep | service_role | 27 | all |
| research | anon | 1 | SELECT |
| research | authenticated | 2 | SELECT |
| research | service_role | 50 | all |
| leaders | service_role | 27 | all |

Notes:
- `affiliate` has grants only on one object for service_role (the `all_applications` view, presumably); the Affiliate app is RPC-only (`affiliate.*` SECURITY DEFINER functions, see functions file) with no direct table grants to anon/authenticated. Since anon/authenticated lack USAGE on `affiliate`, its functions must be invoked from `public`-schema wrappers or by service_role - check the edge functions.
- `leaders`: only service_role has table grants even though anon/authenticated have schema USAGE and the `authenticator` role too. The Student Leaders app therefore cannot touch its tables directly from the browser (it also has zero RLS policies).
- `public`: anon and authenticated still carry the Supabase default "ALL" grants on most legacy tables (DIRSTTU), which is why RLS-without-policy matters so much there.

## Per-object grants for anon and authenticated

Format `object [privs]` with D/I/S/U/R/T letters.

### premed
- anon: aeo_outputs [S], aeo_page_optimizations [S], aeo_pages [S], all_cohorts [S], all_programs [S], cohort_settings [S], concierge_faqs [S], masterclass_series_overrides [S], masterclass_sessions_public [S], program_cohorts [S]
- authenticated: aeo_keywords [DISU], aeo_outputs [DISU], aeo_page_optimizations [DISU], aeo_pages [DISU], aeo_prompts [DISU], aeo_settings [DISU], all_cohorts [S], all_masterclasses [S], all_mentors [S], all_partners [S], all_programs [S], app_settings [DISU], captured_leads [DISU], challenge_quiz_attempts [S], cohort_settings [DISU], concierge_ai_usage [S], concierge_conversations [ISU], concierge_faqs [DISU], premed_masterclass_regs [S], email_templates [DISU], fellowship_projects [DISU], goal_reviews [DISU], intake_raw [DISU], knowledge_entries [DISU], learning_assessments [DISU], live_class_rsvps [DISU], masterclass_registration_sessions [S], masterclass_series_overrides [S], masterclass_sessions [DIU], masterclass_sessions_public [S], mmi_coach_sessions [DISU], mmi_practice_attempts [DISU], parent_reports [DISU], program_attachments [DISU], program_cohorts [S], program_email_settings [DISU], programme_applications_read [S], saved_universities [DISU], sent_emails [DISU], session_attendance [DISU], side_effects_queue [DISU], student_goals [DISU], student_notifications [DISU], student_university_applications [DISU], universities [S], user_profiles [ISU], whatsapp_messages [DISU]

### finance
- anon: all_cohorts [S], all_partners [S], all_programs [S]
- authenticated: all_cohorts [S], all_masterclasses [S], all_mentors [S], all_partners [S], all_programs [S], codes_dropdown [S], expense_codes [S], program_codes [S], stripe_payments [S]

### finedu
- anon: all_cohorts [S], all_masterclasses [S], all_mentors [S], all_partners [S], all_programs [S], assessment_questions [S], concierge_leads [I], page_events [I], page_visits [I]
- authenticated: admin_masterclasses [S], ai_agent_logs [DISU], all_cohorts [S], all_masterclasses [S], all_mentors [S], all_partners [S], all_programs [S], app_settings [DISU], assessment_questions [DISU], concierge_leads [DISU], email_attachments [DISU], email_send_log [DISU], email_send_state [DISU], email_templates [DISU], email_unsubscribe_tokens [DISU], finedu_masterclass_regs [S], page_events [DISU], page_visits [DISU], payment_reminder_log [S], program_email_settings [DISU], school_proposals [DISU], suppressed_emails [DISU], user_roles [DISU], voucher_redemptions [IS], whatsapp_messages [DISU]

### ops
- anon: (none)
- authenticated: profiles [ISU], user_roles [S]

### prelaw
- anon: all_cohorts [S], all_masterclasses [S], all_mentors [S], all_partners [S], all_programs [S]
- authenticated: all_cohorts [S], all_masterclasses [S], all_mentors [S], all_partners [S], all_programs [S], email_logs [S], email_templates [DISU], unmatched_payments [S], user_roles [S], whatsapp_messages [S], whatsapp_settings [DISU]

### lms
- anon: all_cohorts [S], all_mentors [S], all_partners [S], all_programs [S], mentor_settings [S]
- authenticated: admin_profiles [DISU], all_cohorts [DISU], all_masterclasses [S], all_mentors [DISU], all_partners [DISU], all_programs [DISU], birthday_grants [DISU], certificate_templates [DISU], certificates [DISU], cohort_completions [DISU], cohort_group_members [DISU], cohort_group_settings [DISU], cohort_groups [DISU], cohort_pathways [DISU], cohort_section_members [DISU], cohort_sections [DISU], community_ai_reviews [DISU], community_channels [DISU], community_mentions [DISU], community_messages [DISU], community_moderation_log [DISU], community_reads [DISU], community_session_reminder_log [DISU], email_templates [DISU], enrollment_errors [DISU], enrollment_events [S], enrollment_run_items [DISU], enrollment_runs [DISU], enrollment_settings [S], enrollment_sync_items [S], enrollment_sync_runs [S], enrollments [DISU], feedback_nudge_log [DISU], grade_yearly_runs [S], help_messages [DISU], login_rollout_queue [S], login_rollout_settings [S], masterclass_registration_counts [DISU], masterclass_registrations [DISU], mentor_accounts [DISU], mentor_feedback [DISU], mentor_settings [DISU], notification_log [DISU], notification_recipients [DISU], notifications [DISU], partner_checkout_config [DISU], program_feedback [DISU], referral_settings [S], registration_attempts [S], resources [DISU], session_attendance [DISU], session_feedback [DISU], session_group_links [DISU], session_join_clicks [S], session_mentor_overrides [DISU], session_mentors [DISU], session_recording_watch [S], session_reminder_log [DISU], session_section_links [DISU], session_section_recordings [DISU], session_submissions [DISU], sessions [DISU], student_profiles [DISU], support_tickets [DISU], ticket_messages [DISU], user_roles [DISU], vault_fx_rates [DISU], vault_ledger [S], vault_lots [S], vault_review_flags [S], vault_settings [S], vault_settings_history [S], vault_walkthrough_seen [DISU], whatsapp_blocklist_archive [S], whatsapp_templates [DISU]

### portal
- anon: captured_leads [I], applications [I], newsletter_subscribers [I], school_program_inquiries [I]
- authenticated: app_settings [DISU], application_installments [DISU], camp_leads [DISU], captured_leads [DISU], codes_dropdown [S], email_reminder_settings [DISU], email_send_log [DISU], email_templates [DISU], form_templates [DISU], applications [DISU], newsletter_subscribers [DISU], payment_links [DISU], program_attachments [DISU], program_codes [S], program_email_settings [DISU], school_program_inquiries [DISU], user_profiles [S], whatsapp_drip_sequences [DISU]

### fellowship
- anon: all_cohorts [S]
- authenticated: all_cohorts [S]

### public
- anon: all_cohorts [RST], all_masterclasses_public [RST], all_partners [RT], all_programs [RST], codes_dropdown [DIRSTTU], finedu_masterclass_regs [DIRSTTU], prelaw_masterclass_regs [DIRSTTU], applications [I], form_responses [DIRSTTU], masterclass_regs [DIRSTTU], master_data [DIRSTTU], fellowship_masterclass_regs [DIRSTTU], platform_api_clients [DIRSTTU], program_codes [DIRSTTU], promo_codes [DIRSTTU], promo_redemptions [DIRSTTU], razorpay_payments [DIRSTTU], testprep_masterclass_regs [I], research_masterclass_regs [DIRSTTU], week_without_walls_inquiries [DIRSTTU]
- authenticated: all_cohorts [DIRSTTU], all_masterclasses [DIRSTTU], all_masterclasses_public [DIRSTTU], all_mentors [IRTU], all_partners [DIRTTU], all_programs [DIRSTTU], codes_dropdown [DIRSTTU], premed_masterclass_regs [DIRSTTU], finedu_masterclass_regs [DIRSTTU], prelaw_masterclass_regs [DIRSTTU], applications [DISU], form_responses [DIRSTTU], masterclass_regs [DIRSTTU], master_data [DIRSTTU], fellowship_masterclass_regs [DIRSTTU], platform_api_clients [DIRSTTU], program_codes [DIRSTTU], promo_codes [DIRSTTU], promo_redemptions [DIRSTTU], razorpay_payments [DIRSTTU], testprep_masterclass_regs [I], research_masterclass_regs [DIRSTTU], week_without_walls_inquiries [DIRSTTU]

Notable: `public.all_mentors` has no SELECT grant for anon (only R,T) and only IRTU for authenticated, yet policies grant reads - reads of mentors from browsers therefore go through the per-platform `*.all_mentors` views (which carry their own SELECT grants) or `lms.all_mentors`. `public.all_partners` likewise has no SELECT grant for anon; `public.all_masterclasses` has no anon grant at all (anon uses `all_masterclasses_public`).

### testprep
- anon and authenticated (identical): all_cohorts [S], all_masterclasses [S], all_mentors [S], all_partners [S], all_programs [S], country_dial_codes [S], lead_events [I], leads [I], program_cohorts [S], testprep_masterclass_regs [I], sat_questions [S], school_inquiries [I], session_analytics [I]

### research
- anon: all_mentors [S]
- authenticated: all_masterclasses [S], all_mentors [S]

(Grants exist but anon/authenticated lack schema USAGE on `research`, so these are unreachable through the API.)

### affiliate, leaders
- anon / authenticated: no table grants at all.

## Roles (pg_roles, excluding pg_*)

| role | login | superuser | bypass RLS | createrole/db | member of |
|---|---|---|---|---|---|
| anon | no | no | no | no | - |
| authenticated | no | no | no | no | - |
| authenticator | yes | no | no | no | anon, authenticated, service_role |
| service_role | no | no | **yes** | no | - |
| postgres | yes | no | **yes** | yes/yes | pg_monitor, pg_signal_backend, pg_read_all_data, pg_create_subscription, anon, authenticated, service_role, authenticator, supabase_functions_admin, supabase_privileged_role, fdw_reader, fellowship_writer |
| **fdw_reader** | yes | no | **yes** | no | - (custom: read-only cross-project FDW login; bypasses RLS) |
| **fellowship_writer** | yes | no | no | no | - (custom: Fellowship Platform site writes via its own DB login; has RLS policies on public.fellowship_masterclass_regs, public.week_without_walls_inquiries, public.all_mentors) |
| dashboard_user | no | no | no | yes/yes | - |
| supabase_admin | yes | **yes** | yes | yes/yes | - |
| supabase_auth_admin | yes | no | no | yes/no | - |
| supabase_etl_admin | yes | no | yes | no | pg_monitor, pg_read_all_data, supabase_privileged_role |
| supabase_functions_admin | yes | no | no | yes/no | - |
| supabase_privileged_role | no | no | no | no | - |
| supabase_read_only_user | yes | no | yes | no | pg_monitor, pg_read_all_data |
| supabase_realtime_admin | no | no | no | no | - |
| supabase_replication_admin | yes | no | no | no | - |
| supabase_storage_admin | yes | no | no | yes/no | authenticator |

Two custom login roles exist beyond the Supabase defaults: `fdw_reader` (BYPASSRLS - presumably the role the other Group Supabase projects use for postgres_fdw/dblink into this estate) and `fellowship_writer`.
