# Brain project: triggers

Live snapshot taken 2026-11-24 from <DATA_REF>. All non-internal triggers (pg_trigger.tgisinternal = false), grouped by schema; definition is pg_get_triggerdef with the leading "CREATE TRIGGER name" removed.

**auth.users has 2 triggers**: `lwl_on_auth_user_created` AFTER INSERT -> group.handle_new_user_if_invited(), and `on_auth_user_created` AFTER INSERT -> public.accept_staff_invitation(). Both fire on every sign-up and are the mechanism by which an invited e-mail becomes an group.user_profiles row / public.staff row (no self-service staff creation).

Totals: group 64, leads 37, public 29, revenue_brain 15, storage 11, brain 9, attribution 7, auth 2, cron 1, realtime 1 (176).

## Schema: attribution (7 triggers)

| table | trigger | enabled | definition |
|---|---|---|---|
| campaigns | campaigns_permanent | enabled | BEFORE UPDATE ON attribution.campaigns FOR EACH ROW EXECUTE FUNCTION attribution.ids_are_permanent('campaign_id') |
| contacts | contacts_permanent | enabled | BEFORE UPDATE ON attribution.contacts FOR EACH ROW EXECUTE FUNCTION attribution.ids_are_permanent('contact_id') |
| contacts | contacts_school | enabled | BEFORE UPDATE ON attribution.contacts FOR EACH ROW EXECUTE FUNCTION attribution.ids_are_permanent('school_id') |
| outreach | outreach_permanent | enabled | BEFORE UPDATE ON attribution.outreach FOR EACH ROW EXECUTE FUNCTION attribution.ids_are_permanent('outreach_id') |
| outreach | outreach_rules | enabled | BEFORE INSERT OR UPDATE ON attribution.outreach FOR EACH ROW EXECUTE FUNCTION attribution.enforce_rules() |
| outreach | outreach_suppress | enabled | BEFORE UPDATE OF reply_status ON attribution.outreach FOR EACH ROW EXECUTE FUNCTION attribution.suppress_on_reply() |
| schools | schools_permanent | enabled | BEFORE UPDATE ON attribution.schools FOR EACH ROW EXECUTE FUNCTION attribution.ids_are_permanent('school_id') |

## Schema: auth (2 triggers)

| table | trigger | enabled | definition |
|---|---|---|---|
| users | lwl_on_auth_user_created | enabled | AFTER INSERT ON auth.users FOR EACH ROW EXECUTE FUNCTION group.handle_new_user_if_invited() |
| users | on_auth_user_created | enabled | AFTER INSERT ON auth.users FOR EACH ROW EXECUTE FUNCTION accept_staff_invitation() |

## Schema: brain (9 triggers)

| table | trigger | enabled | definition |
|---|---|---|---|
| knowledge_assets | knowledge_assets_touch | enabled | BEFORE UPDATE ON brain.knowledge_assets FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| knowledge_base | knowledge_base_touch | enabled | BEFORE UPDATE ON brain.knowledge_base FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| knowledge_cadence | knowledge_cadence_touch | enabled | BEFORE UPDATE ON brain.knowledge_cadence FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| knowledge_faq | knowledge_faq_touch | enabled | BEFORE UPDATE ON brain.knowledge_faq FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| knowledge_intents | knowledge_intents_touch | enabled | BEFORE UPDATE ON brain.knowledge_intents FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| knowledge_org | knowledge_org_touch | enabled | BEFORE UPDATE ON brain.knowledge_org FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| knowledge_programs | knowledge_programs_touch | enabled | BEFORE UPDATE ON brain.knowledge_programs FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| knowledge_scenarios | knowledge_scenarios_touch | enabled | BEFORE UPDATE ON brain.knowledge_scenarios FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| knowledge_voice_rules | knowledge_voice_rules_touch | enabled | BEFORE UPDATE ON brain.knowledge_voice_rules FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |

## Schema: cron (1 triggers)

| table | trigger | enabled | definition |
|---|---|---|---|
| job | cron_job_cache_invalidate | enabled | AFTER INSERT OR DELETE OR UPDATE OR TRUNCATE ON cron.job FOR EACH STATEMENT EXECUTE FUNCTION cron.job_cache_invalidate() |

## Schema: leads (37 triggers)

| table | trigger | enabled | definition |
|---|---|---|---|
| agent_jobs | trg_agent_jobs_updated_at | enabled | BEFORE UPDATE ON leads.agent_jobs FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| deals | deals_price_conflict | enabled | BEFORE INSERT OR UPDATE OF price_quoted_minor ON leads.deals FOR EACH ROW EXECUTE FUNCTION leads.deals_flag_price_conflict() |
| deals | deals_stage_stamp | enabled | BEFORE INSERT OR UPDATE ON leads.deals FOR EACH ROW EXECUTE FUNCTION leads.deals_stage_stamp() |
| email_sends | trg_tasks_on_email_draft | enabled | AFTER INSERT ON leads.email_sends FOR EACH ROW EXECUTE FUNCTION leads.tasks_on_email_draft() |
| email_sends | trg_tasks_resolve_on_email_change | enabled | AFTER UPDATE ON leads.email_sends FOR EACH ROW EXECUTE FUNCTION leads.tasks_resolve_on_email_change() |
| email_sends | trg_tasks_resolve_on_email_delete | enabled | BEFORE DELETE ON leads.email_sends FOR EACH ROW EXECUTE FUNCTION leads.tasks_resolve_on_email_delete() |
| email_templates | trg_email_templates_updated_at | enabled | BEFORE UPDATE ON leads.email_templates FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| knowledge_assets | trg_knowledge_assets_updated | enabled | BEFORE UPDATE ON leads.knowledge_assets FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| knowledge_base | knowledge_base_set_updated_at | enabled | BEFORE UPDATE ON leads.knowledge_base FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| knowledge_cadence | trg_knowledge_cadence_updated | enabled | BEFORE UPDATE ON leads.knowledge_cadence FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| knowledge_faq | trg_knowledge_faq_updated | enabled | BEFORE UPDATE ON leads.knowledge_faq FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| knowledge_intents | trg_knowledge_intents_updated | enabled | BEFORE UPDATE ON leads.knowledge_intents FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| knowledge_org | trg_knowledge_org_updated | enabled | BEFORE UPDATE ON leads.knowledge_org FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| knowledge_programs | trg_knowledge_programs_updated | enabled | BEFORE UPDATE ON leads.knowledge_programs FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| knowledge_scenarios | trg_knowledge_scenarios_updated | enabled | BEFORE UPDATE ON leads.knowledge_scenarios FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| knowledge_voice_rules | trg_knowledge_voice_updated | enabled | BEFORE UPDATE ON leads.knowledge_voice_rules FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| lead_ai_calls | trg_lead_ai_calls_updated_at | enabled | BEFORE UPDATE ON leads.lead_ai_calls FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| applications | trg_lwl_apps_updated | enabled | BEFORE UPDATE ON leads.applications FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| meeting_assignees | meeting_assignees_propagate_tasks | enabled | AFTER INSERT ON leads.meeting_assignees FOR EACH ROW EXECUTE FUNCTION leads.propagate_assignee_to_tasks() |
| meeting_assignees | trg_notify_on_assignment | enabled | AFTER INSERT ON leads.meeting_assignees FOR EACH ROW EXECUTE FUNCTION leads.notify_on_assignment() |
| meeting_sync_queue | meeting_sync_queue_set_updated_at | enabled | BEFORE UPDATE ON leads.meeting_sync_queue FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| meetings | meetings_lifecycle_sync | enabled | BEFORE UPDATE ON leads.meetings FOR EACH ROW EXECUTE FUNCTION leads.sync_lifecycle_from_status() |
| meetings | meetings_updated_at | enabled | BEFORE UPDATE ON leads.meetings FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| meetings | trg_clear_score_on_non_completion | enabled | BEFORE INSERT OR UPDATE OF meeting_status ON leads.meetings FOR EACH ROW EXECUTE FUNCTION leads.clear_score_on_non_completion() |
| meetings | trg_guard_member_meeting_update | enabled | BEFORE UPDATE ON leads.meetings FOR EACH ROW EXECUTE FUNCTION leads.guard_member_meeting_update() |
| meetings | trg_meetings_ensure_utm_direct | enabled | BEFORE INSERT OR UPDATE ON leads.meetings FOR EACH ROW EXECUTE FUNCTION leads.ensure_utm_direct() |
| meetings | trg_sync_lead_segment | enabled | BEFORE INSERT OR UPDATE OF designation ON leads.meetings FOR EACH ROW EXECUTE FUNCTION leads.sync_lead_segment_from_designation() |
| program_cohorts | trg_program_cohorts_updated | enabled | BEFORE UPDATE ON leads.program_cohorts FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| scheduled_emails | scheduled_emails_set_updated_at | enabled | BEFORE UPDATE ON leads.scheduled_emails FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| school_aliases | trg_school_aliases_updated | enabled | BEFORE UPDATE ON leads.school_aliases FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| school_registry | trg_school_registry_updated | enabled | BEFORE UPDATE ON leads.school_registry FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| sync_state | trg_sync_state_updated | enabled | BEFORE UPDATE ON leads.sync_state FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| tasks | trg_tasks_completed_at | enabled | BEFORE UPDATE ON leads.tasks FOR EACH ROW EXECUTE FUNCTION leads.tasks_set_completed_at() |
| tasks | trg_tasks_updated_at | enabled | BEFORE UPDATE ON leads.tasks FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| user_profiles | trg_guard_role | enabled | BEFORE UPDATE ON leads.user_profiles FOR EACH ROW EXECUTE FUNCTION leads.guard_profile_role_change() |
| user_profiles | trg_user_profiles_updated_at | enabled | BEFORE UPDATE ON leads.user_profiles FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |
| webhook_logs | update_webhook_logs_updated_at | enabled | BEFORE UPDATE ON leads.webhook_logs FOR EACH ROW EXECUTE FUNCTION leads.update_updated_at_column() |

## Schema: group (64 triggers)

| table | trigger | enabled | definition |
|---|---|---|---|
| agent_jobs | trg_agent_jobs_updated_at | enabled | BEFORE UPDATE ON group.agent_jobs FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| automations | trg_automations_apply_cron | enabled | AFTER UPDATE ON group.automations FOR EACH ROW EXECUTE FUNCTION group.automations_apply_cron() |
| core_applications | trg_core_applications_utm_member | enabled | BEFORE INSERT OR UPDATE OF utm_source, utm_medium, utm_campaign, referral_source ON group.core_applications FOR EACH ROW EXECUTE FUNCTION group.core_applications_set_utm_member() |
| deals | deals_guard_merge | enabled | BEFORE INSERT OR UPDATE OF merged_into ON group.deals FOR EACH ROW EXECUTE FUNCTION group.deals_guard_merge() |
| deals | deals_price_conflict | enabled | BEFORE INSERT OR UPDATE OF price_quoted_minor ON group.deals FOR EACH ROW EXECUTE FUNCTION group.deals_flag_price_conflict() |
| deals | deals_stage_stamp | enabled | BEFORE INSERT OR UPDATE ON group.deals FOR EACH ROW EXECUTE FUNCTION group.deals_stage_stamp() |
| email_sends | email_sends_sheet_kick_ins | enabled | AFTER INSERT ON group.email_sends REFERENCING NEW TABLE AS new_rows FOR EACH STATEMENT EXECUTE FUNCTION group.meetings_sheet_kick_mail() |
| email_sends | email_sends_sheet_kick_upd | enabled | AFTER UPDATE ON group.email_sends REFERENCING NEW TABLE AS new_rows FOR EACH STATEMENT EXECUTE FUNCTION group.meetings_sheet_kick_mail() |
| email_sends | member_report_email_change | enabled | AFTER INSERT OR DELETE OR UPDATE ON group.email_sends FOR EACH STATEMENT EXECUTE FUNCTION group.member_sheet_report_kick() |
| email_sends | member_report_email_dirty | enabled | AFTER INSERT OR DELETE OR UPDATE ON group.email_sends FOR EACH ROW EXECUTE FUNCTION group.member_report_email_dirty() |
| email_sends | trg_tasks_on_email_draft | enabled | AFTER INSERT ON group.email_sends FOR EACH ROW EXECUTE FUNCTION group.tasks_on_email_draft() |
| email_sends | trg_tasks_on_inbound_reply | enabled | AFTER INSERT ON group.email_sends FOR EACH ROW EXECUTE FUNCTION group.tasks_on_inbound_reply() |
| email_sends | trg_tasks_resolve_on_email_change | enabled | AFTER UPDATE ON group.email_sends FOR EACH ROW EXECUTE FUNCTION group.tasks_resolve_on_email_change() |
| email_sends | trg_tasks_resolve_on_email_delete | enabled | BEFORE DELETE ON group.email_sends FOR EACH ROW EXECUTE FUNCTION group.tasks_resolve_on_email_delete() |
| email_templates | trg_email_templates_updated_at | enabled | BEFORE UPDATE ON group.email_templates FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| followup_plans | followup_plans_updated_at | enabled | BEFORE UPDATE ON group.followup_plans FOR EACH ROW EXECUTE FUNCTION group.touch_followup_updated_at() |
| followup_touches | followup_touches_updated_at | enabled | BEFORE UPDATE ON group.followup_touches FOR EACH ROW EXECUTE FUNCTION group.touch_followup_updated_at() |
| knowledge_assets | trg_knowledge_assets_updated | enabled | BEFORE UPDATE ON group.knowledge_assets FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| knowledge_base | knowledge_base_set_updated_at | enabled | BEFORE UPDATE ON group.knowledge_base FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| knowledge_cadence | trg_knowledge_cadence_updated | enabled | BEFORE UPDATE ON group.knowledge_cadence FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| knowledge_faq | trg_knowledge_faq_updated | enabled | BEFORE UPDATE ON group.knowledge_faq FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| knowledge_intents | trg_knowledge_intents_updated | enabled | BEFORE UPDATE ON group.knowledge_intents FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| knowledge_org | trg_knowledge_org_updated | enabled | BEFORE UPDATE ON group.knowledge_org FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| knowledge_programs | trg_knowledge_programs_updated | enabled | BEFORE UPDATE ON group.knowledge_programs FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| knowledge_scenarios | trg_knowledge_scenarios_updated | enabled | BEFORE UPDATE ON group.knowledge_scenarios FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| knowledge_voice_rules | trg_knowledge_voice_updated | enabled | BEFORE UPDATE ON group.knowledge_voice_rules FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| lead_ai_calls | trg_lead_ai_calls_updated_at | enabled | BEFORE UPDATE ON group.lead_ai_calls FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| applications | trg_lwl_apps_updated | enabled | BEFORE UPDATE ON group.applications FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| meeting_assignees | meeting_assignees_propagate_tasks | enabled | AFTER INSERT ON group.meeting_assignees FOR EACH ROW EXECUTE FUNCTION group.propagate_assignee_to_tasks() |
| meeting_assignees | meeting_assignees_sync_assigned_to | enabled | AFTER INSERT OR DELETE OR UPDATE ON group.meeting_assignees FOR EACH ROW EXECUTE FUNCTION group.trg_sync_meeting_assigned_to() |
| meeting_assignees | member_report_assignment_dirty | enabled | AFTER INSERT OR DELETE OR UPDATE ON group.meeting_assignees FOR EACH ROW EXECUTE FUNCTION group.member_report_assignment_dirty() |
| meeting_assignees | member_report_owner_change | enabled | AFTER INSERT OR DELETE OR UPDATE ON group.meeting_assignees FOR EACH STATEMENT EXECUTE FUNCTION group.member_sheet_report_kick() |
| meeting_assignees | trg_notify_meeting_freed | enabled | AFTER DELETE ON group.meeting_assignees FOR EACH ROW EXECUTE FUNCTION group.notify_meeting_freed() |
| meeting_assignees | trg_notify_on_assignment | enabled | AFTER INSERT ON group.meeting_assignees FOR EACH ROW EXECUTE FUNCTION group.notify_on_assignment() |
| meeting_followups | member_report_followup_change | enabled | AFTER INSERT OR DELETE OR UPDATE ON group.meeting_followups FOR EACH STATEMENT EXECUTE FUNCTION group.member_sheet_report_kick() |
| meeting_followups | member_report_followup_dirty | enabled | AFTER INSERT OR DELETE OR UPDATE ON group.meeting_followups FOR EACH ROW EXECUTE FUNCTION group.member_report_email_dirty() |
| meeting_participants | trg_meeting_participants_autoassign | enabled | AFTER INSERT OR UPDATE OF left_at, participant_kind, staff_user_id ON group.meeting_participants FOR EACH ROW EXECUTE FUNCTION group.trg_zoom_autoassign() |
| meeting_participants | trg_touch_meeting_participants | enabled | BEFORE UPDATE ON group.meeting_participants FOR EACH ROW EXECUTE FUNCTION group.touch_meeting_welcome_jobs() |
| meeting_sync_queue | meeting_sync_queue_set_updated_at | enabled | BEFORE UPDATE ON group.meeting_sync_queue FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| meeting_welcome_jobs | trg_touch_meeting_welcome_jobs | enabled | BEFORE UPDATE ON group.meeting_welcome_jobs FOR EACH ROW EXECUTE FUNCTION group.touch_meeting_welcome_jobs() |
| meetings | a_meeting_manual_lock | enabled | BEFORE UPDATE ON group.meetings FOR EACH ROW EXECUTE FUNCTION group.meeting_manual_lock() |
| meetings | meetings_lifecycle_sync | enabled | BEFORE UPDATE ON group.meetings FOR EACH ROW EXECUTE FUNCTION group.sync_lifecycle_from_status() |
| meetings | meetings_sheet_kick_ins | enabled | AFTER INSERT ON group.meetings REFERENCING NEW TABLE AS new_rows FOR EACH STATEMENT EXECUTE FUNCTION group.meetings_sheet_kick() |
| meetings | meetings_sheet_kick_upd | enabled | AFTER UPDATE ON group.meetings REFERENCING OLD TABLE AS old_rows NEW TABLE AS new_rows FOR EACH STATEMENT EXECUTE FUNCTION group.meetings_sheet_kick() |
| meetings | meetings_updated_at | enabled | BEFORE UPDATE ON group.meetings FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| meetings | member_report_meeting_change | enabled | AFTER INSERT OR DELETE OR UPDATE ON group.meetings FOR EACH STATEMENT EXECUTE FUNCTION group.member_sheet_report_kick() |
| meetings | member_report_meeting_dirty | enabled | AFTER INSERT OR DELETE OR UPDATE ON group.meetings FOR EACH ROW EXECUTE FUNCTION group.member_report_meeting_dirty() |
| meetings | trg_clear_score_on_non_completion | enabled | BEFORE INSERT OR UPDATE OF meeting_status ON group.meetings FOR EACH ROW EXECUTE FUNCTION group.clear_score_on_non_completion() |
| meetings | trg_guard_member_meeting_update | enabled | BEFORE UPDATE ON group.meetings FOR EACH ROW EXECUTE FUNCTION group.guard_member_meeting_update() |
| meetings | trg_meetings_set_booking_rep | enabled | BEFORE INSERT OR UPDATE OF calendly_host_email ON group.meetings FOR EACH ROW EXECUTE FUNCTION group.meetings_set_booking_rep() |
| meetings | trg_notify_unassigned_meeting | enabled | AFTER INSERT ON group.meetings FOR EACH ROW EXECUTE FUNCTION group.notify_unassigned_meeting() |
| meetings | trg_sync_lead_type | enabled | BEFORE INSERT OR UPDATE OF designation, email, school_name, school_code ON group.meetings FOR EACH ROW EXECUTE FUNCTION group.sync_lead_type_from_designation() |
| member_sheet_reports | member_sheet_report_touch | enabled | BEFORE UPDATE ON group.member_sheet_reports FOR EACH ROW EXECUTE FUNCTION group.member_sheet_report_touch() |
| program_cohorts | trg_program_cohorts_updated | enabled | BEFORE UPDATE ON group.program_cohorts FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| scheduled_emails | scheduled_emails_set_updated_at | enabled | BEFORE UPDATE ON group.scheduled_emails FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| school_aliases | trg_school_aliases_updated | enabled | BEFORE UPDATE ON group.school_aliases FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| school_registry | trg_school_registry_updated | enabled | BEFORE UPDATE ON group.school_registry FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| sync_state | trg_sync_state_updated | enabled | BEFORE UPDATE ON group.sync_state FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| tasks | trg_tasks_completed_at | enabled | BEFORE UPDATE ON group.tasks FOR EACH ROW EXECUTE FUNCTION group.tasks_set_completed_at() |
| tasks | trg_tasks_updated_at | enabled | BEFORE UPDATE ON group.tasks FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| user_gmail_accounts | trg_user_gmail_accounts_updated_at | enabled | BEFORE UPDATE ON group.user_gmail_accounts FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| user_profiles | trg_guard_role | enabled | BEFORE UPDATE ON group.user_profiles FOR EACH ROW EXECUTE FUNCTION group.guard_profile_role_change() |
| user_profiles | trg_user_profiles_updated_at | enabled | BEFORE UPDATE ON group.user_profiles FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |
| webhook_logs | update_webhook_logs_updated_at | enabled | BEFORE UPDATE ON group.webhook_logs FOR EACH ROW EXECUTE FUNCTION group.update_updated_at_column() |

## Schema: public (29 triggers)

| table | trigger | enabled | definition |
|---|---|---|---|
| allocations | allocations_no_overlap | enabled | BEFORE INSERT OR UPDATE ON public.allocations FOR EACH ROW EXECUTE FUNCTION allocation_no_overlap() |
| autonomy_thresholds | autonomy_thresholds_touch | enabled | BEFORE UPDATE ON public.autonomy_thresholds FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| campaigns | campaigns_touch | enabled | BEFORE UPDATE ON public.campaigns FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| cohorts | cohorts_touch | enabled | BEFORE UPDATE ON public.cohorts FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| meetings | meetings_link_pitched_programme | enabled | BEFORE INSERT OR UPDATE ON public.meetings FOR EACH ROW EXECUTE FUNCTION link_pitched_programme() |
| meetings | meetings_source_code | enabled | BEFORE INSERT OR UPDATE ON public.meetings FOR EACH ROW EXECUTE FUNCTION guard_source_code() |
| meetings | meetings_to_activity | enabled | AFTER INSERT OR UPDATE ON public.meetings FOR EACH ROW EXECUTE FUNCTION meeting_to_activity() |
| meetings | meetings_touch | enabled | BEFORE UPDATE ON public.meetings FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| memberships | memberships_touch | enabled | BEFORE UPDATE ON public.memberships FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| messages | messages_normalise_direction | enabled | BEFORE INSERT OR UPDATE ON public.messages FOR EACH ROW EXECUTE FUNCTION normalise_message_direction() |
| messages | messages_touch | enabled | BEFORE UPDATE ON public.messages FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| organizations | organizations_source_code | enabled | BEFORE INSERT OR UPDATE ON public.organizations FOR EACH ROW EXECUTE FUNCTION guard_source_code() |
| organizations | organizations_touch | enabled | BEFORE UPDATE ON public.organizations FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| payments | payments_source_code | enabled | BEFORE INSERT OR UPDATE ON public.payments FOR EACH ROW EXECUTE FUNCTION guard_source_code() |
| payments | payments_touch | enabled | BEFORE UPDATE ON public.payments FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| people | people_source_code | enabled | BEFORE INSERT OR UPDATE ON public.people FOR EACH ROW EXECUTE FUNCTION guard_source_code() |
| people | people_touch | enabled | BEFORE UPDATE ON public.people FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| person_identities | identities_source_code | enabled | BEFORE INSERT OR UPDATE ON public.person_identities FOR EACH ROW EXECUTE FUNCTION guard_source_code() |
| person_identities | person_identities_key | enabled | BEFORE INSERT OR UPDATE OF kind, value_raw, value_norm ON public.person_identities FOR EACH ROW EXECUTE FUNCTION set_identity_key() |
| programmes | programmes_touch | enabled | BEFORE UPDATE ON public.programmes FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |
| refunds | refunds_not_before_payment | enabled | BEFORE INSERT OR UPDATE ON public.refunds FOR EACH ROW EXECUTE FUNCTION refund_not_before_payment() |
| refunds | refunds_source_code | enabled | BEFORE INSERT OR UPDATE ON public.refunds FOR EACH ROW EXECUTE FUNCTION guard_source_code() |
| refunds | refunds_within_payment | enabled | BEFORE INSERT OR UPDATE ON public.refunds FOR EACH ROW EXECUTE FUNCTION refund_within_payment() |
| research_contacts | research_contact_facets_sync | enabled | AFTER INSERT OR DELETE OR UPDATE OF deleted_at, category, person_type, school, country, state, city, email, subject, seniority, sop_priority, grade_band, school_type, school_type_confidence, grade_band_confidence, sop_level, curriculum, geo_region ON public.research_contacts FOR EACH ROW EXECUTE FUNCTION research_contact_facets_sync() |
| research_contacts | research_contact_facets_trust_sync | enabled | AFTER INSERT OR UPDATE OF trust_state, trust_reasons ON public.research_contacts FOR EACH ROW EXECUTE FUNCTION research_contact_facets_trust_sync() |
| research_contacts | research_contacts_tool_in | enabled | BEFORE INSERT OR UPDATE OF research_tool ON public.research_contacts FOR EACH ROW EXECUTE FUNCTION research_contacts_tool_in() |
| research_contacts | research_contacts_upload_folders | enabled | AFTER INSERT ON public.research_contacts FOR EACH ROW EXECUTE FUNCTION research_contacts_upload_folders() |
| research_duplicate_log | research_dup_merge_on_insert | enabled | AFTER INSERT ON public.research_duplicate_log FOR EACH ROW WHEN ((new.decision = 'pending'::text)) EXECUTE FUNCTION research_dup_merge_on_insert() |
| staff | staff_touch | enabled | BEFORE UPDATE ON public.staff FOR EACH ROW EXECUTE FUNCTION touch_updated_at() |

## Schema: realtime (1 triggers)

| table | trigger | enabled | definition |
|---|---|---|---|
| subscription | tr_check_filters | enabled | BEFORE INSERT OR UPDATE ON realtime.subscription FOR EACH ROW EXECUTE FUNCTION realtime.subscription_check_filters() |

## Schema: revenue_brain (15 triggers)

| table | trigger | enabled | definition |
|---|---|---|---|
| campaign_routes | trg_campaign_routes_updated_at | enabled | BEFORE UPDATE ON revenue_brain.campaign_routes FOR EACH ROW EXECUTE FUNCTION revenue_brain.set_updated_at() |
| claims_registry | trg_claims_registry_updated_at | enabled | BEFORE UPDATE ON revenue_brain.claims_registry FOR EACH ROW EXECUTE FUNCTION revenue_brain.set_updated_at() |
| eb_conversations | eb_t_conv_upd | enabled | BEFORE UPDATE ON revenue_brain.eb_conversations FOR EACH ROW EXECUTE FUNCTION revenue_brain.eb_touch_updated_at() |
| eb_leads | eb_t_leads_upd | enabled | BEFORE UPDATE ON revenue_brain.eb_leads FOR EACH ROW EXECUTE FUNCTION revenue_brain.eb_touch_updated_at() |
| eb_reply_drafts | eb_t_draft_upd | enabled | BEFORE UPDATE ON revenue_brain.eb_reply_drafts FOR EACH ROW EXECUTE FUNCTION revenue_brain.eb_touch_updated_at() |
| outbound_queue | trg_enforce_test_recipient_allowlist | enabled | BEFORE INSERT OR UPDATE OF delivery_email, test_mode ON revenue_brain.outbound_queue FOR EACH ROW EXECUTE FUNCTION revenue_brain.enforce_test_recipient_allowlist() |
| outbound_queue | trg_outbound_queue_updated_at | enabled | BEFORE UPDATE ON revenue_brain.outbound_queue FOR EACH ROW EXECUTE FUNCTION revenue_brain.set_updated_at() |
| outbound_queue | trg_production_kill_switch | enabled | BEFORE INSERT OR UPDATE OF queue_status, test_mode ON revenue_brain.outbound_queue FOR EACH ROW EXECUTE FUNCTION revenue_brain.enforce_production_kill_switch() |
| programme_profiles | trg_programme_profiles_updated_at | enabled | BEFORE UPDATE ON revenue_brain.programme_profiles FOR EACH ROW EXECUTE FUNCTION revenue_brain.set_updated_at() |
| school_program_state | trg_protect_paid_customer_state | enabled | BEFORE UPDATE OF relationship_state, school_lock_state ON revenue_brain.school_program_state FOR EACH ROW EXECUTE FUNCTION revenue_brain.protect_paid_customer_state() |
| school_program_state | trg_school_program_state_updated_at | enabled | BEFORE UPDATE ON revenue_brain.school_program_state FOR EACH ROW EXECUTE FUNCTION revenue_brain.set_updated_at() |
| sender_accounts | trg_sender_accounts_updated_at | enabled | BEFORE UPDATE ON revenue_brain.sender_accounts FOR EACH ROW EXECUTE FUNCTION revenue_brain.set_updated_at() |
| sequence_steps | trg_sequence_steps_updated_at | enabled | BEFORE UPDATE ON revenue_brain.sequence_steps FOR EACH ROW EXECUTE FUNCTION revenue_brain.set_updated_at() |
| sequence_templates | trg_sequence_templates_updated_at | enabled | BEFORE UPDATE ON revenue_brain.sequence_templates FOR EACH ROW EXECUTE FUNCTION revenue_brain.set_updated_at() |
| webhook_inbox | trg_instantly_campaign_ingress | enabled | BEFORE INSERT ON revenue_brain.webhook_inbox FOR EACH ROW EXECUTE FUNCTION revenue_brain.enforce_instantly_campaign_ingress() |

## Schema: storage (11 triggers)

| table | trigger | enabled | definition |
|---|---|---|---|
| buckets | enforce_bucket_name_length_trigger | enabled | BEFORE INSERT OR UPDATE OF name ON storage.buckets FOR EACH ROW EXECUTE FUNCTION storage.enforce_bucket_name_length() |
| buckets | enforce_versioning_status_transition | enabled | BEFORE UPDATE OF versioning_status ON storage.buckets FOR EACH ROW EXECUTE FUNCTION storage.enforce_versioning_status_transition() |
| buckets | protect_active_lifecycle_states | enabled | BEFORE DELETE ON storage.buckets FOR EACH ROW EXECUTE FUNCTION storage.protect_active_lifecycle_states() |
| buckets | protect_bucket_control_insert | enabled | BEFORE INSERT ON storage.buckets FOR EACH ROW EXECUTE FUNCTION storage.protect_bucket_control_columns('service_role') |
| buckets | protect_bucket_control_update | enabled | BEFORE UPDATE OF lifecycle_configuration, lifecycle_configuration_generation ON storage.buckets FOR EACH ROW EXECUTE FUNCTION storage.protect_bucket_control_columns() |
| buckets | protect_bucket_control_update_role | enabled | AFTER UPDATE OF lifecycle_configuration, lifecycle_configuration_generation ON storage.buckets FOR EACH ROW EXECUTE FUNCTION storage.enforce_bucket_lifecycle_service_role('service_role') |
| buckets | protect_buckets_delete | enabled | BEFORE DELETE ON storage.buckets FOR EACH STATEMENT EXECUTE FUNCTION storage.protect_delete() |
| buckets | wake_lifecycle_on_policy_write | enabled | AFTER INSERT OR UPDATE OF lifecycle_configuration_generation ON storage.buckets FOR EACH ROW EXECUTE FUNCTION storage.wake_lifecycle_state() |
| buckets | wake_lifecycle_on_versioning_enable | enabled | AFTER UPDATE OF versioning_status ON storage.buckets FOR EACH ROW WHEN ((new.lifecycle_configuration IS NOT NULL)) EXECUTE FUNCTION storage.wake_lifecycle_state() |
| objects | protect_objects_delete | enabled | BEFORE DELETE ON storage.objects FOR EACH STATEMENT EXECUTE FUNCTION storage.protect_delete() |
| objects | update_objects_updated_at | enabled | BEFORE UPDATE ON storage.objects FOR EACH ROW EXECUTE FUNCTION storage.update_updated_at_column() |

