# Brain project: grants

Live snapshot taken 2026-11-24 from <DATA_REF>. Table/view privileges from information_schema.role_table_grants for anon / authenticated / service_role, compacted per schema. Letters: S=SELECT I=INSERT U=UPDATE D=DELETE T=TRUNCATE R=REFERENCES G=TRIGGER; "-" = no grant.

PostgREST exposed schemas: `current_setting('pgrst.db_schemas', true)` returns NULL at the SQL level (the setting lives on the authenticator role / PostgREST config, see the role config lines below). Schemas with no grants at all to anon/authenticated (hence invisible through the API regardless of RLS): attribution, staging, core_ro, legacy_ro, legacy_sat; quarantine and migration only have service_role grants.

Observations: group.* grants are very broad (most tables grant all 7 privileges to anon AND authenticated) and rely entirely on RLS for protection; a handful are deliberately narrower (core_applications/core_payments/core_pull_state: authenticated RSG only; agent_gmail_drafts, application_school_matches, member_* tables: S only; email_sends / deals / deal_events: no anon grant). leads.* grants DISU to anon and authenticated. public.* grants nothing to anon; authenticated mostly S (plus DISU on li_* and a few research_* write tables, ISU on research_field_changes); the spine PII tables (people, person_identities, households, payments, refunds, staff*) have NO authenticated grant and are read only through the *_v views (see people_v comment in views file).

## Schema ACLs, default ACLs, roles

- db_role_setting (all roles): app.settings.jwt_exp=3600
- db_role_setting anon: statement_timeout=3s
- db_role_setting authenticated: statement_timeout=8s
- db_role_setting authenticator: session_preload_libraries=supautils, safeupdate; statement_timeout=8s; lock_timeout=8s; pgrst.db_schemas=public, graphql_public, leads, group
- db_role_setting postgres: search_path="\$user", public, extensions
- db_role_setting supabase_admin: search_path="$user", public, auth, extensions; log_statement=none
- db_role_setting supabase_auth_admin: search_path=auth; idle_in_transaction_session_timeout=60000; log_statement=none
- db_role_setting supabase_read_only_user: default_transaction_read_only=on
- db_role_setting supabase_realtime_admin: search_path=public, extensions, realtime
- db_role_setting supabase_storage_admin: search_path=storage; log_statement=none
- default_acl auth / role supabase_auth_admin / objtype S: {postgres=rwU/supabase_auth_admin,dashboard_user=rwU/supabase_auth_admin}
- default_acl auth / role supabase_auth_admin / objtype f: {postgres=X/supabase_auth_admin,dashboard_user=X/supabase_auth_admin}
- default_acl auth / role supabase_auth_admin / objtype r: {postgres=arwdDxtm/supabase_auth_admin,dashboard_user=arwdDxtm/supabase_auth_admin}
- default_acl brain / role postgres / objtype r: {service_role=r/postgres}
- default_acl cron / role supabase_admin / objtype S: {postgres=r*w*U*/supabase_admin}
- default_acl cron / role supabase_admin / objtype f: {postgres=X*/supabase_admin}
- default_acl cron / role supabase_admin / objtype r: {postgres=a*r*w*d*D*x*t*m*/supabase_admin}
- default_acl extensions / role supabase_admin / objtype S: {postgres=r*w*U*/supabase_admin}
- default_acl extensions / role supabase_admin / objtype f: {postgres=X*/supabase_admin}
- default_acl extensions / role supabase_admin / objtype r: {postgres=a*r*w*d*D*x*t*m*/supabase_admin}
- default_acl graphql / role supabase_admin / objtype S: {postgres=rwU/supabase_admin,anon=rwU/supabase_admin,authenticated=rwU/supabase_admin,service_role=rwU/supabase_admin}
- default_acl graphql / role supabase_admin / objtype f: {postgres=X/supabase_admin,anon=X/supabase_admin,authenticated=X/supabase_admin,service_role=X/supabase_admin}
- default_acl graphql / role supabase_admin / objtype r: {postgres=arwdDxtm/supabase_admin,anon=arwdDxtm/supabase_admin,authenticated=arwdDxtm/supabase_admin,service_role=arwdDxtm/supabase_admin}
- default_acl graphql_public / role supabase_admin / objtype S: {postgres=rwU/supabase_admin,anon=rwU/supabase_admin,authenticated=rwU/supabase_admin,service_role=rwU/supabase_admin}
- default_acl graphql_public / role supabase_admin / objtype f: {postgres=X/supabase_admin,anon=X/supabase_admin,authenticated=X/supabase_admin,service_role=X/supabase_admin}
- default_acl graphql_public / role supabase_admin / objtype r: {postgres=arwdDxtm/supabase_admin,anon=arwdDxtm/supabase_admin,authenticated=arwdDxtm/supabase_admin,service_role=arwdDxtm/supabase_admin}
- default_acl leads / role postgres / objtype S: {anon=rU/postgres,authenticated=rU/postgres,service_role=rU/postgres}
- default_acl leads / role postgres / objtype f: {anon=X/postgres,authenticated=X/postgres,service_role=X/postgres}
- default_acl leads / role postgres / objtype r: {anon=arwd/postgres,authenticated=arwd/postgres,service_role=arwdDxt/postgres}
- default_acl public / role postgres / objtype S: {postgres=rwU/postgres,service_role=rwU/postgres}
- default_acl public / role postgres / objtype f: {postgres=X/postgres,service_role=X/postgres}
- default_acl public / role postgres / objtype r: {postgres=arwdDxtm/postgres,service_role=arwdDxtm/postgres}
- default_acl public / role supabase_admin / objtype S: {postgres=rwU/supabase_admin,anon=rwU/supabase_admin,authenticated=rwU/supabase_admin,service_role=rwU/supabase_admin}
- default_acl public / role supabase_admin / objtype f: {postgres=X/supabase_admin,anon=X/supabase_admin,authenticated=X/supabase_admin,service_role=X/supabase_admin}
- default_acl public / role supabase_admin / objtype r: {postgres=arwdDxtm/supabase_admin,anon=arwdDxtm/supabase_admin,authenticated=arwdDxtm/supabase_admin,service_role=arwdDxtm/supabase_admin}
- default_acl realtime / role supabase_admin / objtype S: {postgres=rwU/supabase_admin,dashboard_user=rwU/supabase_admin}
- default_acl realtime / role supabase_admin / objtype f: {postgres=X/supabase_admin,dashboard_user=X/supabase_admin}
- default_acl realtime / role supabase_admin / objtype r: {postgres=a*r*wdDxtm/supabase_admin,dashboard_user=arwdDxtm/supabase_admin}
- default_acl storage / role postgres / objtype S: {postgres=rwU/postgres,anon=rwU/postgres,authenticated=rwU/postgres,service_role=rwU/postgres}
- default_acl storage / role postgres / objtype f: {postgres=X/postgres,anon=X/postgres,authenticated=X/postgres,service_role=X/postgres}
- default_acl storage / role postgres / objtype r: {postgres=arwdDxtm/postgres,anon=arwdDxtm/postgres,authenticated=arwdDxtm/postgres,service_role=arwdDxtm/postgres}
- role anon: login=false super=false bypassrls=false createrole=false createdb=false inherit=true config=statement_timeout=3s
- role authenticated: login=false super=false bypassrls=false createrole=false createdb=false inherit=true config=statement_timeout=8s
- role authenticator: login=true super=false bypassrls=false createrole=false createdb=false inherit=false config=session_preload_libraries=supautils, safeupdate; statement_timeout=8s; lock_timeout=8s; pgrst.db_schemas=public, graphql_public, leads, group
- role dashboard_user: login=false super=false bypassrls=false createrole=true createdb=true inherit=true
- role lwl_puller: login=true super=false bypassrls=false createrole=false createdb=false inherit=false
- role postgres: login=true super=false bypassrls=true createrole=true createdb=true inherit=true config=search_path="\$user", public, extensions
- role service_role: login=false super=false bypassrls=true createrole=false createdb=false inherit=true
- role supabase_admin: login=true super=true bypassrls=true createrole=true createdb=true inherit=true config=search_path="$user", public, auth, extensions; log_statement=none
- role supabase_auth_admin: login=true super=false bypassrls=false createrole=true createdb=false inherit=false config=search_path=auth; idle_in_transaction_session_timeout=60000; log_statement=none
- role supabase_etl_admin: login=true super=false bypassrls=true createrole=false createdb=false inherit=true
- role supabase_functions_admin: login=true super=false bypassrls=false createrole=true createdb=false inherit=false
- role supabase_privileged_role: login=false super=false bypassrls=false createrole=false createdb=false inherit=true
- role supabase_read_only_user: login=true super=false bypassrls=true createrole=false createdb=false inherit=true config=default_transaction_read_only=on
- role supabase_realtime_admin: login=false super=false bypassrls=false createrole=false createdb=false inherit=false config=search_path=public, extensions, realtime
- role supabase_replication_admin: login=true super=false bypassrls=false createrole=false createdb=false inherit=true
- role supabase_storage_admin: login=true super=false bypassrls=false createrole=true createdb=false inherit=false config=search_path=storage; log_statement=none
- role_membership authenticator member of: anon, authenticated, service_role
- role_membership postgres member of: anon, authenticated, authenticator, lwl_puller, pg_create_subscription, pg_monitor, pg_read_all_data, pg_signal_backend, service_role, supabase_privileged_role
- role_membership supabase_etl_admin member of: pg_monitor, pg_read_all_data, supabase_privileged_role
- role_membership supabase_read_only_user member of: pg_monitor, pg_read_all_data
- role_membership supabase_realtime_admin member of: anon, authenticated, service_role
- role_membership supabase_storage_admin member of: authenticator
- schema_acl attribution: (none) | USAGE anon=false authenticated=false service_role=false
- schema_acl auth: {supabase_admin=UC/supabase_admin,anon=U/supabase_admin,authenticated=U/supabase_admin,service_role=U/supabase_admin,supabase_auth_admin=UC/supabase_admin,dashboard_user=UC/supabase_admin,postgres=U/supabase_admin} | USAGE anon=true authenticated=true service_role=true
- schema_acl brain: {postgres=UC/postgres,service_role=U/postgres,authenticated=U/postgres} | USAGE anon=false authenticated=true service_role=true
- schema_acl core_ro: {postgres=UC/postgres} | USAGE anon=false authenticated=false service_role=false
- schema_acl cron: {supabase_admin=UC/supabase_admin,postgres=U*/supabase_admin} | USAGE anon=false authenticated=false service_role=false
- schema_acl extensions: {postgres=UC/postgres,anon=U/postgres,authenticated=U/postgres,service_role=U/postgres,dashboard_user=UC/postgres} | USAGE anon=true authenticated=true service_role=true
- schema_acl graphql_public: {supabase_admin=UC/supabase_admin,postgres=U*/supabase_admin,anon=U/supabase_admin,authenticated=U/supabase_admin,service_role=U/supabase_admin} | USAGE anon=true authenticated=true service_role=true
- schema_acl leads: {postgres=UC/postgres,authenticated=U/postgres,service_role=UC/postgres,anon=U/postgres} | USAGE anon=true authenticated=true service_role=true
- schema_acl legacy_ro: {postgres=UC/postgres,service_role=U/postgres} | USAGE anon=false authenticated=false service_role=true
- schema_acl legacy_sat: (none) | USAGE anon=false authenticated=false service_role=false
- schema_acl group: {postgres=UC/postgres,anon=U/postgres,authenticated=U/postgres,service_role=U/postgres} | USAGE anon=true authenticated=true service_role=true
- schema_acl migration: {postgres=UC/postgres,service_role=U/postgres} | USAGE anon=false authenticated=false service_role=true
- schema_acl net: {supabase_admin=UC/supabase_admin,=U/supabase_admin,supabase_functions_admin=U/supabase_admin,postgres=U/supabase_admin,anon=U/supabase_admin,authenticated=U/supabase_admin,service_role=U/supabase_admin} | USAGE anon=true authenticated=true service_role=true
- schema_acl public: {pg_database_owner=UC/pg_database_owner,=U/pg_database_owner,postgres=U/pg_database_owner,anon=U/pg_database_owner,authenticated=U/pg_database_owner,service_role=U/pg_database_owner,lwl_puller=U/pg_database_owner} | USAGE anon=true authenticated=true service_role=true
- schema_acl quarantine: {postgres=UC/postgres,service_role=U/postgres} | USAGE anon=false authenticated=false service_role=true
- schema_acl revenue_brain: {postgres=UC/postgres} | USAGE anon=false authenticated=false service_role=false
- schema_acl staging: {postgres=UC/postgres,service_role=U/postgres} | USAGE anon=false authenticated=false service_role=true
- schema_acl storage: {supabase_admin=UC/supabase_admin,postgres=U*/supabase_admin,anon=U/supabase_admin,authenticated=U/supabase_admin,service_role=U/supabase_admin,supabase_storage_admin=U*C*/supabase_admin,dashboard_user=UC/supabase_admin} | USAGE anon=true authenticated=true service_role=true
- schema_acl vault: {supabase_admin=UC/supabase_admin,postgres=U*/supabase_admin,service_role=U/supabase_admin} | USAGE anon=false authenticated=false service_role=true

## Table grants: brain

| table/view | anon | authenticated | service_role |
|---|---|---|---|
| knowledge_assets | - | S | DISU |
| knowledge_base | - | S | DISU |
| knowledge_cadence | - | S | DISU |
| knowledge_faq | - | S | DISU |
| knowledge_intents | - | S | DISU |
| knowledge_org | - | S | DISU |
| knowledge_programs | - | S | DISU |
| knowledge_scenarios | - | S | DISU |
| knowledge_voice_rules | - | S | DISU |

## Table grants: leads

| table/view | anon | authenticated | service_role |
|---|---|---|---|
| agent_decisions | DISU | DISU | DIRSGTU |
| agent_jobs | DISU | DISU | DIRSGTU |
| agent_rate_limits | DISU | DISU | DIRSGTU |
| deal_events | - | DISU | DIRSGTU |
| deals | - | DISU | DIRSGTU |
| email_events | DISU | DISU | DIRSGTU |
| email_sends | DISU | DISU | DIRSGTU |
| email_suppression | DISU | DISU | DIRSGTU |
| email_templates | DISU | DISU | DIRSGTU |
| invitations | DISU | DISU | DIRSGTU |
| knowledge_assets | DISU | DISU | DIRSGTU |
| knowledge_base | DISU | DISU | DIRSGTU |
| knowledge_cadence | DISU | DISU | DIRSGTU |
| knowledge_faq | DISU | DISU | DIRSGTU |
| knowledge_intents | DISU | DISU | DIRSGTU |
| knowledge_org | DISU | DISU | DIRSGTU |
| knowledge_programs | DISU | DISU | DIRSGTU |
| knowledge_scenarios | DISU | DISU | DIRSGTU |
| knowledge_voice_rules | DISU | DISU | DIRSGTU |
| lead_ai_calls | DISU | DISU | DIRSGTU |
| applications | DISU | DISU | DIRSGTU |
| meeting_assignees | DISU | DISU | DIRSGTU |
| meeting_sync_queue | DISU | DISU | DIRSGTU |
| meetings | DISU | DISU | DIRSGTU |
| notifications | DISU | DISU | DIRSGTU |
| program_cohorts | DISU | DISU | DIRSGTU |
| scheduled_emails | DISU | DISU | DIRSGTU |
| school_aliases | DISU | DISU | DIRSGTU |
| school_merge_log | DISU | DISU | DIRSGTU |
| school_registry | DISU | DISU | DIRSGTU |
| sop_academic_calendar | - | DIRSGTU | DIRSGTU |
| sop_lists | - | DIRSGTU | DIRSGTU |
| sop_pricing_floors | - | DIRSGTU | DIRSGTU |
| sop_targets | - | DIRSGTU | DIRSGTU |
| sync_state | DISU | DISU | DIRSGTU |
| tasks | DISU | DISU | DIRSGTU |
| user_profiles | DISU | DISU | DIRSGTU |
| v_deal_stage_spans | DISU | DISU | DIRSGTU |
| webhook_logs | DISU | DISU | DIRSGTU |
| weekly_review_runs | DISU | DISU | DIRSGTU |

## Table grants: group

| table/view | anon | authenticated | service_role |
|---|---|---|---|
| _wa_resend_audit_20260918 | - | DIRSGTU | DIRSGTU |
| agent_decisions | DIRSGTU | DIRSGTU | DIRSGTU |
| agent_email_examples | - | - | DIRSGTU |
| agent_gmail_drafts | - | S | DIRSGTU |
| agent_jobs | DIRSGTU | DIRSGTU | DIRSGTU |
| agent_rate_limits | DIRSGTU | DIRSGTU | DIRSGTU |
| ai_settings | DIRSGTU | DIRSGTU | DIRSGTU |
| application_school_matches | - | S | DIRSGTU |
| attention_state | DIRSGTU | DIRSGTU | DIRSGTU |
| automations | DIRSGTU | DIRSGTU | DIRSGTU |
| bf_assignee | - | - | DIRSGTU |
| bf_insert | - | - | DIRSGTU |
| bf_update | - | - | DIRSGTU |
| calendly_question_registry | DIRSGTU | DIRSGTU | DIRSGTU |
| campaign_registry | - | DIRSGTU | DIRSGTU |
| core_applications | - | RSG | DIRSGTU |
| core_applications_removed_20261003 | DIRSGTU | DIRSGTU | DIRSGTU |
| core_bank_refunds | DIRSGTU | DIRSGTU | DIRSGTU |
| core_payments | - | RSG | DIRSGTU |
| core_pull_state | - | RSG | DIRSGTU |
| deal_events | - | DIRSGTU | DIRSGTU |
| deal_merge_log | DIRSGTU | DIRSGTU | DIRSGTU |
| deals | - | DIRSGTU | DIRSGTU |
| email_body_repair_attempts | - | - | DIRSGTU |
| email_events | DIRSGTU | DIRSGTU | DIRSGTU |
| email_sends | - | DIRSGTU | DIRSGTU |
| email_settings | DIRSGTU | DIRSGTU | DIRSGTU |
| email_suppression | DIRSGTU | DIRSGTU | DIRSGTU |
| email_templates | DIRSGTU | DIRSGTU | DIRSGTU |
| followup_plans | DIRSGTU | DIRSGTU | DIRSGTU |
| followup_report_runs | DIRSGTU | DIRSGTU | DIRSGTU |
| followup_report_settings | DIRSGTU | DIRSGTU | DIRSGTU |
| followup_touches | DIRSGTU | DIRSGTU | DIRSGTU |
| gmail_account_requests | DIRSGTU | DIRSGTU | DIRSGTU |
| institution_application_stats | - | DIRSGTU | DIRSGTU |
| invitations | DIRGTU | DIRSGTU | DIRSGTU |
| knowledge_assets | RSGT | DIRSGTU | DIRSGTU |
| knowledge_base | DIRSGTU | DIRSGTU | DIRSGTU |
| knowledge_cadence | RSGT | DIRSGTU | DIRSGTU |
| knowledge_faq | RSGT | DIRSGTU | DIRSGTU |
| knowledge_intents | RSGT | DIRSGTU | DIRSGTU |
| knowledge_org | RSGT | DIRSGTU | DIRSGTU |
| knowledge_programs | RSGT | DIRSGTU | DIRSGTU |
| knowledge_scenarios | RSGT | DIRSGTU | DIRSGTU |
| knowledge_voice_rules | RSGT | DIRSGTU | DIRSGTU |
| lead_ai_calls | DIRSGTU | DIRSGTU | DIRSGTU |
| applications | DIRSGTU | DIRSGTU | DIRSGTU |
| meeting_assignees | DIRSGTU | DIRSGTU | DIRSGTU |
| meeting_edit_log | DIRSGTU | DIRSGTU | DIRSGTU |
| meeting_followups | DIRSGTU | DIRSGTU | DIRSGTU |
| meeting_participants | DIRSGTU | DIRSGTU | DIRSGTU |
| meeting_programme_interest | DIRSGTU | DIRSGTU | DIRSGTU |
| meeting_report_runs | DIRSGTU | DIRSGTU | DIRSGTU |
| meeting_report_settings | DIRSGTU | DIRSGTU | DIRSGTU |
| meeting_sync_queue | DIRSGTU | DIRSGTU | DIRSGTU |
| meeting_welcome_jobs | DIRSGTU | DIRSGTU | DIRSGTU |
| meeting_zoom_links | DIRSGTU | DIRSGTU | DIRSGTU |
| meetings | DIRSGTU | DIRSGTU | DIRSGTU |
| meetings_sheet_sync_state | - | - | DIRSGTU |
| member_daily_email_runs | - | S | DIRSGTU |
| member_daily_email_settings | - | S | DIRSGTU |
| member_email_agent | - | S | DIRSGTU |
| member_report_days | - | - | DIRSGTU |
| member_sheet_reports | - | - | DIRSGTU |
| mv_meeting_outcomes | - | RSG | DIRSGTU |
| notifications | DIRSGTU | DIRSGTU | DIRSGTU |
| program_cohorts | DIRSGTU | DIRSGTU | DIRSGTU |
| review_commitments | DIRSGTU | DIRSGTU | DIRSGTU |
| scheduled_emails | DIRSGTU | DIRSGTU | DIRSGTU |
| school_aliases | DIRSGTU | DIRSGTU | DIRSGTU |
| school_domains | DIRSGTU | DIRSGTU | DIRSGTU |
| school_match_log | DIRSGTU | DIRSGTU | DIRSGTU |
| school_memory | DIRSGTU | DIRSGTU | DIRSGTU |
| school_merge_log | DIRSGTU | DIRSGTU | DIRSGTU |
| school_ownership | DIRSGTU | DIRSGTU | DIRSGTU |
| school_registry | DIRSGTU | DIRSGTU | DIRSGTU |
| school_research | DIRSGTU | DIRSGTU | DIRSGTU |
| sheet_import_stage | DIRSGTU | DIRSGTU | DIRSGTU |
| sop_academic_calendar | - | DIRSGTU | DIRSGTU |
| sop_lists | - | DIRSGTU | DIRSGTU |
| sop_pricing_floors | - | DIRSGTU | DIRSGTU |
| sop_targets | - | DIRSGTU | DIRSGTU |
| staging_utm_restore_20261021 | DIRSGTU | DIRSGTU | DIRSGTU |
| sync_runs | - | RSGT | DIRSGTU |
| sync_state | DIRSGTU | DIRSGTU | DIRSGTU |
| system_health | DIRSGTU | DIRSGTU | DIRSGTU |
| tasks | DIRSGTU | DIRSGTU | DIRSGTU |
| team_mailbox_import | DIRSGTU | DIRSGTU | DIRSGTU |
| user_gmail_accounts | DIRSGTU | DIRSGTU | DIRSGTU |
| user_profiles | DIRSGTU | DIRSGTU | DIRSGTU |
| user_profiles_safe | DIRSGTU | DIRSGTU | DIRSGTU |
| v_activity | - | DIRSGTU | DIRSGTU |
| v_activity_feed | DIRSGTU | DIRSGTU | DIRSGTU |
| v_claims_this_week | DIRSGTU | DIRSGTU | DIRSGTU |
| v_conversion_evidence | - | - | DIRSGTU |
| v_core_applications_unique | - | S | S |
| v_deal_stage_spans | DIRSGTU | DIRSGTU | DIRSGTU |
| v_deal_work | DIRSGTU | DIRSGTU | DIRSGTU |
| v_deal_worklist | DIRSGTU | DIRSGTU | DIRSGTU |
| v_lead_journey | - | DIRSGTU | DIRSGTU |
| v_lead_ledger | - | DIRSGTU | DIRSGTU |
| v_lead_source | DIRSGTU | DIRSGTU | DIRSGTU |
| v_meeting_assignee | DIRSGTU | DIRSGTU | DIRSGTU |
| v_meeting_attendance | - | DIRSGTU | DIRSGTU |
| v_meeting_clean | DIRSGTU | DIRSGTU | DIRSGTU |
| v_meeting_closeout | DIRSGTU | DIRSGTU | DIRSGTU |
| v_meeting_lead_type | - | DIRSGTU | DIRSGTU |
| v_meeting_ledger | - | DIRSGTU | DIRSGTU |
| v_meeting_member | - | S | DIRSGTU |
| v_meeting_outcomes | - | RSG | DIRSGTU |
| v_meeting_outcomes_base | - | S | DIRSGTU |
| v_meeting_revenue | - | S | DIRSGTU |
| v_meeting_roi | - | RSG | DIRSGTU |
| v_member_meeting | - | DIRSGTU | DIRSGTU |
| v_member_performance | - | DIRSGTU | DIRSGTU |
| v_payment_attribution | - | S | DIRSGTU |
| v_payment_meeting | - | S | DIRSGTU |
| v_school_ledger | - | DIRSGTU | DIRSGTU |
| v_school_owner | DIRSGTU | DIRSGTU | DIRSGTU |
| v_school_source | DIRSGTU | DIRSGTU | DIRSGTU |
| view_def_backup | DIRSGTU | DIRSGTU | DIRSGTU |
| wa_agent_runs | DIRSGTU | DIRSGTU | DIRSGTU |
| wa_claim_taps | DIRSGTU | DIRSGTU | DIRSGTU |
| wa_inbound | DIRSGTU | DIRSGTU | DIRSGTU |
| wa_limits | DIRSGTU | DIRSGTU | DIRSGTU |
| wa_poll_state | - | DIRSGTU | DIRSGTU |
| wa_sends | DIRSGTU | DIRSGTU | DIRSGTU |
| wa_session_window | DIRSGTU | DIRSGTU | DIRSGTU |
| wa_settings | DIRSGTU | DIRSGTU | DIRSGTU |
| wa_templates | DIRSGTU | DIRSGTU | DIRSGTU |
| webhook_logs | DIRSGTU | DIRSGTU | DIRSGTU |
| weekly_commitments | DIRSGTU | DIRSGTU | DIRSGTU |
| weekly_review_runs | DIRSGTU | DIRSGTU | DIRSGTU |

## Table grants: migration

| table/view | anon | authenticated | service_role |
|---|---|---|---|
| id_map | - | - | DIRSGTU |
| run_log | - | - | DIRSGTU |

## Table grants: public

| table/view | anon | authenticated | service_role |
|---|---|---|---|
| activities | - | S | DIRSGTU |
| activities_2026_08 | - | S | DIRSGTU |
| activities_2026_09 | - | S | DIRSGTU |
| activities_2026_10 | - | S | DIRSGTU |
| activities_default | - | S | DIRSGTU |
| affiliations | - | S | DIRSGTU |
| agent_runs | - | - | DIRSGTU |
| agent_setting_history | - | - | DIRSGTU |
| agent_settings | - | - | DIRSGTU |
| allocations | - | - | DIRSGTU |
| applications | - | - | DIRSGTU |
| approved_facts | - | S | DIRSGTU |
| autonomy_thresholds | - | S | DIRSGTU |
| bank_refunds | - | - | DIRSGTU |
| campaigns | - | S | DIRSGTU |
| cohorts | - | S | DIRSGTU |
| consents | - | S | DIRSGTU |
| frequency_counters | - | - | DIRSGTU |
| household_members | - | S | DIRSGTU |
| households | - | - | DIRSGTU |
| households_v | - | S | DIRSGTU |
| inbound_records | - | S | DIRSGTU |
| jobs | - | - | DIRSGTU |
| li_connections | - | DISU | DIRSGTU |
| li_content_posts | - | DISU | DIRSGTU |
| li_daily_runs | - | DISU | DIRSGTU |
| li_linkedin_connections | - | S | DIRSGTU |
| li_linkedin_oauth_states | - | - | DIRSGTU |
| li_research_items | - | DISU | DIRSGTU |
| li_work_updates | - | DISU | DIRSGTU |
| meetings | - | - | DIRSGTU |
| memberships | - | S | DIRSGTU |
| merge_lineage | - | S | DIRSGTU |
| messages | - | - | DIRSGTU |
| org_aliases | - | S | DIRSGTU |
| org_group_domains | - | - | DIRSGTU |
| org_groups | - | - | DIRSGTU |
| organizations | - | S | DIRSGTU |
| outreach_data_export | - | - | DIRSGTU |
| payments | - | - | DIRSGTU |
| people | - | - | DIRSGTU |
| people_v | - | S | DIRSGTU |
| person_identities | - | - | DIRSGTU |
| person_identities_v | - | S | DIRSGTU |
| programme_aliases | - | - | DIRSGTU |
| programme_codes | - | - | DIRSGTU |
| programmes | - | S | DIRSGTU |
| refunds | - | - | DIRSGTU |
| research_activity | - | S | DIRSGTU |
| research_agent_runs | - | S | DIRSGTU |
| research_agent_settings | - | S | DIRSGTU |
| research_ai_verified | - | S | DIRSGTU |
| research_allocations | - | S | DIRSGTU |
| research_alumni | - | - | DIRSGTU |
| research_applications | - | S | DIRSGTU |
| research_audit_blank_city | - | - | DIRSGTU |
| research_audit_city_x | - | - | DIRSGTU |
| research_audit_cityref | - | - | DIRSGTU |
| research_audit_cs | - | - | DIRSGTU |
| research_audit_csc | - | - | DIRSGTU |
| research_audit_school_city | - | - | DIRSGTU |
| research_autopilot | - | S | DIRSGTU |
| research_b2c_contacts | - | S | DIRSGTU |
| research_b2c_people | - | S | DIRSGTU |
| research_backfill_cursor | - | - | DIRSGTU |
| research_campaign_contacts | - | S | DIRSGTU |
| research_campaign_reports | - | DISU | DIRSGTU |
| research_campaigns | - | S | DIRSGTU |
| research_cell_flags | - | S | DIRSGTU |
| research_city_targets | - | - | DIRSGTU |
| research_claims | - | S | DIRSGTU |
| research_cleaning_runs | - | - | DIRSGTU |
| research_contact_facets | - | - | DIRSGTU |
| research_contact_health | - | - | DIRSGTU |
| research_contact_segments | - | S | DIRSGTU |
| research_contacts | - | S | DIRSGTU |
| research_contrib_cursor | - | - | DIRSGTU |
| research_data_health | - | S | DIRSGTU |
| research_datasets | - | S | DIRSGTU |
| research_derived_map | - | - | DIRSGTU |
| research_designation_proposals | - | S | DIRSGTU |
| research_district_directories | - | S | DIRSGTU |
| research_district_scans | - | - | DIRSGTU |
| research_domain_checks | - | S | DIRSGTU |
| research_domain_known | - | - | DIRSGTU |
| research_domain_names | - | - | DIRSGTU |
| research_domain_sites | - | S | DIRSGTU |
| research_duplicate_log | - | S | DIRSGTU |
| research_email_check_requests | - | - | DIRSGTU |
| research_email_check_runs | - | S | DIRSGTU |
| research_email_check_state | - | S | DIRSGTU |
| research_email_checks | - | S | DIRSGTU |
| research_email_health | - | S | DIRSGTU |
| research_email_pending | - | - | DIRSGTU |
| research_email_queue | - | - | DIRSGTU |
| research_email_targets | - | - | DIRSGTU |
| research_email_templates | - | DISU | DIRSGTU |
| research_events | - | S | DIRSGTU |
| research_exports | - | - | DIRSGTU |
| research_fact_fill_cols | - | - | DIRSGTU |
| research_fact_fill_plan | - | - | DIRSGTU |
| research_fact_fill_schools | - | - | DIRSGTU |
| research_field_changes | - | ISU | DIRSGTU |
| research_field_defs | - | S | DIRSGTU |
| research_fix_cityst | - | - | DIRSGTU |
| research_fix_g22 | - | - | DIRSGTU |
| research_fix_g45 | - | - | DIRSGTU |
| research_fix_g45src | - | - | DIRSGTU |
| research_fix_g6 | - | - | DIRSGTU |
| research_fix_plan | - | - | DIRSGTU |
| research_fix_r1 | - | - | DIRSGTU |
| research_fix_r6 | - | - | DIRSGTU |
| research_fix_typo | - | - | DIRSGTU |
| research_fix_web | - | - | DIRSGTU |
| research_fix_web_a | - | - | DIRSGTU |
| research_folder_members | - | - | DIRSGTU |
| research_folders | - | - | DIRSGTU |
| research_found_people | - | S | DIRSGTU |
| research_generated_docs | - | S | DIRSGTU |
| research_groups | - | S | DIRSGTU |
| research_identity_matches | - | S | DIRSGTU |
| research_intel_chats | - | S | DIRSGTU |
| research_intel_messages | - | S | DIRSGTU |
| research_job_locks | - | S | DIRSGTU |
| research_jobs | - | S | DIRSGTU |
| research_leads | - | S | DIRSGTU |
| research_legacy_source_map | - | - | DIRSGTU |
| research_library_cursor | - | - | DIRSGTU |
| research_locations | - | S | DIRSGTU |
| research_optouts | - | S | DIRSGTU |
| research_outreach_claims | - | DISU | DIRSGTU |
| research_pages | - | S | DIRSGTU |
| research_people | - | S | DIRSGTU |
| research_person_emails | - | S | DIRSGTU |
| research_person_events | - | S | DIRSGTU |
| research_person_links | - | S | DIRSGTU |
| research_person_phones | - | S | DIRSGTU |
| research_person_schools | - | S | DIRSGTU |
| research_pipeline_links | - | S | DIRSGTU |
| research_place_fill_plan | - | - | DIRSGTU |
| research_proposals | - | S | DIRSGTU |
| research_raw_rows | - | S | DIRSGTU |
| research_reach_cache | - | - | DIRSGTU |
| research_record_actions | - | DISU | DIRSGTU |
| research_ref_cities | - | - | DIRSGTU |
| research_ref_countries | - | - | DIRSGTU |
| research_ref_states | - | - | DIRSGTU |
| research_registry_dry | - | - | DIRSGTU |
| research_registry_schools | - | S | DIRSGTU |
| research_resolve_items | - | S | DIRSGTU |
| research_resolve_runs | - | S | DIRSGTU |
| research_review_domain_school | - | - | DIRSGTU |
| research_review_outcomes | - | S | DIRSGTU |
| research_review_rows | - | S | DIRSGTU |
| research_saved_views | - | S | DIRSGTU |
| research_schema_export_test | - | - | DIRSGTU |
| research_school_facts | - | S | DIRSGTU |
| research_school_findings | - | S | DIRSGTU |
| research_school_intel | - | S | DIRSGTU |
| research_school_link_plan | - | - | DIRSGTU |
| research_school_links | - | - | DIRSGTU |
| research_school_notes | - | S | DIRSGTU |
| research_school_profiles | - | S | DIRSGTU |
| research_school_web | - | S | DIRSGTU |
| research_schools | - | S | DIRSGTU |
| research_segments | - | S | DIRSGTU |
| research_send_exports | - | S | DIRSGTU |
| research_settings | - | S | DIRSGTU |
| research_short_links | - | S | DIRSGTU |
| research_sop_acknowledgements | - | S | DIRSGTU |
| research_sop_versions | - | S | DIRSGTU |
| research_source_tools | - | - | DIRSGTU |
| research_spend | - | S | DIRSGTU |
| research_staff_name_keys | - | - | DIRSGTU |
| research_staff_profiles | - | S | DIRSGTU |
| research_stats_contributions | - | - | DIRSGTU |
| research_stats_contributions_meta | - | - | DIRSGTU |
| research_stats_email_checks | - | - | DIRSGTU |
| research_tasks | - | S | DIRSGTU |
| research_territory_briefs | - | S | DIRSGTU |
| research_tracking_events | - | S | DIRSGTU |
| research_tracking_tokens | - | S | DIRSGTU |
| research_trust_totals_cache | - | - | DIRSGTU |
| research_upload_tabs | - | S | DIRSGTU |
| research_uploads | - | S | DIRSGTU |
| research_us_states | - | S | DIRSGTU |
| research_view_snapshots | - | S | DIRSGTU |
| schema_migrations | - | - | DIRSGTU |
| send_attempts | - | S | DIRSGTU |
| sensitive_access_log | - | - | DIRSGTU |
| sequence_steps | - | S | DIRSGTU |
| sequences | - | S | DIRSGTU |
| source_codes | - | S | DIRSGTU |
| source_systems | - | S | DIRSGTU |
| staff | - | - | DIRSGTU |
| staff_grants | - | - | DIRSGTU |
| staff_invitations | - | - | DIRSGTU |
| staff_role_assignments | - | - | DIRSGTU |
| staff_scopes | - | - | DIRSGTU |
| staff_targets | - | - | DIRSGTU |
| suppressions | - | S | DIRSGTU |
| tasks | - | - | DIRSGTU |
| v_puller_applications | - | - | DIRSGTU |
| v_puller_bank_refunds | - | - | DIRSGTU |
| v_puller_payments | - | - | DIRSGTU |

## Table grants: quarantine

| table/view | anon | authenticated | service_role |
|---|---|---|---|
| org_candidates | - | - | DIRSGTU |
| research_contacts | - | - | DIRSGTU |

