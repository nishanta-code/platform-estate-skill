# Brain project: RLS policies

Live snapshot taken 2026-11-24 from <DATA_REF>. Every row of pg_policies, grouped by schema. Columns: table | policy name | command | roles (prefixed RESTRICTIVE when not permissive) | USING expression | WITH CHECK expression.

Totals: group 200, leads 189, public 119, storage 12, brain 9, cron 2 (531). Schemas with RLS enabled but **zero policies**: attribution, quarantine, revenue_brain, migration (id_map, run_log), and ~70 public.* tables (deny-all for anon/authenticated).

Patterns worth knowing: group.* policies wrap helper calls in scalar subqueries (`( SELECT group.is_admin_or_owner(( SELECT auth.uid() )) )`) so they are evaluated once per statement; leads.* uses the older per-row form `leads.is_admin_or_owner(auth.uid())` and carries duplicated policy pairs (e.g. admin_delete + email_templates_admin_delete). public.* spine tables use has_scope_all() / can_see_person() / has_grant('comms_read') / auth_staff_id(); research_* tables use research_role(auth.uid()) IS NOT NULL. group.bf_* backfill tables have explicit deny-all (false/false) policies for anon+authenticated. Service-role-only tables: group.meetings_sheet_sync_state, member_report_days, member_sheet_reports, lead_ai_calls (service_role ALL true).

## Schema: brain (9 policies)

| table | policy | cmd | roles | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|
| knowledge_assets | read_knowledge_assets | SELECT | authenticated | (( SELECT auth.uid() AS uid) IS NOT NULL) |  |
| knowledge_base | read_knowledge_base | SELECT | authenticated | (( SELECT auth.uid() AS uid) IS NOT NULL) |  |
| knowledge_cadence | read_knowledge_cadence | SELECT | authenticated | (( SELECT auth.uid() AS uid) IS NOT NULL) |  |
| knowledge_faq | read_knowledge_faq | SELECT | authenticated | (( SELECT auth.uid() AS uid) IS NOT NULL) |  |
| knowledge_intents | read_knowledge_intents | SELECT | authenticated | (( SELECT auth.uid() AS uid) IS NOT NULL) |  |
| knowledge_org | read_knowledge_org | SELECT | authenticated | (( SELECT auth.uid() AS uid) IS NOT NULL) |  |
| knowledge_programs | read_knowledge_programs | SELECT | authenticated | (( SELECT auth.uid() AS uid) IS NOT NULL) |  |
| knowledge_scenarios | read_knowledge_scenarios | SELECT | authenticated | (( SELECT auth.uid() AS uid) IS NOT NULL) |  |
| knowledge_voice_rules | read_knowledge_voice_rules | SELECT | authenticated | (( SELECT auth.uid() AS uid) IS NOT NULL) |  |

## Schema: cron (2 policies)

| table | policy | cmd | roles | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|
| job | cron_job_policy | ALL | public | (username = CURRENT_USER) |  |
| job_run_details | cron_job_run_details_policy | ALL | public | (username = CURRENT_USER) |  |

## Schema: leads (189 policies)

| table | policy | cmd | roles | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|
| agent_decisions | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| agent_decisions | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| agent_decisions | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| agent_decisions | agent_decisions_admin_all | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| agent_decisions | agent_decisions_member_assigned | SELECT | authenticated | ((meeting_id IS NOT NULL) AND leads.is_assigned_to_meeting(meeting_id, auth.uid())) |  |
| agent_jobs | admin_read | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| agent_jobs | agent_jobs_admin_read | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| agent_jobs | agent_jobs_admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| agent_rate_limits | admin_read | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| agent_rate_limits | admins read rate limits | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| deal_events | deal_events_read | SELECT | authenticated | (leads.sop_actor() IS NOT NULL) |  |
| deal_events | deal_events_write | INSERT | authenticated |  | (leads.sop_actor() IS NOT NULL) |
| deals | deals_admin_all | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| deals | deals_member_insert | INSERT | authenticated |  | (leads.sop_actor() IS NOT NULL) |
| deals | deals_member_update | UPDATE | authenticated | ((leads.sop_actor() IS NOT NULL) AND ((owner = auth.uid()) OR (owner IS NULL))) | ((leads.sop_actor() IS NOT NULL) AND ((owner = auth.uid()) OR (owner IS NULL))) |
| deals | deals_read | SELECT | authenticated | (leads.sop_actor() IS NOT NULL) |  |
| email_events | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| email_events | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| email_events | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| email_events | ee_admin_write | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| email_events | ee_select | SELECT | authenticated | (leads.is_admin_or_owner(auth.uid()) OR (EXISTS ( SELECT 1    FROM leads.email_sends es   WHERE ((es.id = email_events.email_send_id) AND (es.meeting_id IS NOT NULL) AND leads.is_assigned_to_meeting(es.meeting_id, auth.uid()))))) |  |
| email_sends | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| email_sends | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| email_sends | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| email_sends | email_sends_admin_all | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| email_sends | email_sends_member_assigned | SELECT | authenticated | ((meeting_id IS NOT NULL) AND leads.is_assigned_to_meeting(meeting_id, auth.uid())) |  |
| email_suppression | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| email_suppression | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| email_suppression | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| email_suppression | supp_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| email_suppression | supp_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| email_suppression | supp_select | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| email_suppression | supp_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| email_templates | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| email_templates | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| email_templates | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| email_templates | email_templates_admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| email_templates | email_templates_admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| email_templates | email_templates_admin_write | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| email_templates | email_templates_select_authenticated | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| invitations | admin_all | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| invitations | admin_manage_invitations | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_assets | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_assets | admin_delete_knowledge_assets | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_assets | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_assets | admin_insert_knowledge_assets | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_assets | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_assets | admin_update_knowledge_assets | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_assets | auth_read_knowledge_assets | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| knowledge_base | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_base | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_base | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_base | knowledge_base read for signed-in | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| knowledge_base | knowledge_base_admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_base | knowledge_base_admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_base | knowledge_base_admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_cadence | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_cadence | admin_delete_knowledge_cadence | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_cadence | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_cadence | admin_insert_knowledge_cadence | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_cadence | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_cadence | admin_update_knowledge_cadence | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_cadence | auth_read_knowledge_cadence | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| knowledge_faq | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_faq | admin_delete_knowledge_faq | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_faq | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_faq | admin_insert_knowledge_faq | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_faq | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_faq | admin_update_knowledge_faq | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_faq | auth_read_knowledge_faq | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| knowledge_intents | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_intents | admin_delete_knowledge_intents | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_intents | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_intents | admin_insert_knowledge_intents | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_intents | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_intents | admin_update_knowledge_intents | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_intents | auth_read_knowledge_intents | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| knowledge_org | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_org | admin_delete_knowledge_org | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_org | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_org | admin_insert_knowledge_org | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_org | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_org | admin_update_knowledge_org | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_org | auth_read_knowledge_org | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| knowledge_programs | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_programs | admin_delete_knowledge_programs | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_programs | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_programs | admin_insert_knowledge_programs | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_programs | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_programs | admin_update_knowledge_programs | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_programs | auth_read_knowledge_programs | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| knowledge_scenarios | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_scenarios | admin_delete_knowledge_scenarios | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_scenarios | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_scenarios | admin_insert_knowledge_scenarios | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_scenarios | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_scenarios | admin_update_knowledge_scenarios | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_scenarios | auth_read_knowledge_scenarios | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| knowledge_voice_rules | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_voice_rules | admin_delete_knowledge_voice_rules | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| knowledge_voice_rules | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_voice_rules | admin_insert_knowledge_voice_rules | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| knowledge_voice_rules | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_voice_rules | admin_update_knowledge_voice_rules | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| knowledge_voice_rules | auth_read_knowledge_voice_rules | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| lead_ai_calls | Authenticated can read ai calls | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| lead_ai_calls | Service role manages ai calls | ALL | service_role | true | true |
| lead_ai_calls | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| lead_ai_calls | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| lead_ai_calls | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| applications | Admins write applications | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| applications | Members read applications | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| applications | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| applications | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| applications | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| meeting_assignees | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| meeting_assignees | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| meeting_assignees | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| meeting_assignees | meeting_assignees_admin_write | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| meeting_assignees | meeting_assignees_read | SELECT | authenticated | (leads.is_admin_or_owner(auth.uid()) OR (user_id = auth.uid()) OR leads.is_assigned_to_meeting(meeting_id, auth.uid())) |  |
| meeting_sync_queue | Admins read sync queue | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| meeting_sync_queue | admin_read | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| meetings | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| meetings | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| meetings | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| meetings | assignee_update_meeting | UPDATE | authenticated | leads.is_assigned_to_meeting(id, auth.uid()) | leads.is_assigned_to_meeting(id, auth.uid()) |
| meetings | meetings_admin_all | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| meetings | meetings_member_assigned_select | SELECT | authenticated | leads.is_assigned_to_meeting(id, auth.uid()) |  |
| meetings | meetings_member_assigned_update | UPDATE | authenticated | leads.is_assigned_to_meeting(id, auth.uid()) | leads.is_assigned_to_meeting(id, auth.uid()) |
| notifications | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| notifications | notifications_admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| notifications | notifications_own | ALL | authenticated | (user_id = auth.uid()) | (user_id = auth.uid()) |
| notifications | own_delete | DELETE | authenticated | (user_id = auth.uid()) |  |
| notifications | own_read | SELECT | authenticated | (user_id = auth.uid()) |  |
| notifications | own_update | UPDATE | authenticated | (user_id = auth.uid()) | (user_id = auth.uid()) |
| program_cohorts | Admins manage program_cohorts | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| program_cohorts | Staff can read program_cohorts | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| program_cohorts | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| program_cohorts | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| program_cohorts | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| scheduled_emails | Admins and owners can manage scheduled emails | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| scheduled_emails | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| scheduled_emails | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| scheduled_emails | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| scheduled_emails | scheduled_emails_select_scoped | SELECT | authenticated | (leads.is_admin_or_owner(auth.uid()) OR leads.is_assigned_to_meeting(meeting_id, auth.uid())) |  |
| school_aliases | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| school_aliases | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| school_aliases | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| school_aliases | aliases admin delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| school_aliases | aliases admin insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| school_aliases | aliases admin update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| school_aliases | aliases read all authed | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| school_merge_log | admin_read | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| school_merge_log | admins read merge log | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| school_registry | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| school_registry | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| school_registry | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| school_registry | school_registry admin delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| school_registry | school_registry admin update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| school_registry | school_registry admin write | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| school_registry | school_registry read all authed | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| sop_academic_calendar | calendar_admin | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| sop_academic_calendar | calendar_read | SELECT | authenticated | (leads.sop_actor() IS NOT NULL) |  |
| sop_lists | sop_lists_admin | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| sop_lists | sop_lists_read | SELECT | authenticated | (leads.sop_actor() IS NOT NULL) |  |
| sop_pricing_floors | floors_admin | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| sop_pricing_floors | floors_read | SELECT | authenticated | (leads.sop_actor() IS NOT NULL) |  |
| sop_targets | targets_admin | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| sop_targets | targets_read | SELECT | authenticated | (leads.sop_actor() IS NOT NULL) |  |
| sync_state | Admins write sync_state | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| sync_state | Members read sync_state | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| sync_state | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| sync_state | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| sync_state | admin_update | UPDATE | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| tasks | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| tasks | assignee_or_admin_update | UPDATE | authenticated | ((assigned_to = auth.uid()) OR leads.is_admin_or_owner(auth.uid())) | ((assigned_to = auth.uid()) OR leads.is_admin_or_owner(auth.uid())) |
| tasks | tasks_admins_full | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| tasks | tasks_members_update_own | UPDATE | authenticated | ((assigned_to = auth.uid()) OR ((meeting_id IS NOT NULL) AND leads.is_assigned_to_meeting(meeting_id, auth.uid()))) | ((assigned_to = auth.uid()) OR ((meeting_id IS NOT NULL) AND leads.is_assigned_to_meeting(meeting_id, auth.uid()))) |
| tasks | tasks_members_view | SELECT | authenticated | ((assigned_to = auth.uid()) OR ((meeting_id IS NOT NULL) AND leads.is_assigned_to_meeting(meeting_id, auth.uid()))) |  |
| user_profiles | admin_delete | DELETE | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| user_profiles | admin_insert | INSERT | authenticated |  | leads.is_admin_or_owner(auth.uid()) |
| user_profiles | admin_manage_profiles | ALL | authenticated | leads.is_admin_or_owner(auth.uid()) | leads.is_admin_or_owner(auth.uid()) |
| user_profiles | profiles_self_or_admin_read | SELECT | authenticated | ((auth.uid() = id) OR leads.is_admin_or_owner(auth.uid())) |  |
| user_profiles | self_update | UPDATE | authenticated | ((id = auth.uid()) OR leads.is_admin_or_owner(auth.uid())) | ((id = auth.uid()) OR leads.is_admin_or_owner(auth.uid())) |
| user_profiles | self_update_profile | UPDATE | authenticated | (auth.uid() = id) | ((auth.uid() = id) AND (role = ( SELECT user_profiles_1.role    FROM leads.user_profiles user_profiles_1   WHERE (user_profiles_1.id = auth.uid()))) AND (is_active = ( SELECT user_profiles_1.is_active    FROM leads.user_profiles user_profiles_1   WHERE (user_profiles_1.id = auth.uid())))) |
| webhook_logs | Admins read webhook logs | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| webhook_logs | admin_read | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| weekly_review_runs | admin_read | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |
| weekly_review_runs | admins read weekly reviews | SELECT | authenticated | leads.is_admin_or_owner(auth.uid()) |  |

## Schema: group (200 policies)

| table | policy | cmd | roles | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|
| _wa_resend_audit_20260918 | admin_read_wa_resend_audit | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| agent_decisions | agent_decisions_admin_all | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| agent_decisions | agent_decisions_member_assigned | SELECT | authenticated | ((meeting_id IS NOT NULL) AND group.is_assigned_to_meeting(meeting_id, ( SELECT auth.uid() AS uid))) |  |
| agent_gmail_drafts | Staff see agent drafts | SELECT | authenticated | (( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) OR (member_id = ( SELECT auth.uid() AS uid))) |  |
| agent_jobs | agent_jobs_admin_read | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| agent_jobs | agent_jobs_admin_update | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| agent_rate_limits | admins read rate limits | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| ai_settings | ai settings readable by staff | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| ai_settings | ai settings written by owners | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| application_school_matches | Owners and admins read school matches | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| attention_state | staff delete attention state | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| attention_state | staff insert attention state | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| attention_state | staff read attention state | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| attention_state | staff update attention state | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| automations | au_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| automations | au_update | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| bf_assignee | bf_assignee_no_client_access | ALL | anon,authenticated | false | false |
| bf_insert | bf_insert_no_client_access | ALL | anon,authenticated | false | false |
| bf_update | bf_update_no_client_access | ALL | anon,authenticated | false | false |
| calendly_question_registry | admins map calendly questions | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| calendly_question_registry | staff read calendly questions | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| campaign_registry | campaign_registry_admin_write | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| campaign_registry | campaign_registry_staff_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| core_applications | core_apps_staff_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| core_bank_refunds | core_bank_refunds_admin_read | SELECT | public | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| core_payments | core_pays_admin_read | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| core_pull_state | core_state_staff_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| deal_events | deal_events_read | SELECT | authenticated | (group.sop_actor() IS NOT NULL) |  |
| deal_events | deal_events_write | INSERT | authenticated |  | ((group.sop_actor() IS NOT NULL) AND (actor = ( SELECT auth.uid() AS uid))) |
| deal_merge_log | merge log readable by staff | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| deals | deals_admin_all | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| deals | deals_member_insert | INSERT | authenticated |  | ((group.sop_actor() IS NOT NULL) AND ((owner IS NULL) OR (owner = ( SELECT auth.uid() AS uid)))) |
| deals | deals_member_update | UPDATE | authenticated | ((group.sop_actor() IS NOT NULL) AND ((owner = ( SELECT auth.uid() AS uid)) OR (owner IS NULL))) | ((group.sop_actor() IS NOT NULL) AND (owner = ( SELECT auth.uid() AS uid))) |
| deals | deals_read | SELECT | authenticated | (( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) OR ((group.sop_actor() IS NOT NULL) AND ((owner = ( SELECT auth.uid() AS uid)) OR (next_action_owner = ( SELECT auth.uid() AS uid))))) |  |
| email_events | ee_admin_write | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| email_events | ee_select | SELECT | authenticated | (( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) OR (EXISTS ( SELECT 1    FROM group.email_sends es   WHERE ((es.id = email_events.email_send_id) AND (es.meeting_id IS NOT NULL) AND group.is_assigned_to_meeting(es.meeting_id, ( SELECT auth.uid() AS uid)))))) |  |
| email_sends | email_sends_admin_all | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| email_sends | email_sends_member_assigned | SELECT | authenticated | ((meeting_id IS NOT NULL) AND group.is_assigned_to_meeting(meeting_id, ( SELECT auth.uid() AS uid))) |  |
| email_settings | email settings readable by staff | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| email_settings | email settings written by owners | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| email_suppression | supp_delete | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| email_suppression | supp_insert | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| email_suppression | supp_select | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| email_suppression | supp_update | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| email_templates | email_templates_admin_delete | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| email_templates | email_templates_admin_update | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| email_templates | email_templates_admin_write | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| email_templates | email_templates_staff_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| followup_plans | staff read followup plans | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| followup_plans | staff update followup plans | UPDATE | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |
| followup_plans | staff write followup plans | INSERT | authenticated |  | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |
| followup_report_runs | Admins read follow-up report runs | SELECT | authenticated | ( SELECT group.is_admin_or_owner(auth.uid()) AS is_admin_or_owner) |  |
| followup_report_settings | Admins read follow-up report settings | SELECT | authenticated | ( SELECT group.is_admin_or_owner(auth.uid()) AS is_admin_or_owner) |  |
| followup_report_settings | Admins update follow-up report settings | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(auth.uid()) AS is_admin_or_owner) |  |
| followup_touches | staff delete followup touches | DELETE | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| followup_touches | staff insert followup touches | INSERT | authenticated |  | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |
| followup_touches | staff read followup touches | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| followup_touches | staff update followup touches | UPDATE | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |
| gmail_account_requests | gmail_requests_read | SELECT | authenticated | ((user_id = ( SELECT auth.uid() AS uid)) OR ( SELECT group.is_admin_or_owner(auth.uid()) AS is_admin_or_owner)) |  |
| invitations | admin_manage_invitations | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_assets | admin_delete_knowledge_assets | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| knowledge_assets | admin_insert_knowledge_assets | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_assets | admin_update_knowledge_assets | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_assets | staff_read_knowledge_assets | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| knowledge_base | knowledge_base_admin_delete | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| knowledge_base | knowledge_base_admin_insert | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_base | knowledge_base_admin_update | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_base | staff_read_knowledge_base | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| knowledge_cadence | admin_delete_knowledge_cadence | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| knowledge_cadence | admin_insert_knowledge_cadence | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_cadence | admin_update_knowledge_cadence | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_cadence | staff_read_knowledge_cadence | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| knowledge_faq | admin_delete_knowledge_faq | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| knowledge_faq | admin_insert_knowledge_faq | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_faq | admin_update_knowledge_faq | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_faq | staff_read_knowledge_faq | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| knowledge_intents | admin_delete_knowledge_intents | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| knowledge_intents | admin_insert_knowledge_intents | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_intents | admin_update_knowledge_intents | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_intents | staff_read_knowledge_intents | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| knowledge_org | admin_delete_knowledge_org | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| knowledge_org | admin_insert_knowledge_org | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_org | admin_update_knowledge_org | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_org | staff_read_knowledge_org | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| knowledge_programs | admin_delete_knowledge_programs | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| knowledge_programs | admin_insert_knowledge_programs | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_programs | admin_update_knowledge_programs | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_programs | staff_read_knowledge_programs | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| knowledge_scenarios | admin_delete_knowledge_scenarios | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| knowledge_scenarios | admin_insert_knowledge_scenarios | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_scenarios | admin_update_knowledge_scenarios | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_scenarios | staff_read_knowledge_scenarios | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| knowledge_voice_rules | admin_delete_knowledge_voice_rules | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| knowledge_voice_rules | admin_insert_knowledge_voice_rules | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_voice_rules | admin_update_knowledge_voice_rules | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| knowledge_voice_rules | staff_read_knowledge_voice_rules | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| lead_ai_calls | Service role manages ai calls | ALL | service_role | true | true |
| lead_ai_calls | lead_ai_calls_staff_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| applications | Admins write applications | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| applications | applications_admin_read | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| meeting_assignees | meeting_assignees_admin_write | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| meeting_assignees | meeting_assignees_coassign | INSERT | authenticated |  | (( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) AND group.is_active_staff(user_id) AND group.is_assigned_to_meeting(meeting_id, ( SELECT auth.uid() AS uid))) |
| meeting_assignees | meeting_assignees_self_claim | INSERT | authenticated |  | ((user_id = ( SELECT auth.uid() AS uid)) AND ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) AND (NOT group.meeting_has_assignee(meeting_id))) |
| meeting_assignees | meeting_assignees_self_release | DELETE | authenticated | (user_id = ( SELECT auth.uid() AS uid)) |  |
| meeting_assignees | meeting_assignees_staff_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| meeting_edit_log | meeting_edit_log_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| meeting_followups | followups read | SELECT | authenticated | (( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) OR (user_id = ( SELECT auth.uid() AS uid)) OR (meeting_id IN ( SELECT group.my_assigned_meeting_ids() AS my_assigned_meeting_ids))) |  |
| meeting_participants | participants readable by staff | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| meeting_programme_interest | mpi_delete | DELETE | authenticated | (( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) OR group.is_assigned_to_meeting(meeting_id, ( SELECT auth.uid() AS uid)) OR group.is_meeting_host(meeting_id, ( SELECT auth.uid() AS uid))) |  |
| meeting_programme_interest | mpi_insert | INSERT | authenticated |  | (( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) OR group.is_assigned_to_meeting(meeting_id, ( SELECT auth.uid() AS uid)) OR group.is_meeting_host(meeting_id, ( SELECT auth.uid() AS uid))) |
| meeting_programme_interest | mpi_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| meeting_programme_interest | mpi_update | UPDATE | authenticated | (( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) OR group.is_assigned_to_meeting(meeting_id, ( SELECT auth.uid() AS uid)) OR group.is_meeting_host(meeting_id, ( SELECT auth.uid() AS uid))) | (( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) OR group.is_assigned_to_meeting(meeting_id, ( SELECT auth.uid() AS uid)) OR group.is_meeting_host(meeting_id, ( SELECT auth.uid() AS uid))) |
| meeting_report_runs | meeting_report_runs_read | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| meeting_report_settings | meeting_report_settings_read | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| meeting_report_settings | meeting_report_settings_write | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| meeting_sync_queue | Admins read sync queue | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| meeting_welcome_jobs | welcome jobs manageable by admins | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| meeting_welcome_jobs | welcome jobs readable by staff | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| meeting_zoom_links | meeting_zoom_links_admin | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| meeting_zoom_links | meeting_zoom_links_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| meetings | meetings_admin_all | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| meetings | meetings_member_assigned_select | SELECT | authenticated | (id IN ( SELECT group.my_assigned_meeting_ids() AS my_assigned_meeting_ids)) |  |
| meetings | meetings_member_assigned_update | UPDATE | authenticated | (id IN ( SELECT group.my_assigned_meeting_ids() AS my_assigned_meeting_ids)) |  |
| meetings | meetings_staff_select | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| meetings_sheet_sync_state | meetings_sheet_server_only | ALL | service_role | true | true |
| member_daily_email_runs | Owners and admins read member daily runs | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| member_daily_email_settings | Owners and admins read member daily settings | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| member_email_agent | Staff see agent settings | SELECT | authenticated | (( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) OR (member_id = ( SELECT auth.uid() AS uid))) |  |
| member_report_days | member_report_days_service | ALL | service_role | true | true |
| member_sheet_reports | member_sheet_service_only | ALL | service_role | true | true |
| notifications | notifications_admin_insert | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| notifications | notifications_own | ALL | authenticated | (user_id = ( SELECT auth.uid() AS uid)) | (user_id = ( SELECT auth.uid() AS uid)) |
| program_cohorts | Admins manage program_cohorts | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| program_cohorts | Staff can read program_cohorts | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| review_commitments | rc_delete | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| review_commitments | rc_read | SELECT | authenticated | ((user_id = ( SELECT auth.uid() AS uid)) OR ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner)) |  |
| review_commitments | rc_update | UPDATE | authenticated | ((user_id = ( SELECT auth.uid() AS uid)) OR ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner)) |  |
| review_commitments | rc_write | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| scheduled_emails | Admins and owners can manage scheduled emails | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| scheduled_emails | scheduled_emails_select_scoped | SELECT | authenticated | (( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) OR group.is_assigned_to_meeting(meeting_id, ( SELECT auth.uid() AS uid))) |  |
| school_aliases | aliases admin delete | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| school_aliases | aliases admin insert | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| school_aliases | aliases admin update | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| school_aliases | aliases read all authed | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| school_domains | school domains readable by staff | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| school_domains | school domains written by admins | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| school_memory | school_memory_admin_delete | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| school_memory | school_memory_staff_insert | INSERT | authenticated |  | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |
| school_memory | school_memory_staff_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| school_memory | school_memory_staff_update | UPDATE | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |
| school_merge_log | admins read merge log | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| school_ownership | so_delete | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| school_ownership | so_staff_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| school_ownership | so_update | UPDATE | authenticated | ((owner = ( SELECT auth.uid() AS uid)) OR ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner)) |  |
| school_ownership | so_write | INSERT | authenticated |  | ((owner = ( SELECT auth.uid() AS uid)) OR ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner)) |
| school_registry | school_registry admin delete | DELETE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| school_registry | school_registry admin update | UPDATE | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| school_registry | school_registry admin write | INSERT | authenticated |  | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| school_registry | school_registry_staff_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| school_research | school_research_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| sheet_import_stage | admins manage sheet import stage | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| sheet_import_stage | staff read sheet import stage | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| sop_academic_calendar | calendar_admin | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| sop_academic_calendar | calendar_read | SELECT | authenticated | (group.sop_actor() IS NOT NULL) |  |
| sop_lists | sop_lists_admin | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| sop_lists | sop_lists_read | SELECT | authenticated | (group.sop_actor() IS NOT NULL) |  |
| sop_pricing_floors | floors_admin | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| sop_pricing_floors | floors_read | SELECT | authenticated | (group.sop_actor() IS NOT NULL) |  |
| sop_targets | targets_admin | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| sop_targets | targets_read | SELECT | authenticated | (group.sop_actor() IS NOT NULL) |  |
| sync_runs | staff read sync runs | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| sync_state | Admins write sync_state | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| sync_state | Members read sync_state | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| system_health | system_health_read | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| tasks | tasks_admins_full | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| tasks | tasks_members_update_own | UPDATE | authenticated | ((assigned_to = ( SELECT auth.uid() AS uid)) OR ((meeting_id IS NOT NULL) AND group.is_assigned_to_meeting(meeting_id, ( SELECT auth.uid() AS uid)))) | ((assigned_to = ( SELECT auth.uid() AS uid)) OR ((meeting_id IS NOT NULL) AND group.is_assigned_to_meeting(meeting_id, ( SELECT auth.uid() AS uid)))) |
| tasks | tasks_members_view | SELECT | authenticated | ((assigned_to = ( SELECT auth.uid() AS uid)) OR ((meeting_id IS NOT NULL) AND group.is_assigned_to_meeting(meeting_id, ( SELECT auth.uid() AS uid)))) |  |
| team_mailbox_import | team_mailbox_import_admin_read | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| user_gmail_accounts | user_gmail_accounts_select | SELECT | authenticated | ((user_id = ( SELECT auth.uid() AS uid)) OR ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner)) |  |
| user_profiles | admin_manage_profiles | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| user_profiles | profiles_self_or_admin_read | SELECT | authenticated | ((( SELECT auth.uid() AS uid) = id) OR ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner)) |  |
| user_profiles | profiles_staff_directory_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| user_profiles | self_update_profile | UPDATE | authenticated | (( SELECT auth.uid() AS uid) = id) | ((( SELECT auth.uid() AS uid) = id) AND ((role)::text = ( SELECT r.role    FROM group.my_profile_role_active() r(role, is_active))) AND (is_active = ( SELECT r.is_active    FROM group.my_profile_role_active() r(role, is_active)))) |
| wa_agent_runs | owners read agent runs | SELECT | authenticated | ( SELECT group.has_role(( SELECT auth.uid() AS uid), 'owner'::group.app_role) AS has_role) |  |
| wa_inbound | owners read inbound | SELECT | authenticated | ( SELECT group.has_role(( SELECT auth.uid() AS uid), 'owner'::group.app_role) AS has_role) |  |
| wa_limits | limits managed by owners | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| wa_limits | limits readable by staff | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| wa_poll_state | admin_read_wa_poll_state | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| wa_sends | wa_sends_staff_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| wa_session_window | staff read session window | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| wa_settings | wa_settings_admin_write | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| wa_settings | wa_settings_staff_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| wa_templates | wa_templates_admin_write | ALL | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |
| wa_templates | wa_templates_staff_read | SELECT | authenticated | ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff) |  |
| webhook_logs | Admins read webhook logs | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |
| weekly_commitments | wc_own_delete | DELETE | authenticated | ((user_id = ( SELECT auth.uid() AS uid)) OR ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner)) |  |
| weekly_commitments | wc_own_read | SELECT | authenticated | ((user_id = ( SELECT auth.uid() AS uid)) OR ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner)) |  |
| weekly_commitments | wc_own_update | UPDATE | authenticated | ((user_id = ( SELECT auth.uid() AS uid)) OR ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner)) |  |
| weekly_commitments | wc_own_write | INSERT | authenticated |  | ((user_id = ( SELECT auth.uid() AS uid)) OR ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner)) |
| weekly_review_runs | admins read weekly reviews | SELECT | authenticated | ( SELECT group.is_admin_or_owner(( SELECT auth.uid() AS uid)) AS is_admin_or_owner) |  |

## Schema: public (119 policies)

| table | policy | cmd | roles | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|
| activities | staff_scoped_activities | SELECT | authenticated | (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id))) |  |
| activities_2026_08 | staff_scoped_activities_2026_08 | SELECT | authenticated | (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id))) |  |
| activities_2026_09 | staff_scoped_activities_2026_09 | SELECT | authenticated | (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id))) |  |
| activities_2026_10 | staff_scoped_activities_2026_10 | SELECT | authenticated | (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id))) |  |
| activities_default | staff_scoped_activities_default | SELECT | authenticated | (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id))) |  |
| affiliations | staff_scoped_affiliations | SELECT | authenticated | (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id))) |  |
| agent_runs | staff_scoped_agent_runs | SELECT | authenticated | (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id))) |  |
| approved_facts | staff_read_approved_facts | SELECT | authenticated | (auth_staff_id() IS NOT NULL) |  |
| autonomy_thresholds | staff_read_autonomy_thresholds | SELECT | authenticated | (auth_staff_id() IS NOT NULL) |  |
| campaigns | staff_read_campaigns | SELECT | authenticated | (auth_staff_id() IS NOT NULL) |  |
| cohorts | staff_read_cohorts | SELECT | authenticated | (auth_staff_id() IS NOT NULL) |  |
| consents | staff_scoped_consents | SELECT | authenticated | (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id))) |  |
| household_members | staff_scoped_household_members | SELECT | authenticated | (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id))) |  |
| households | staff_scoped_households | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) AND (( SELECT has_scope_all() AS has_scope_all) OR (EXISTS ( SELECT 1    FROM household_members hm   WHERE ((hm.household_id = households.id) AND can_see_person(hm.person_id)))))) |  |
| inbound_records | inbound_records_read | SELECT | authenticated | (( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) |  |
| li_connections | li_connections_owner_all | ALL | authenticated | (user_id = ( SELECT auth.uid() AS uid)) | (user_id = ( SELECT auth.uid() AS uid)) |
| li_content_posts | li_content_posts_owner_all | ALL | authenticated | (user_id = ( SELECT auth.uid() AS uid)) | (user_id = ( SELECT auth.uid() AS uid)) |
| li_daily_runs | li_daily_runs_owner_all | ALL | authenticated | (user_id = ( SELECT auth.uid() AS uid)) | (user_id = ( SELECT auth.uid() AS uid)) |
| li_linkedin_connections | li_linkedin_connections_select_own | SELECT | authenticated | (user_id = auth.uid()) |  |
| li_research_items | li_research_items_owner_all | ALL | authenticated | (user_id = ( SELECT auth.uid() AS uid)) | (user_id = ( SELECT auth.uid() AS uid)) |
| li_work_updates | li_work_updates_owner_all | ALL | authenticated | (user_id = ( SELECT auth.uid() AS uid)) | (user_id = ( SELECT auth.uid() AS uid)) |
| memberships | staff_scoped_memberships | SELECT | authenticated | (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id))) |  |
| merge_lineage | staff_read_merge_lineage | SELECT | authenticated | (auth_staff_id() IS NOT NULL) |  |
| messages | staff_comms_messages | SELECT | authenticated | (( SELECT has_grant('comms_read'::staff_grant) AS has_grant) AND (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id)))) |  |
| org_aliases | staff_read_org_aliases | SELECT | authenticated | (auth_staff_id() IS NOT NULL) |  |
| organizations | staff_read_organizations | SELECT | authenticated | (auth_staff_id() IS NOT NULL) |  |
| people | staff_scoped_people | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) AND (( SELECT has_scope_all() AS has_scope_all) OR can_see_person(id))) |  |
| person_identities | staff_scoped_person_identities | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) AND (( SELECT has_scope_all() AS has_scope_all) OR can_see_person(person_id))) |  |
| programmes | staff_read_programmes | SELECT | authenticated | (auth_staff_id() IS NOT NULL) |  |
| research_activity | staff read activity | SELECT | authenticated | (EXISTS ( SELECT 1    FROM research_staff_profiles p   WHERE (p.id = auth.uid()))) |  |
| research_agent_runs | staff read agent runs | SELECT | authenticated | (EXISTS ( SELECT 1    FROM research_staff_profiles p   WHERE (p.id = auth.uid()))) |  |
| research_agent_settings | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_allocations | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_alumni | research_alumni_read | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_alumni | research_alumni_write | ALL | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |
| research_applications | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_autopilot | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_b2c_contacts | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_b2c_people | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_campaign_contacts | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_campaign_reports | staff manage campaign reports | ALL | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |
| research_campaigns | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_cell_flags | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_claims | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_contact_segments | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_contacts | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_datasets | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_designation_proposals | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_district_directories | district directories staff read | SELECT | authenticated | (EXISTS ( SELECT 1    FROM research_staff_profiles p   WHERE ((p.user_id = auth.uid()) AND p.active))) |  |
| research_domain_checks | staff read domain checks | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_domain_sites | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_duplicate_log | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_email_check_runs | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_email_check_state | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_email_checks | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_email_health | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_email_templates | staff manage templates | ALL | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |
| research_events | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_field_changes | Staff read change log | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_field_defs | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_generated_docs | generated docs readable by staff | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_groups | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_identity_matches | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_intel_chats | intel chats readable by staff | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_intel_messages | intel messages readable by staff | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_job_locks | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_jobs | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_leads | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_locations | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_optouts | staff read optouts | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_outreach_claims | staff manage outreach claims | ALL | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |
| research_pages | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_people | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_person_emails | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_person_events | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_person_links | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_person_phones | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_person_schools | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_pipeline_links | staff read pipeline links | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_proposals | proposals readable by staff | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_raw_rows | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_record_actions | staff read record actions | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_record_actions | staff update record actions | UPDATE | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |
| research_record_actions | staff write record actions | INSERT | authenticated |  | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |
| research_registry_schools | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_resolve_items | staff read resolve items | SELECT | authenticated | (EXISTS ( SELECT 1    FROM research_staff_profiles p   WHERE (p.id = auth.uid()))) |  |
| research_resolve_runs | staff read resolve runs | SELECT | authenticated | (EXISTS ( SELECT 1    FROM research_staff_profiles p   WHERE (p.id = auth.uid()))) |  |
| research_review_outcomes | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_review_rows | Staff can read staged review rows | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_saved_views | saved views readable by staff | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_school_facts | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_school_findings | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_school_intel | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_school_notes | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_school_profiles | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_school_web | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_schools | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_segments | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_send_exports | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_settings | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_short_links | staff read short links | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_sop_acknowledgements | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_sop_versions | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_spend | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_staff_profiles | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_tasks | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_territory_briefs | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_tracking_events | staff read tracking events | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_tracking_tokens | staff read tracking tokens | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| research_upload_tabs | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_uploads | staff can read | SELECT | authenticated | (research_role(auth.uid()) IS NOT NULL) |  |
| research_view_snapshots | view snapshots readable by staff | SELECT | authenticated | ((( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) OR ( SELECT group.is_active_staff(( SELECT auth.uid() AS uid)) AS is_active_staff)) |  |
| send_attempts | staff_comms_send_attempts | SELECT | authenticated | (( SELECT has_grant('comms_read'::staff_grant) AS has_grant) AND (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id)))) |  |
| sequence_steps | staff_read_sequence_steps | SELECT | authenticated | (auth_staff_id() IS NOT NULL) |  |
| sequences | staff_read_sequences | SELECT | authenticated | (auth_staff_id() IS NOT NULL) |  |
| source_codes | staff_read_source_codes | SELECT | authenticated | (auth_staff_id() IS NOT NULL) |  |
| source_systems | source_systems_read | SELECT | authenticated | (( SELECT auth_staff_id() AS auth_staff_id) IS NOT NULL) |  |
| suppressions | staff_read_suppressions | SELECT | authenticated | (auth_staff_id() IS NOT NULL) |  |
| tasks | staff_scoped_tasks | SELECT | authenticated | (( SELECT has_scope_all() AS has_scope_all) OR ((person_id IS NOT NULL) AND can_see_person(person_id))) |  |

## Schema: storage (12 policies)

| table | policy | cmd | roles | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|
| objects | lwl_email_attach_admin_all | ALL | authenticated | ((bucket_id = 'email-attachments'::text) AND group.is_admin_or_owner(auth.uid())) | ((bucket_id = 'email-attachments'::text) AND group.is_admin_or_owner(auth.uid())) |
| objects | lwl_email_attach_member_select | SELECT | authenticated | ((bucket_id = 'email-attachments'::text) AND (EXISTS ( SELECT 1    FROM (group.email_sends es      CROSS JOIN LATERAL jsonb_array_elements(         CASE             WHEN (jsonb_typeof(es.attachments) = 'array'::text) THEN es.attachments             ELSE '[]'::jsonb         END) a(item))   WHERE ((es.meeting_id IS NOT NULL) AND group.is_assigned_to_meeting(es.meeting_id, auth.uid()) AND ((objects.name =         CASE             WHEN (jsonb_typeof(a.item) = 'string'::text) THEN (a.item #>> '{}'::text[])             ELSE NULL::text         END) OR (objects.name = (a.item ->> 'path'::text)) OR (objects.name = (a.item ->> 'name'::text)) OR (objects.name = (a.item ->> 'storage_path'::text))))))) |  |
| objects | lwl_email_attach_staff_delete | DELETE | authenticated | ((bucket_id = 'email-attachments'::text) AND (owner = auth.uid()) AND group.is_active_staff(auth.uid())) |  |
| objects | lwl_email_attach_staff_insert | INSERT | authenticated |  | ((bucket_id = 'email-attachments'::text) AND (owner = auth.uid()) AND group.is_active_staff(auth.uid())) |
| objects | lwl_email_attach_staff_own_select | SELECT | authenticated | ((bucket_id = 'email-attachments'::text) AND (owner = auth.uid()) AND group.is_active_staff(auth.uid())) |  |
| objects | lwl_email_attach_staff_update | UPDATE | authenticated | ((bucket_id = 'email-attachments'::text) AND (owner = auth.uid()) AND group.is_active_staff(auth.uid())) | ((bucket_id = 'email-attachments'::text) AND (owner = auth.uid()) AND group.is_active_staff(auth.uid())) |
| objects | lwl_task_attach_admin_all | ALL | authenticated | ((bucket_id = 'task-attachments'::text) AND group.is_admin_or_owner(auth.uid())) | ((bucket_id = 'task-attachments'::text) AND group.is_admin_or_owner(auth.uid())) |
| objects | lwl_task_attach_member_select | SELECT | authenticated | ((bucket_id = 'task-attachments'::text) AND (EXISTS ( SELECT 1    FROM group.tasks t   WHERE (((t.id)::text = split_part(objects.name, '/'::text, 1)) AND ((t.assigned_to = auth.uid()) OR ((t.meeting_id IS NOT NULL) AND group.is_assigned_to_meeting(t.meeting_id, auth.uid()))))))) |  |
| objects | lwl_task_attach_staff_delete | DELETE | authenticated | ((bucket_id = 'task-attachments'::text) AND (owner = auth.uid()) AND group.is_active_staff(auth.uid())) |  |
| objects | lwl_task_attach_staff_insert | INSERT | authenticated |  | ((bucket_id = 'task-attachments'::text) AND (owner = auth.uid()) AND group.is_active_staff(auth.uid())) |
| objects | lwl_task_attach_staff_own_select | SELECT | authenticated | ((bucket_id = 'task-attachments'::text) AND (owner = auth.uid()) AND group.is_active_staff(auth.uid())) |  |
| objects | lwl_task_attach_staff_update | UPDATE | authenticated | ((bucket_id = 'task-attachments'::text) AND (owner = auth.uid()) AND group.is_active_staff(auth.uid())) | ((bucket_id = 'task-attachments'::text) AND (owner = auth.uid()) AND group.is_active_staff(auth.uid())) |

