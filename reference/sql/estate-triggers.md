# Shared estate triggers

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

## cron (1 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| job | cron_job_cache_invalidate | AFTER INSERT/DELETE/UPDATE/TRUNCATE STATEMENT | `cron.job_cache_invalidate()` | origin |

## premed (36 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| aeo_outputs | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| aeo_page_optimizations | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| aeo_pages | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| aeo_settings | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| app_settings | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| captured_leads | captured_leads_defaults | BEFORE INSERT ROW | `premed.captured_leads_set_defaults()` | origin |
| challenge_quiz_attempts | challenge_quiz_attempts_touch | BEFORE UPDATE ROW | `set_updated_at()` | origin |
| cohort_settings | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| concierge_ai_usage | concierge_ai_usage_touch | BEFORE UPDATE ROW | `touch_updated_at()` | origin |
| concierge_conversations | concierge_conversations_touch | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| email_templates | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| fellowship_projects | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| goal_reviews | goal_reviews_notify | AFTER INSERT ROW | `premed.notify_goal_review_published()` | origin |
| goal_reviews | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| intake_raw | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| knowledge_entries | knowledge_entries_search | BEFORE INSERT/UPDATE ROW | `premed.knowledge_entries_search_vector_update()` | origin |
| knowledge_entries | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| live_class_rsvps | live_class_rsvps_notify | AFTER INSERT ROW | `premed.notify_rsvp_created()` | origin |
| live_class_rsvps | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| masterclass_series_overrides | masterclass_series_overrides_updated_at | BEFORE UPDATE ROW | `premed.set_updated_at()` | origin |
| masterclass_sessions | masterclass_sessions_write | INSTEAD OF INSERT/DELETE/UPDATE ROW | `premed.tg_masterclass_sessions_write()` | origin |
| mmi_coach_sessions | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| parent_reports | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| program_attachments | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| program_email_settings | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| session_attendance | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| side_effects_queue | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| stripe_charge_cache | stripe_charge_cache_updated_at | BEFORE UPDATE ROW | `update_updated_at_column()` | origin |
| stripe_reconcile_state | stripe_reconcile_state_updated_at | BEFORE UPDATE ROW | `update_updated_at_column()` | origin |
| student_goals | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| student_goals | student_goals_notify | AFTER UPDATE ROW | `premed.notify_goal_completed()` | origin |
| student_university_applications | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| universities | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| user_profiles | set_updated_at | BEFORE UPDATE ROW | `premed.handle_updated_at()` | origin |
| user_profiles | user_profiles_no_escalation | BEFORE UPDATE ROW | `premed.prevent_role_escalation()` | origin |
| user_profiles | user_profiles_phone | BEFORE INSERT/UPDATE ROW | `premed.tg_normalize_profile_phone()` | origin |

## finance (14 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| attachments | set_updated_at | BEFORE UPDATE ROW | `set_updated_at()` | origin |
| codes_dropdown | code_sheet_sync | AFTER INSERT/DELETE/UPDATE STATEMENT | `finance.request_code_sheet_sync()` | origin |
| expense_codes | code_sheet_sync | AFTER INSERT/DELETE/UPDATE STATEMENT | `finance.request_code_sheet_sync()` | origin |
| overhead_allocations | set_updated_at | BEFORE UPDATE ROW | `set_updated_at()` | origin |
| overhead_entries | set_updated_at | BEFORE UPDATE ROW | `set_updated_at()` | origin |
| program_budgets | set_updated_at | BEFORE UPDATE ROW | `set_updated_at()` | origin |
| program_codes | code_sheet_sync | AFTER INSERT/DELETE/UPDATE STATEMENT | `finance.request_code_sheet_sync()` | origin |
| program_completions | set_updated_at | BEFORE UPDATE ROW | `set_updated_at()` | origin |
| program_marketing_metrics | set_updated_at | BEFORE UPDATE ROW | `set_updated_at()` | origin |
| travel_expenses | set_updated_at | BEFORE UPDATE ROW | `set_updated_at()` | origin |
| user_profiles | update_user_profiles_updated_at | BEFORE UPDATE ROW | `update_updated_at_column()` | origin |
| vendor_bills | set_updated_at | BEFORE UPDATE ROW | `set_updated_at()` | origin |
| vendors | set_updated_at | BEFORE UPDATE ROW | `set_updated_at()` | origin |
| xero_invoices | set_updated_at | BEFORE UPDATE ROW | `set_updated_at()` | origin |

## finedu (6 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| app_settings | trg_app_settings_touch | BEFORE UPDATE ROW | `finedu.touch_updated_at()` | origin |
| concierge_leads | trg_concierge_leads_touch | BEFORE UPDATE ROW | `finedu.touch_updated_at()` | origin |
| email_attachments | trg_email_attachments_touch | BEFORE UPDATE ROW | `finedu.touch_updated_at()` | origin |
| email_templates | trg_email_templates_touch | BEFORE UPDATE ROW | `finedu.touch_updated_at()` | origin |
| program_email_settings | trg_program_email_settings_touch | BEFORE UPDATE ROW | `finedu.touch_updated_at()` | origin |
| school_proposals | trg_school_proposals_touch | BEFORE UPDATE ROW | `finedu.touch_updated_at()` | origin |

## ops (2 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| agent_campaign_contacts | enforce_contact_suppression | BEFORE INSERT/UPDATE ROW | `ops.enforce_contact_suppression()` | origin |
| profiles | trg_profiles_updated | BEFORE UPDATE ROW | `ops.touch_updated_at()` | origin |

## prelaw (3 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| email_templates | email_templates_updated_at | BEFORE UPDATE ROW | `prelaw.set_updated_at()` | origin |
| unmatched_payments | unmatched_payments_updated_at | BEFORE UPDATE ROW | `prelaw.set_updated_at()` | origin |
| whatsapp_settings | whatsapp_settings_updated_at | BEFORE UPDATE ROW | `prelaw.set_updated_at()` | origin |

## lms (39 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| all_mentors | tg_all_mentors_view_del | INSTEAD OF DELETE ROW | `lms.tg_all_mentors_view_write()` | origin |
| all_mentors | tg_all_mentors_view_ins | INSTEAD OF INSERT ROW | `lms.tg_all_mentors_view_write()` | origin |
| all_mentors | tg_all_mentors_view_upd | INSTEAD OF UPDATE ROW | `lms.tg_all_mentors_view_write()` | origin |
| cohort_completions | vault_cohort_completed | AFTER INSERT/UPDATE ROW | `lms.tg_vault_cohort_completed()` | origin |
| cohort_group_members | group_member_section_check | BEFORE INSERT/UPDATE ROW | `lms.tg_group_member_section_check()` | origin |
| cohort_section_members | section_member_cleanup | AFTER DELETE/UPDATE ROW | `lms.tg_section_member_cleanup()` | origin |
| community_channels | add_community_welcome_message | AFTER INSERT ROW | `lms.add_community_welcome_message()` | origin |
| community_messages | community_mentions_record | AFTER INSERT ROW | `lms.record_community_mentions()` | origin |
| community_messages | community_messages_moderation | BEFORE INSERT/UPDATE ROW | `lms.community_messages_moderate()` | origin |
| community_messages | community_messages_stamp | BEFORE INSERT ROW | `lms.stamp_community_sender()` | origin |
| community_messages | zz_community_reminder_team | BEFORE INSERT ROW | `lms.stamp_community_automation_team()` | origin |
| email_templates | t_email_templates_touch | BEFORE UPDATE ROW | `lms.touch_updated_at()` | origin |
| enrollment_events | enrollment_events_immutable | BEFORE DELETE/UPDATE ROW | `lms.tg_enrollment_events_immutable()` | origin |
| enrollment_events | enrollment_events_no_truncate | BEFORE TRUNCATE STATEMENT | `lms.tg_enrollment_events_immutable()` | origin |
| enrollments | enrollments_ensure_channel | AFTER INSERT/UPDATE ROW | `lms.ensure_community_channel()` | origin |
| enrollments | t_enrollments_touch | BEFORE UPDATE ROW | `lms.touch_updated_at()` | origin |
| enrollments | vault_cashback | AFTER INSERT/UPDATE ROW | `lms.tg_vault_cashback()` | origin |
| enrollments | vault_referral_reward | AFTER INSERT/UPDATE ROW | `lms.tg_vault_referral_reward()` | origin |
| program_feedback | vault_earn_feedback | AFTER INSERT ROW | `lms.tg_vault_earn_feedback()` | origin |
| resources | t_resources_touch | BEFORE UPDATE ROW | `lms.touch_updated_at()` | origin |
| session_attendance | vault_earn_attendance | AFTER INSERT/DELETE/UPDATE ROW | `lms.tg_vault_earn_attendance()` | origin |
| session_feedback | vault_earn_feedback | AFTER INSERT ROW | `lms.tg_vault_earn_feedback()` | origin |
| session_mentor_overrides | touch_session_mentor_overrides | BEFORE UPDATE ROW | `lms.touch_updated_at()` | origin |
| session_submissions | submission_review_guard | BEFORE INSERT/UPDATE ROW | `lms.tg_submission_review_guard()` | origin |
| session_submissions | vault_earn_project | AFTER INSERT/UPDATE ROW | `lms.tg_vault_earn_project()` | origin |
| sessions | t_sessions_normalize_mode | BEFORE INSERT/UPDATE ROW | `lms.normalize_session_mode()` | origin |
| sessions | t_sessions_touch | BEFORE UPDATE ROW | `lms.touch_updated_at()` | origin |
| student_profiles | profile_referral_guard | BEFORE INSERT/UPDATE ROW | `lms.tg_profile_referral_guard()` | origin |
| student_profiles | t_dob_guard | BEFORE UPDATE ROW | `lms.tg_dob_guard()` | origin |
| student_profiles | t_student_profiles_touch | BEFORE UPDATE ROW | `lms.touch_updated_at()` | origin |
| student_profiles | t_vault_welcome | AFTER INSERT ROW | `lms.t_vault_welcome_on_profile()` | origin |
| support_tickets | support_tickets_updated | BEFORE UPDATE ROW | `lms.touch_updated_at()` | origin |
| ticket_messages | ticket_messages_touch | AFTER INSERT ROW | `lms.tg_ticket_touch()` | origin |
| vault_ledger | vault_ledger_immutable | BEFORE DELETE/UPDATE ROW | `lms.tg_vault_ledger_immutable()` | origin |
| vault_lots | vault_lot_notify | AFTER INSERT/UPDATE ROW | `lms.tg_vault_lot_notify()` | origin |
| vault_review_flags | vault_flag_auto_resolve | BEFORE INSERT ROW | `lms.vault_flag_auto_resolve()` | origin |
| whatsapp_blocklist_archive | whatsapp_blocklist_updated | BEFORE UPDATE ROW | `set_updated_at()` | origin |
| whatsapp_templates | t_whatsapp_templates_touch | BEFORE UPDATE ROW | `lms.touch_updated_at()` | origin |
| whatsapp_templates | validate_whatsapp_recipient_config | BEFORE INSERT/UPDATE ROW | `lms.validate_whatsapp_recipient_config()` | origin |

## portal (25 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| app_settings | app_settings_set_updated_at | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| application_installments | update_application_installments_updated_at | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| camp_leads | camp_leads_touch | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| captured_leads | captured_leads_set_updated_at | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| captured_leads | trg_default_utm | BEFORE INSERT/UPDATE ROW | `portal.tg_default_utm()` | origin |
| captured_leads | trg_normalize_phones | BEFORE INSERT/UPDATE ROW | `portal.tg_normalize_captured_lead_phones()` | origin |
| email_reminder_settings | trg_email_reminder_settings_updated | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| email_templates | trg_email_templates_updated | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| form_templates | update_form_templates_updated_at | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| legacy_form_templates | lwl_custom_form_align_history | AFTER UPDATE ROW | `portal.custom_form_align_history()` | origin |
| legacy_form_templates | legacy_form_templates_touch | BEFORE UPDATE ROW | `lwl_form_touch_updated_at()` | origin |
| masterclass_form_templates | update_masterclass_form_templates_updated_at | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| mentors | update_mentors_updated_at | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| payment_links | payment_links_updated_at | BEFORE UPDATE ROW | `portal.payment_links_touch()` | origin |
| payment_links | trg_payment_links_cohort_guard | BEFORE INSERT/UPDATE ROW | `portal.payment_links_cohort_guard()` | origin |
| payment_links | trg_payment_links_guard | BEFORE INSERT/UPDATE ROW | `portal.payment_links_guard()` | origin |
| program_attachments | trg_program_attachments_updated | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| program_email_settings | trg_program_email_settings_updated | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| school_program_inquiries | trg_normalize_phones | BEFORE INSERT/UPDATE ROW | `portal.tg_normalize_school_inquiry_phones()` | origin |
| school_program_inquiries | update_school_inquiries_updated_at | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| user_profiles | sync_profile_changes | BEFORE UPDATE ROW | `portal.sync_profile_to_applications()` | origin |
| user_profiles | trg_normalize_phones | BEFORE INSERT/UPDATE ROW | `portal.tg_normalize_user_profile_phones()` | origin |
| user_profiles | trg_prevent_profile_privilege_escalation | BEFORE UPDATE ROW | `portal.prevent_profile_privilege_escalation()` | origin |
| user_profiles | update_user_profiles_updated_at | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |
| whatsapp_drip_sequences | update_whatsapp_drip_sequences_updated_at | BEFORE UPDATE ROW | `portal.update_updated_at_column()` | origin |

## fellowship (3 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| all_cohorts | all_cohorts_dml_trg | INSTEAD OF INSERT/DELETE/UPDATE ROW | `fellowship.all_cohorts_dml()` | origin |
| interest_signups | onward_interest_phone | BEFORE INSERT/UPDATE ROW | `fellowship.tg_prefix_phone_interest()` | origin |
| profiles | profiles_set_updated_at | BEFORE UPDATE ROW | `fellowship.set_updated_at()` | origin |

## public (45 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| all_cohorts | script sync all cohorts | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-type":"application/json"}', '{}', '5000')` | origin |
| all_cohorts | t_all_cohorts_touch | BEFORE UPDATE ROW | `touch_updated_at()` | origin |
| all_masterclasses | t_all_masterclasses_touch | BEFORE UPDATE ROW | `touch_updated_at()` | origin |
| all_mentors | t_all_mentors_touch | BEFORE UPDATE ROW | `touch_updated_at()` | origin |
| all_partners | t_all_partners_touch | BEFORE UPDATE ROW | `touch_updated_at()` | origin |
| all_programs | script sync all programs | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-type":"application/json"}', '{}', '5000')` | origin |
| all_programs | t_all_programs_touch | BEFORE UPDATE ROW | `touch_updated_at()` | origin |
| premed_masterclass_regs | doctors_mcreg_category | BEFORE INSERT/UPDATE ROW | `premed.enforce_masterclass_registration_category()` | origin |
| premed_masterclass_regs | doctors_mcreg_dedupe | BEFORE INSERT ROW | `premed.dedupe_masterclass_registration()` | origin |
| premed_masterclass_regs | doctors_mcreg_normalize | BEFORE INSERT/UPDATE ROW | `premed.tg_normalize_masterclass_phones()` | origin |
| premed_masterclass_regs | doctors_mcreg_utm | BEFORE INSERT/UPDATE ROW | `premed.fill_default_utm()` | origin |
| premed_masterclass_regs | script sync premed masterclass | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-type":"application/json"}', '{}', '30000')` | origin |
| finedu_masterclass_regs | script sync finedu mast | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-type":"application/json"}', '{}', '5000')` | origin |
| finedu_masterclass_regs | trg_finedu_masterclass_regs_touch | BEFORE UPDATE ROW | `touch_updated_at()` | origin |
| prelaw_masterclass_regs | script sync lawyer mast | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-type":"application/json"}', '{}', '5000')` | origin |
| applications | lms_referral_application | AFTER INSERT/UPDATE ROW | `lms.tg_referral_application()` | origin |
| applications | script sync group apps | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-type":"application/json"}', '{}', '5000')` | origin |
| applications | trg_aa_normalize_formats | BEFORE INSERT/UPDATE ROW | `portal.tg_lwl_normalize_formats()` | origin |
| applications | trg_apply_internal_tag | BEFORE INSERT/UPDATE ROW | `portal.tg_apply_internal_tag()` | origin |
| applications | trg_default_utm | BEFORE INSERT/UPDATE ROW | `portal.tg_default_utm()` | origin |
| applications | trg_applications_set_updated_at | BEFORE UPDATE ROW | `set_updated_at()` | origin |
| applications | trg_lwl_shared_student_id | BEFORE INSERT ROW | `tg_lwl_shared_student_id()` | origin |
| applications | trg_normalize_phones | BEFORE INSERT/UPDATE ROW | `portal.tg_normalize_lwl_application_phones()` | origin |
| applications | trg_payment_links_count_use | AFTER UPDATE ROW | `portal.payment_links_count_use()` | origin |
| applications | trg_purge_leads_for_application | AFTER INSERT ROW | `portal.tg_purge_leads_for_application()` | origin |
| applications | trg_wati_payment_received | AFTER INSERT/UPDATE ROW | `portal.notify_wati_payment_received()` | origin |
| form_responses | lwl_custom_response_current_identity | BEFORE INSERT ROW | `portal.custom_response_current_identity()` | origin |
| form_responses | form_responses_touch | BEFORE UPDATE ROW | `lwl_form_touch_updated_at()` | origin |
| form_responses | script sync resp sheets | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-type":"application/json"}', '{}', '5000')` | origin |
| masterclass_regs | script masterclass response sheets sync | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-type":"application/json"}', '{}', '5000')` | origin |
| fellowship_masterclass_regs | onward_mast_phone | BEFORE INSERT/UPDATE ROW | `fellowship.tg_prefix_phone_masterclass()` | origin |
| fellowship_masterclass_regs | script sync fellowship mast | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-type":"application/json"}', '{}', '5000')` | origin |
| promo_codes | trg_promo_codes_touch | BEFORE UPDATE ROW | `promo_touch_updated_at()` | origin |
| promo_redemptions | lms_vault_promo_redeemed | AFTER INSERT ROW | `tg_lms_vault_promo_redeemed()` | origin |
| promo_redemptions | trg_promo_redemption_count | AFTER INSERT/DELETE ROW | `promo_sync_redemption_count()` | origin |
| razorpay_payments | trg_razorpay_payments_touch | BEFORE INSERT/UPDATE ROW | `razorpay_payments_touch_synced_at()` | origin |
| testprep_masterclass_regs | trg_enforce_utm_defaults | BEFORE INSERT/UPDATE ROW | `testprep.enforce_utm_defaults()` | origin |
| testprep_masterclass_regs | trg_normalize_masterclass_phones | BEFORE INSERT/UPDATE ROW | `testprep.normalize_masterclass_phones()` | origin |
| testprep_masterclass_regs | trg_touch_masterclass | BEFORE UPDATE ROW | `testprep.touch_updated_at()` | origin |
| research_masterclass_regs | script sync research mast | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-type":"application/json"}', '{}', '5000')` | origin |
| research_masterclass_regs | sirluther_regs_default_utm | BEFORE INSERT/UPDATE ROW | `research.tg_default_utm()` | origin |
| research_masterclass_regs | sirluther_regs_safe_phone | BEFORE INSERT/UPDATE ROW | `research.tg_sheet_safe_phone()` | origin |
| leaders_applications | script sync stud apps | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-Type":"application/json"}', '{}', '5000')` | origin |
| week_without_walls_inquiries | onward_wwi_phone | BEFORE INSERT/UPDATE ROW | `fellowship.tg_prefix_phone_wwi()` | origin |
| week_without_walls_inquiries | script sync week walls | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-type":"application/json"}', '{}', '5000')` | origin |

## realtime (1 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| subscription | tr_check_filters | BEFORE INSERT/UPDATE ROW | `realtime.subscription_check_filters()` | origin |

## testprep (15 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| captured_leads | captured_leads_touch | BEFORE UPDATE ROW | `testprep.touch_updated_at()` | origin |
| drill_responses | on_drill_response_created | AFTER INSERT ROW | `testprep.update_trap_stats()` | origin |
| drill_sessions | on_session_end_snapshot_potential | AFTER UPDATE ROW | `testprep.snapshot_student_potential()` | origin |
| email_templates | email_templates_touch | BEFORE UPDATE ROW | `testprep.touch_updated_at()` | origin |
| leads | leads_touch | BEFORE UPDATE ROW | `testprep.touch_updated_at()` | origin |
| program_cohorts | program_cohorts_del | INSTEAD OF DELETE ROW | `testprep.program_cohorts_write()` | origin |
| program_cohorts | program_cohorts_ins | INSTEAD OF INSERT ROW | `testprep.program_cohorts_write()` | origin |
| program_cohorts | program_cohorts_upd | INSTEAD OF UPDATE ROW | `testprep.program_cohorts_write()` | origin |
| program_email_settings | program_email_settings_touch | BEFORE UPDATE ROW | `testprep.touch_updated_at()` | origin |
| school_inquiries | trg_normalize_school_inquiry_phone | BEFORE INSERT/UPDATE ROW | `testprep.normalize_school_inquiry_phone()` | origin |
| school_inquiries | trg_touch_school_inquiries | BEFORE UPDATE ROW | `testprep.touch_updated_at()` | origin |
| user_roles | trg_user_roles_touch | BEFORE UPDATE ROW | `testprep.touch_updated_at()` | origin |
| whatsapp_custom_variables | wa_vars_touch | BEFORE UPDATE ROW | `testprep.touch_updated_at()` | origin |
| whatsapp_sends | trg_normalize_whatsapp_send_phone | BEFORE INSERT/UPDATE ROW | `testprep.normalize_whatsapp_send_phone()` | origin |
| whatsapp_templates | wa_templates_touch | BEFORE UPDATE ROW | `testprep.touch_updated_at()` | origin |

## research (16 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| captured_leads | captured_leads_set_updated_at | BEFORE UPDATE ROW | `research.update_updated_at_column()` | origin |
| email_templates | trg_email_templates_updated_at | BEFORE UPDATE ROW | `research.update_updated_at_column()` | origin |
| masterclass_events | update_masterclass_events_updated_at | BEFORE UPDATE ROW | `research.update_updated_at_column()` | origin |
| mentor_questions | sheet_safe_phone_biu | BEFORE INSERT/UPDATE ROW | `research.tg_sheet_safe_phone()` | origin |
| mentor_questions | update_mentor_questions_updated_at | BEFORE UPDATE ROW | `research.update_updated_at_column()` | origin |
| profiles | update_profiles_updated_at | BEFORE UPDATE ROW | `research.update_updated_at_column()` | origin |
| program_email_settings | trg_program_email_settings_updated_at | BEFORE UPDATE ROW | `research.update_updated_at_column()` | origin |
| questionnaire_responses | script sync research qr resp | AFTER INSERT/DELETE/UPDATE ROW | `supabase_functions.http_request('https://script.google.com/macros/s/<redacted-deployment-id>/exec', 'POST', '{"Content-type":"application/json"}', '{}', '5000')` | origin |
| research_projects | update_research_projects_updated_at | BEFORE UPDATE ROW | `research.update_updated_at_column()` | origin |
| school_partners | school_partners_updated_at | BEFORE UPDATE ROW | `research.update_updated_at_column()` | origin |
| school_teacher_contacts | set_school_teacher_contacts_updated_at | BEFORE UPDATE ROW | `research.update_updated_at_column()` | origin |
| site_leads | sheet_safe_phone_biu | BEFORE INSERT/UPDATE ROW | `research.tg_sheet_safe_phone()` | origin |
| student_outcomes | update_student_outcomes_updated_at | BEFORE UPDATE ROW | `research.update_updated_at_column()` | origin |
| teacher_feedback | update_teacher_feedback_updated_at | BEFORE UPDATE ROW | `research.update_updated_at_column()` | origin |
| teacher_students | trg_lock_teacher_students_student_update | BEFORE UPDATE ROW | `research.lock_teacher_students_student_update()` | origin |
| whatsapp_settings | whatsapp_settings_updated_at | BEFORE UPDATE ROW | `research.update_updated_at_column()` | origin |

## storage (11 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| buckets | enforce_bucket_name_length_trigger | BEFORE INSERT/UPDATE ROW | `storage.enforce_bucket_name_length()` | origin |
| buckets | enforce_versioning_status_transition | BEFORE UPDATE ROW | `storage.enforce_versioning_status_transition()` | origin |
| buckets | protect_active_lifecycle_states | BEFORE DELETE ROW | `storage.protect_active_lifecycle_states()` | origin |
| buckets | protect_bucket_control_insert | BEFORE INSERT ROW | `storage.protect_bucket_control_columns('service_role')` | origin |
| buckets | protect_bucket_control_update | BEFORE UPDATE ROW | `storage.protect_bucket_control_columns()` | origin |
| buckets | protect_bucket_control_update_role | AFTER UPDATE ROW | `storage.enforce_bucket_lifecycle_service_role('service_role')` | origin |
| buckets | protect_buckets_delete | BEFORE DELETE STATEMENT | `storage.protect_delete()` | origin |
| buckets | wake_lifecycle_on_policy_write | AFTER INSERT/UPDATE ROW | `storage.wake_lifecycle_state()` | origin |
| buckets | wake_lifecycle_on_versioning_enable | AFTER UPDATE ROW WHEN(...) | `storage.wake_lifecycle_state()` | origin |
| objects | protect_objects_delete | BEFORE DELETE STATEMENT | `storage.protect_delete()` | origin |
| objects | update_objects_updated_at | BEFORE UPDATE ROW | `storage.update_updated_at_column()` | origin |

## leaders (5 triggers)

| table | trigger | timing / events / level | function (from pg_get_triggerdef) | enabled |
|---|---|---|---|---|
| level_progress | touch_progress | BEFORE UPDATE ROW | `leaders.touch_updated_at()` | origin |
| levels | touch_levels | BEFORE UPDATE ROW | `leaders.touch_updated_at()` | origin |
| masterclass_regs | masterclass_regs_updated_at | BEFORE UPDATE ROW | `leaders.set_updated_at()` | origin |
| profiles | touch_profiles | BEFORE UPDATE ROW | `leaders.touch_updated_at()` | origin |
| sl_task_progress | sl_task_progress_touch | BEFORE UPDATE ROW | `leaders.sl_touch()` | origin |

## Observations

- **auth.users: no triggers.** There is no `auth` section above because no trigger exists on any `auth.*` table. `public.handle_new_user()` is defined as a trigger function but is not bound to any trigger; per-platform profile rows are created by application / edge-function code (e.g. `premed.ensure_user_profile()` RPC, Affiliate `apply_partner`), not by a DB trigger on sign-up.
- Totals: **222 triggers** across 15 schemas (cron 1, premed 36, finance 14, finedu 6, ops 2, prelaw 3, lms 39, portal 25, fellowship 3, public 45, realtime 1, testprep 15, research 16, storage 11, leaders 5). All are enabled (`origin`); none are disabled.
- **Google Sheets webhooks**: 14 `supabase_functions.http_request(...)` triggers (named `script sync ...`) fire AFTER INSERT/UPDATE/DELETE on `public.all_cohorts`, `public.all_programs`, `public.premed_masterclass_regs`, `public.finedu_masterclass_regs`, `public.prelaw_masterclass_regs`, `public.applications`, `public.form_responses`, `public.masterclass_regs`, `public.fellowship_masterclass_regs`, `public.research_masterclass_regs`, `public.leaders_applications`, `public.week_without_walls_inquiries` and `research.questionnaire_responses`, POSTing the row payload to Google Apps Script web-app endpoints (`https://script.google.com/macros/s/<redacted-deployment-id>/exec`, timeouts 5000-30000 ms). These are the Supabase "Database Webhooks"; their request log lives in `supabase_functions.hooks` (115k rows, 16 MB).
- `public.applications` is the most heavily wired table: 11 BEFORE/AFTER triggers from `portal.*` (format normalisation, UTM defaults, internal tag, phone normalisation, payment-link use counting, lead purge, WATI payment-received notification), `lms.tg_referral_application` (referral attribution into `lms.referral_*`), `public.tg_lwl_shared_student_id` (shared student id allocation) and the Sheets webhook.
- INSTEAD OF triggers make these views writable: `premed.masterclass_sessions` (`premed.tg_masterclass_sessions_write`), `lms.all_mentors` (`lms.tg_all_mentors_view_write`), `fellowship.all_cohorts` (`fellowship.all_cohorts_dml`), `testprep.program_cohorts` (`testprep.program_cohorts_write`). Each writes through to the central `public.all_*` tables.
- Immutability guards: `lms.enrollment_events` (BEFORE UPDATE/DELETE row + BEFORE TRUNCATE statement) and `lms.vault_ledger` (BEFORE UPDATE/DELETE) raise via `lms.tg_enrollment_events_immutable` / `lms.tg_vault_ledger_immutable`.
- Privilege-escalation guards on profile tables: `premed.user_profiles` (`prevent_role_escalation`), `portal.user_profiles` (`prevent_profile_privilege_escalation`, plus `sync_profile_to_applications`), `lms.student_profiles` (`tg_profile_referral_guard`, `tg_dob_guard`).
- Vault (LMS points) economy is trigger-driven: `lms.enrollments` (cashback, referral reward, ensure community channel), `lms.session_attendance`, `lms.session_feedback`, `lms.program_feedback`, `lms.session_submissions`, `lms.cohort_completions`, `lms.vault_lots` (notify), `public.promo_redemptions` (`tg_lms_vault_promo_redeemed`).
- Finance: `finance.codes_dropdown`, `finance.expense_codes`, `finance.program_codes` have statement-level AFTER triggers (`finance.request_code_sheet_sync`) that mark `finance.code_sheet_sync_state` dirty for the Sheet sync worker.

## Event triggers (pg_event_trigger)

| event trigger | event | tags | function |
|---|---|---|---|
| ensure_rls | ddl_command_end | CREATE TABLE, CREATE TABLE AS, SELECT INTO | public.rls_auto_enable (auto-enables RLS on every new table) |
| ensure_finlit_views_security_invoker | ddl_command_end | CREATE VIEW | public.enforce_finlit_view_security_invoker |
| issue_graphql_placeholder | sql_drop | DROP EXTENSION | extensions.set_graphql_placeholder (Supabase platform) |
| issue_pg_cron_access | ddl_command_end | CREATE EXTENSION | extensions.grant_pg_cron_access (platform) |
| issue_pg_graphql_access | ddl_command_end | CREATE FUNCTION | extensions.grant_pg_graphql_access (platform) |
| issue_pg_net_access | ddl_command_end | CREATE EXTENSION | extensions.grant_pg_net_access (platform) |
| pgrst_ddl_watch | ddl_command_end | (all) | extensions.pgrst_ddl_watch (PostgREST schema-cache reload) |
| pgrst_drop_watch | sql_drop | (all) | extensions.pgrst_drop_watch |

The two custom event triggers explain why every table in the estate shows `rls = yes` and why finedu views carry `security_invoker=true`.

