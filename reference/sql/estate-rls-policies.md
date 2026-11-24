# Shared estate RLS policies

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

## affiliate (2 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| resources | resources_no_direct_browser_access | ALL | anon,authenticated | PERMISSIVE | false | false |
| school_achievements | school_achievements_no_direct_browser_access | ALL | anon,authenticated | PERMISSIVE | false | false |

## cron (2 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| job | cron_job_policy | ALL | public | PERMISSIVE | (username = CURRENT_USER) |  |
| job_run_details | cron_job_run_details_policy | ALL | public | PERMISSIVE | (username = CURRENT_USER) |  |

## premed (90 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| aeo_keywords | aeo_keywords_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| aeo_outputs | aeo_outputs_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| aeo_outputs | aeo_outputs_read_approved | SELECT | anon,authenticated | PERMISSIVE | (status = 'approved'::text) |  |
| aeo_page_optimizations | aeo_page_optimizations_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| aeo_page_optimizations | aeo_page_optimizations_read_applied | SELECT | anon,authenticated | PERMISSIVE | (meta_status = 'applied'::text) |  |
| aeo_pages | aeo_pages_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| aeo_pages | aeo_pages_read_approved | SELECT | anon,authenticated | PERMISSIVE | (status = 'approved'::text) |  |
| aeo_prompts | aeo_prompts_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| aeo_settings | aeo_settings_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| app_settings | app_settings_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| captured_leads | captured_leads_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| challenge_quiz_attempts | Admins read quiz attempts | SELECT | authenticated | PERMISSIVE | has_role(auth.uid(), 'admin'::app_role) |  |
| cohort_settings | cohort_settings_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| cohort_settings | cohort_settings_read | SELECT | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'mentor'::premed.app_role) |  |
| cohort_settings | cohort_settings_read_active | SELECT | anon,authenticated | PERMISSIVE | (is_active = true) |  |
| concierge_ai_usage | concierge_ai_usage_admin_read | SELECT | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) |  |
| concierge_conversations | concierge_conversations_staff_read | SELECT | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) |  |
| concierge_conversations | concierge_conversations_staff_update | UPDATE | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| concierge_conversations | concierge_conversations_student_insert | INSERT | authenticated | PERMISSIVE |  | ((user_id = auth.uid()) AND (tier = 'student'::text)) |
| concierge_conversations | concierge_conversations_student_read | SELECT | authenticated | PERMISSIVE | ((user_id IS NOT NULL) AND (user_id = auth.uid())) |  |
| concierge_conversations | concierge_conversations_student_update | UPDATE | authenticated | PERMISSIVE | ((user_id = auth.uid()) AND (tier = 'student'::text)) | ((user_id = auth.uid()) AND (tier = 'student'::text)) |
| concierge_faqs | concierge_faqs_admin_delete | DELETE | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) |  |
| concierge_faqs | concierge_faqs_admin_insert | INSERT | authenticated | PERMISSIVE |  | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| concierge_faqs | concierge_faqs_admin_read | SELECT | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) |  |
| concierge_faqs | concierge_faqs_admin_update | UPDATE | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| concierge_faqs | concierge_faqs_public_read | SELECT | anon,authenticated | PERMISSIVE | is_published |  |
| email_templates | email_templates_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| fellowship_projects | fellowship_projects_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| fellowship_projects | fellowship_projects_mentor_read | SELECT | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'mentor'::premed.app_role) |  |
| goal_reviews | goal_reviews_delete | DELETE | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) |  |
| goal_reviews | goal_reviews_update | UPDATE | authenticated | PERMISSIVE | (premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| goal_reviews | goal_reviews_write | INSERT | authenticated | PERMISSIVE |  | (premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| goal_reviews | reviews_read | SELECT | authenticated | PERMISSIVE | (((user_id = auth.uid()) AND is_published) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| intake_raw | intake_raw_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| knowledge_entries | knowledge_admin_delete | DELETE | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) |  |
| knowledge_entries | knowledge_admin_insert | INSERT | authenticated | PERMISSIVE |  | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| knowledge_entries | knowledge_admin_update | UPDATE | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| knowledge_entries | knowledge_read | SELECT | authenticated | PERMISSIVE | (active AND (expires > CURRENT_DATE)) |  |
| learning_assessments | learning_assessments_delete | DELETE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| learning_assessments | learning_assessments_insert | INSERT | authenticated | PERMISSIVE |  | (user_id = auth.uid()) |
| learning_assessments | learning_assessments_read | SELECT | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| learning_assessments | learning_assessments_update | UPDATE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| live_class_rsvps | live_class_rsvps_delete | DELETE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| live_class_rsvps | live_class_rsvps_insert | INSERT | authenticated | PERMISSIVE |  | (user_id = auth.uid()) |
| live_class_rsvps | live_class_rsvps_read | SELECT | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| live_class_rsvps | live_class_rsvps_update | UPDATE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| masterclass_series_overrides | Admins can read series overrides | SELECT | authenticated | PERMISSIVE | (premed.has_role(auth.uid(), 'admin'::premed.app_role) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role)) |  |
| mmi_coach_sessions | mmi_coach_sessions_delete | DELETE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| mmi_coach_sessions | mmi_coach_sessions_insert | INSERT | authenticated | PERMISSIVE |  | (user_id = auth.uid()) |
| mmi_coach_sessions | mmi_coach_sessions_read | SELECT | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| mmi_coach_sessions | mmi_coach_sessions_update | UPDATE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| mmi_practice_attempts | mmi_practice_attempts_delete | DELETE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| mmi_practice_attempts | mmi_practice_attempts_insert | INSERT | authenticated | PERMISSIVE |  | (user_id = auth.uid()) |
| mmi_practice_attempts | mmi_practice_attempts_read | SELECT | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| mmi_practice_attempts | mmi_practice_attempts_update | UPDATE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| parent_reports | parent_reports_delete | DELETE | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) |  |
| parent_reports | parent_reports_read | SELECT | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| parent_reports | parent_reports_update | UPDATE | authenticated | PERMISSIVE | (premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| parent_reports | parent_reports_write | INSERT | authenticated | PERMISSIVE |  | (premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| program_attachments | program_attachments_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| program_email_settings | program_email_settings_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| registration_attempts | Admins can view registration attempts | SELECT | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) |  |
| saved_universities | saved_universities_delete | DELETE | authenticated | PERMISSIVE | ((student_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| saved_universities | saved_universities_insert | INSERT | authenticated | PERMISSIVE |  | (student_id = auth.uid()) |
| saved_universities | saved_universities_read | SELECT | authenticated | PERMISSIVE | ((student_id = auth.uid()) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| saved_universities | saved_universities_update | UPDATE | authenticated | PERMISSIVE | ((student_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) | ((student_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| sent_emails | sent_emails_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| session_attendance | attendance_read | SELECT | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| session_attendance | session_attendance_delete | DELETE | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) |  |
| session_attendance | session_attendance_update | UPDATE | authenticated | PERMISSIVE | (premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| session_attendance | session_attendance_write | INSERT | authenticated | PERMISSIVE |  | (premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| side_effects_queue | side_effects_queue_admin | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| student_goals | student_goals_delete | DELETE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| student_goals | student_goals_insert | INSERT | authenticated | PERMISSIVE |  | (user_id = auth.uid()) |
| student_goals | student_goals_read | SELECT | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| student_goals | student_goals_update | UPDATE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| student_notifications | student_notifications_delete | DELETE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| student_notifications | student_notifications_insert | INSERT | authenticated | PERMISSIVE |  | (user_id = auth.uid()) |
| student_notifications | student_notifications_read | SELECT | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| student_notifications | student_notifications_update | UPDATE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| student_university_applications | student_university_applications_delete | DELETE | authenticated | PERMISSIVE | ((student_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| student_university_applications | student_university_applications_insert | INSERT | authenticated | PERMISSIVE |  | (student_id = auth.uid()) |
| student_university_applications | student_university_applications_read | SELECT | authenticated | PERMISSIVE | ((student_id = auth.uid()) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| student_university_applications | student_university_applications_update | UPDATE | authenticated | PERMISSIVE | ((student_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) | ((student_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| universities | universities_read_auth | SELECT | authenticated | PERMISSIVE | (premed.get_user_role(( SELECT auth.uid() AS uid)) IS NOT NULL) |  |
| user_profiles | up_self_insert | INSERT | authenticated | PERMISSIVE |  | ((user_id = auth.uid()) AND (role = 'student'::premed.app_role)) |
| user_profiles | up_self_read | SELECT | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| user_profiles | up_self_update | UPDATE | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) | (premed.has_role(auth.uid(), 'admin'::premed.app_role) OR ((user_id = auth.uid()) AND (role = ( SELECT up.role    FROM premed.user_profiles up   WHERE (up.user_id = auth.uid()))))) |
| whatsapp_messages | whatsapp_messages_admin_manage | ALL | authenticated | PERMISSIVE | premed.has_role(auth.uid(), 'admin'::premed.app_role) | premed.has_role(auth.uid(), 'admin'::premed.app_role) |
| whatsapp_messages | whatsapp_messages_staff_read | SELECT | authenticated | PERMISSIVE | (premed.has_role(auth.uid(), 'admin'::premed.app_role) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role)) |  |

## finance (3 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| codes_dropdown | codes_dropdown read | SELECT | authenticated | PERMISSIVE | finance.is_finance_user() |  |
| expense_codes | expense_codes read | SELECT | authenticated | PERMISSIVE | is_finance_admin(auth.uid()) |  |
| program_codes | program_codes read | SELECT | authenticated | PERMISSIVE | finance.is_finance_user() |  |

## finedu (24 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| ai_agent_logs | admins view ai agent logs | SELECT | authenticated | PERMISSIVE | finedu.is_admin() |  |
| app_settings | admins all app_settings | ALL | authenticated | PERMISSIVE | finedu.is_admin() | finedu.is_admin() |
| assessment_questions | admins manage assessment questions | ALL | authenticated | PERMISSIVE | finedu.is_admin() | finedu.is_admin() |
| assessment_questions | anyone reads active assessment questions | SELECT | public | PERMISSIVE | (active = true) |  |
| concierge_leads | admins delete concierge leads | DELETE | authenticated | PERMISSIVE | finedu.is_admin() |  |
| concierge_leads | admins update concierge leads | UPDATE | authenticated | PERMISSIVE | finedu.is_admin() | finedu.is_admin() |
| concierge_leads | admins view concierge leads | SELECT | authenticated | PERMISSIVE | finedu.is_admin() |  |
| concierge_leads | anyone submits concierge lead | INSERT | anon,authenticated | PERMISSIVE |  | (((parent_email IS NULL) OR ((char_length(parent_email) >= 3) AND (char_length(parent_email) <= 255))) AND ((parent_phone IS NULL) OR ((char_length(parent_phone) >= 5) AND (char_length(parent_phone) <= 40))) AND (status = 'new'::finedu.lead_status) AND (score IS NULL) AND (score_reason IS NULL) AND (voucher_code IS NULL) AND (voucher_amount_usd IS NULL) AND (voucher_expires_at IS NULL) AND (voucher_redeemed_at IS NULL) AND (assessment_completed_at IS NULL) AND (assessment_strength_label IS NULL) AND (notes IS NULL)) |
| email_attachments | admins manage email attachments | ALL | authenticated | PERMISSIVE | finedu.is_admin() | finedu.is_admin() |
| email_send_log | admins view send log | SELECT | authenticated | PERMISSIVE | finedu.is_admin() |  |
| email_templates | admins manage email templates | ALL | authenticated | PERMISSIVE | finedu.is_admin() | finedu.is_admin() |
| email_unsubscribe_tokens | admins view unsubscribe tokens | SELECT | authenticated | PERMISSIVE | finedu.is_admin() |  |
| page_events | admins view events | SELECT | authenticated | PERMISSIVE | finedu.is_admin() |  |
| page_events | anyone records event | INSERT | anon,authenticated | PERMISSIVE |  | ((char_length(COALESCE(event, ''::text)) >= 1) AND (char_length(COALESCE(event, ''::text)) <= 200)) |
| page_visits | admins view visits | SELECT | authenticated | PERMISSIVE | finedu.is_admin() |  |
| page_visits | anyone records visit | INSERT | anon,authenticated | PERMISSIVE |  | ((char_length(COALESCE(path, ''::text)) >= 1) AND (char_length(COALESCE(path, ''::text)) <= 2000)) |
| program_email_settings | admins manage program email settings | ALL | authenticated | PERMISSIVE | finedu.is_admin() | finedu.is_admin() |
| school_proposals | admins delete proposals | DELETE | authenticated | PERMISSIVE | finedu.is_admin() |  |
| school_proposals | admins insert proposals | INSERT | authenticated | PERMISSIVE |  | finedu.is_admin() |
| school_proposals | admins update proposals | UPDATE | authenticated | PERMISSIVE | finedu.is_admin() | finedu.is_admin() |
| school_proposals | admins view proposals | SELECT | authenticated | PERMISSIVE | finedu.is_admin() |  |
| suppressed_emails | admins view suppressed emails | SELECT | authenticated | PERMISSIVE | finedu.is_admin() |  |
| user_roles | admins manage roles | ALL | authenticated | PERMISSIVE | finedu.is_admin() | finedu.is_admin() |
| whatsapp_messages | admins view whatsapp messages | SELECT | authenticated | PERMISSIVE | finedu.is_admin() |  |

## ops (5 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| contact_suppressions | service_only_suppressions | ALL | service_role | PERMISSIVE | true | true |
| profiles | profiles_self_insert | INSERT | authenticated | PERMISSIVE |  | (id = auth.uid()) |
| profiles | profiles_self_read | SELECT | authenticated | PERMISSIVE | ((id = auth.uid()) OR ops.is_admin()) |  |
| profiles | profiles_self_write | UPDATE | authenticated | PERMISSIVE | ((id = auth.uid()) OR ops.is_admin()) | ((id = auth.uid()) OR ops.is_admin()) |
| user_roles | user_roles_self_read | SELECT | authenticated | PERMISSIVE | ((user_id = auth.uid()) OR ops.is_admin()) |  |

## prelaw (6 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| email_logs | Admins read logs | SELECT | authenticated | PERMISSIVE | prelaw.has_role(auth.uid(), 'admin'::prelaw.app_role) |  |
| email_templates | Admins manage templates | ALL | authenticated | PERMISSIVE | prelaw.has_role(auth.uid(), 'admin'::prelaw.app_role) | prelaw.has_role(auth.uid(), 'admin'::prelaw.app_role) |
| unmatched_payments | Admins read unmatched payments | SELECT | authenticated | PERMISSIVE | prelaw.has_role(auth.uid(), 'admin'::prelaw.app_role) |  |
| user_roles | Users see own roles | SELECT | authenticated | PERMISSIVE | (auth.uid() = user_id) |  |
| whatsapp_messages | Admins read whatsapp_messages | SELECT | authenticated | PERMISSIVE | prelaw.has_role(auth.uid(), 'admin'::prelaw.app_role) |  |
| whatsapp_settings | Admins manage whatsapp_settings | ALL | authenticated | PERMISSIVE | prelaw.has_role(auth.uid(), 'admin'::prelaw.app_role) | prelaw.has_role(auth.uid(), 'admin'::prelaw.app_role) |

## lms (124 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| admin_profiles | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| birthday_grants | bg_admin | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| birthday_grants | bg_own | SELECT | authenticated | PERMISSIVE | (student_user_id = auth.uid()) |  |
| certificate_templates | cert_tpl_admin | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| certificates | certificates_admin_read | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| cohort_completions | cohort_completions_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| cohort_completions | cohort_completions_read | SELECT | authenticated | PERMISSIVE | (lms.has_role(( SELECT auth.uid() AS uid), 'admin'::text) OR lms.is_cohort_mentor(cohort_id) OR lms.is_cohort_student(cohort_id)) |  |
| cohort_group_members | cgm_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| cohort_group_members | cgm_read_own | SELECT | authenticated | PERMISSIVE | (student_user_id = auth.uid()) |  |
| cohort_group_members | mentor_group_members | SELECT | authenticated | PERMISSIVE | lms.is_cohort_mentor(cohort_id) |  |
| cohort_group_settings | cgs_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| cohort_group_settings | cgs_read | SELECT | authenticated | PERMISSIVE | (lms.has_role(( SELECT auth.uid() AS uid), 'admin'::text) OR lms.is_cohort_mentor(cohort_id) OR lms.is_my_catalog_mentor(default_mentor_id) OR lms.is_cohort_student(cohort_id)) |  |
| cohort_groups | cohort_groups_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| cohort_groups | cohort_groups_read | SELECT | authenticated | PERMISSIVE | (lms.has_role(( SELECT auth.uid() AS uid), 'admin'::text) OR lms.is_cohort_mentor(cohort_id) OR lms.is_my_catalog_mentor(mentor_id) OR lms.is_cohort_student(cohort_id)) |  |
| cohort_pathways | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| cohort_pathways | pathways_readable | SELECT | authenticated | PERMISSIVE | (lms.has_role(( SELECT auth.uid() AS uid), 'admin'::text) OR lms.is_cohort_mentor(cohort_id) OR lms.is_cohort_student(cohort_id)) |  |
| cohort_section_members | secm_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| cohort_section_members | secm_mentor_read | SELECT | authenticated | PERMISSIVE | lms.is_cohort_mentor(cohort_id) |  |
| cohort_section_members | secm_read_own | SELECT | authenticated | PERMISSIVE | (student_user_id = auth.uid()) |  |
| cohort_sections | sec_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| cohort_sections | sec_mentor_read | SELECT | authenticated | PERMISSIVE | lms.is_cohort_mentor(cohort_id) |  |
| community_ai_reviews | ai_reviews_admin_read | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| community_channels | channels_select | SELECT | authenticated | PERMISSIVE | lms.can_view_channel(auth.uid(), id) |  |
| community_mentions | mentions_own_read | SELECT | authenticated | PERMISSIVE | ((mentioned_user_id = auth.uid()) AND (EXISTS ( SELECT 1    FROM lms.community_messages m   WHERE ((m.id = community_mentions.message_id) AND (m.deleted_at IS NULL))))) |  |
| community_mentions | mentions_own_update | UPDATE | authenticated | PERMISSIVE | (mentioned_user_id = auth.uid()) | (mentioned_user_id = auth.uid()) |
| community_messages | messages_insert | INSERT | authenticated | PERMISSIVE |  | ((sender_user_id = auth.uid()) AND lms.can_view_channel(auth.uid(), channel_id)) |
| community_messages | messages_select | SELECT | authenticated | PERMISSIVE | lms.can_view_channel(auth.uid(), channel_id) |  |
| community_messages | messages_update_admin | UPDATE | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| community_messages | messages_update_own | UPDATE | authenticated | PERMISSIVE | (sender_user_id = auth.uid()) | (sender_user_id = auth.uid()) |
| community_moderation_log | moderation_log_admin_read | SELECT | authenticated | PERMISSIVE | (EXISTS ( SELECT 1    FROM lms.admin_profiles a   WHERE (a.user_id = auth.uid()))) |  |
| community_reads | reads_own | ALL | authenticated | PERMISSIVE | (user_id = auth.uid()) | ((user_id = auth.uid()) AND lms.can_view_channel(auth.uid(), channel_id)) |
| community_session_reminder_log | community_reminder_service_only | ALL | service_role | PERMISSIVE | true | true |
| email_templates | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| enrollment_errors | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| enrollment_events | enrollment_events_admin_select | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| enrollment_run_items | admins_read | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| enrollment_runs | admins_read | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| enrollment_settings | enrollment_settings_admin_select | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| enrollment_sync_items | sync_items_admin_select | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| enrollment_sync_runs | sync_runs_admin_select | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| enrollments | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| enrollments | own_enrollments | SELECT | public | PERMISSIVE | ((auth.uid() = student_user_id) AND (cohort_id <> 'bd988fa4-47a6-4c69-907e-9169a5e5fa98'::uuid)) |  |
| feedback_nudge_log | admins read feedback nudge log | SELECT | authenticated | PERMISSIVE | (EXISTS ( SELECT 1    FROM lms.admin_profiles a   WHERE (a.user_id = auth.uid()))) |  |
| grade_yearly_runs | grade_runs_admin_select | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| help_messages | help_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| help_messages | help_own | ALL | public | PERMISSIVE | (student_user_id = auth.uid()) | (student_user_id = auth.uid()) |
| login_rollout_queue | rollout_queue_admin_select | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| login_rollout_settings | rollout_settings_admin_select | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| mentor_accounts | mentor_accounts_admin | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| mentor_accounts | mentor_accounts_own | SELECT | authenticated | PERMISSIVE | (user_id = auth.uid()) |  |
| mentor_feedback | mentor_feedback_admin | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| mentor_feedback | mentor_feedback_own | SELECT | authenticated | PERMISSIVE | ((student_user_id = auth.uid()) AND (EXISTS ( SELECT 1    FROM lms.enrollments e   WHERE ((e.cohort_id = mentor_feedback.cohort_id) AND (e.student_user_id = auth.uid()) AND (e.access_status = 'active'::text))))) |  |
| mentor_settings | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| mentor_settings | mentor_settings_read | SELECT | authenticated | PERMISSIVE | (is_listed = true) |  |
| mentor_settings | mentor_settings_staff_read | SELECT | authenticated | PERMISSIVE | (lms.has_role(( SELECT auth.uid() AS uid), 'admin'::text) OR lms.is_my_catalog_mentor(mentor_id)) |  |
| notification_log | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| notification_recipients | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| notification_recipients | own_notifications | SELECT | public | PERMISSIVE | (auth.uid() = student_user_id) |  |
| notification_recipients | own_notifications_update | UPDATE | authenticated | PERMISSIVE | (auth.uid() = student_user_id) | (auth.uid() = student_user_id) |
| notifications | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| notifications | mentor_announcements | SELECT | authenticated | PERMISSIVE | ((lms.my_mentor_id() IS NOT NULL) AND (lms.is_cohort_mentor(cohort_id) OR ((cohort_id IS NULL) AND (program_id IS NULL) AND (partner_id IS NULL) AND (pathway_id IS NULL)))) |  |
| notifications | notif_recipient_read | SELECT | authenticated | PERMISSIVE | ((EXISTS ( SELECT 1    FROM lms.notification_recipients nr   WHERE ((nr.notification_id = notifications.id) AND (nr.student_user_id = auth.uid())))) AND ((cohort_id IS NULL) OR (EXISTS ( SELECT 1    FROM lms.enrollments e   WHERE ((e.student_user_id = auth.uid()) AND (e.cohort_id = notifications.cohort_id) AND (e.access_status = 'active'::text)))))) |  |
| partner_checkout_config | pcc_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| program_feedback | program_feedback_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| program_feedback | program_feedback_insert_own | INSERT | public | PERMISSIVE |  | ((student_user_id = auth.uid()) AND (EXISTS ( SELECT 1    FROM lms.enrollments e   WHERE ((e.cohort_id = program_feedback.cohort_id) AND (e.student_user_id = auth.uid())))) AND (EXISTS ( SELECT 1    FROM lms.cohort_completions c   WHERE ((c.cohort_id = program_feedback.cohort_id) AND c.is_complete)))) |
| program_feedback | program_feedback_read_own | SELECT | public | PERMISSIVE | (student_user_id = auth.uid()) |  |
| referral_invites | referral_invites_own | SELECT | authenticated | PERMISSIVE | ((inviter_user_id = auth.uid()) OR ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role)) |  |
| referral_settings | referral_settings_admin_select | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| registration_attempts | registration_attempts_admin_select | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| resources | enrolled_resources | SELECT | public | PERMISSIVE | (EXISTS ( SELECT 1    FROM lms.enrollments e   WHERE ((e.student_user_id = auth.uid()) AND (e.cohort_id = resources.cohort_id) AND (e.access_status = 'active'::text) AND ((resources.pathway_id IS NULL) OR (resources.pathway_id = e.pathway_id))))) |  |
| resources | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| resources | mentor_resources | SELECT | authenticated | PERMISSIVE | lms.is_cohort_mentor(cohort_id) |  |
| session_attendance | admins manage attendance | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| session_attendance | students read own attendance | SELECT | authenticated | PERMISSIVE | (student_user_id = auth.uid()) |  |
| session_feedback | admins manage feedback | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| session_feedback | students insert own feedback | INSERT | authenticated | PERMISSIVE |  | ((student_user_id = auth.uid()) AND (EXISTS ( SELECT 1    FROM lms.session_attendance a   WHERE ((a.session_id = session_feedback.session_id) AND (a.student_user_id = auth.uid()) AND (a.status = 'present'::text)))) AND (EXISTS ( SELECT 1    FROM (lms.sessions s      JOIN lms.enrollments e ON ((e.cohort_id = s.cohort_id)))   WHERE ((s.id = session_feedback.session_id) AND (e.student_user_id = auth.uid()) AND (e.access_status = 'active'::text))))) |
| session_feedback | students read own feedback | SELECT | authenticated | PERMISSIVE | (student_user_id = auth.uid()) |  |
| session_group_links | sgl_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| session_group_links | sgl_mentor_read | SELECT | authenticated | PERMISSIVE | lms.is_cohort_mentor(cohort_id) |  |
| session_group_links | sgl_read_own_group | SELECT | authenticated | PERMISSIVE | (EXISTS ( SELECT 1    FROM (lms.cohort_group_members m      JOIN lms.enrollments e ON (((e.student_user_id = m.student_user_id) AND (e.cohort_id = m.cohort_id) AND (e.access_status = 'active'::text))))   WHERE ((m.group_id = session_group_links.group_id) AND (m.student_user_id = auth.uid())))) |  |
| session_join_clicks | admins read join clicks | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| session_mentor_overrides | Admins manage session mentor overrides | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| session_mentor_overrides | Students read overrides for their cohorts | SELECT | authenticated | PERMISSIVE | (EXISTS ( SELECT 1    FROM lms.enrollments e   WHERE ((e.cohort_id = session_mentor_overrides.cohort_id) AND (e.student_user_id = auth.uid()) AND (e.access_status = 'active'::text)))) |  |
| session_mentor_overrides | mentor_overrides | SELECT | authenticated | PERMISSIVE | lms.is_cohort_mentor(cohort_id) |  |
| session_mentors | enrolled_session_mentors | SELECT | public | PERMISSIVE | (EXISTS ( SELECT 1    FROM (lms.sessions s      JOIN lms.enrollments e ON ((e.cohort_id = s.cohort_id)))   WHERE ((s.id = session_mentors.session_id) AND (e.student_user_id = auth.uid()) AND (e.access_status = 'active'::text)))) |  |
| session_mentors | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| session_mentors | mentor_session_mentors | SELECT | authenticated | PERMISSIVE | (EXISTS ( SELECT 1    FROM lms.sessions s   WHERE ((s.id = session_mentors.session_id) AND lms.is_cohort_mentor(s.cohort_id)))) |  |
| session_recording_watch | admins read recording watch | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| session_reminder_log | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| session_section_links | ssl_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| session_section_links | ssl_mentor_read | SELECT | authenticated | PERMISSIVE | lms.is_cohort_mentor(cohort_id) |  |
| session_section_links | ssl_read_own_section | SELECT | authenticated | PERMISSIVE | (EXISTS ( SELECT 1    FROM (lms.cohort_section_members m      JOIN lms.enrollments e ON (((e.student_user_id = m.student_user_id) AND (e.cohort_id = m.cohort_id) AND (e.access_status = 'active'::text))))   WHERE ((m.section_id = session_section_links.section_id) AND (m.student_user_id = auth.uid())))) |  |
| session_section_recordings | ssr_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| session_section_recordings | ssr_mentor_read | SELECT | authenticated | PERMISSIVE | lms.is_cohort_mentor(cohort_id) |  |
| session_section_recordings | ssr_read_own_section | SELECT | authenticated | PERMISSIVE | (EXISTS ( SELECT 1    FROM (lms.cohort_section_members m      JOIN lms.enrollments e ON (((e.student_user_id = m.student_user_id) AND (e.cohort_id = m.cohort_id) AND (e.access_status = 'active'::text))))   WHERE ((m.section_id = session_section_recordings.section_id) AND (m.student_user_id = auth.uid())))) |  |
| session_submissions | mentor_submissions | SELECT | authenticated | PERMISSIVE | lms.is_cohort_mentor(cohort_id) |  |
| session_submissions | submissions_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| session_submissions | submissions_student_delete | DELETE | authenticated | PERMISSIVE | ((student_user_id = auth.uid()) AND (EXISTS ( SELECT 1    FROM lms.enrollments e   WHERE ((e.student_user_id = auth.uid()) AND (e.cohort_id = session_submissions.cohort_id) AND (e.access_status = 'active'::text))))) |  |
| session_submissions | submissions_student_insert | INSERT | authenticated | PERMISSIVE |  | ((student_user_id = auth.uid()) AND (EXISTS ( SELECT 1    FROM lms.enrollments e   WHERE ((e.student_user_id = auth.uid()) AND (e.cohort_id = session_submissions.cohort_id) AND (e.access_status = 'active'::text))))) |
| session_submissions | submissions_student_read | SELECT | authenticated | PERMISSIVE | (student_user_id = auth.uid()) |  |
| sessions | enrolled_sessions | SELECT | authenticated | PERMISSIVE | ((EXISTS ( SELECT 1    FROM lms.enrollments e   WHERE ((e.student_user_id = auth.uid()) AND (e.cohort_id = sessions.cohort_id) AND (e.access_status = 'active'::text) AND ((sessions.pathway_id IS NULL) OR (sessions.pathway_id = e.pathway_id))))) AND ((section_id IS NULL) OR (EXISTS ( SELECT 1    FROM lms.cohort_section_members sm   WHERE ((sm.section_id = sessions.section_id) AND (sm.student_user_id = auth.uid())))))) |  |
| sessions | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| sessions | mentor_sessions | SELECT | authenticated | PERMISSIVE | lms.is_cohort_mentor(cohort_id) |  |
| student_profiles | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| student_profiles | own_profile | SELECT | public | PERMISSIVE | (auth.uid() = user_id) |  |
| support_tickets | tickets_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| support_tickets | tickets_student_insert | INSERT | public | PERMISSIVE |  | (student_user_id = auth.uid()) |
| support_tickets | tickets_student_select | SELECT | public | PERMISSIVE | (student_user_id = auth.uid()) |  |
| support_tickets | tickets_student_update | UPDATE | public | PERMISSIVE | (student_user_id = auth.uid()) | (student_user_id = auth.uid()) |
| ticket_messages | ticket_msgs_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| ticket_messages | ticket_msgs_student_insert | INSERT | public | PERMISSIVE |  | ((sender_user_id = auth.uid()) AND (sender_role = 'student'::text) AND (is_internal = false) AND (EXISTS ( SELECT 1    FROM lms.support_tickets t   WHERE ((t.id = ticket_messages.ticket_id) AND (t.student_user_id = auth.uid()))))) |
| ticket_messages | ticket_msgs_student_select | SELECT | public | PERMISSIVE | ((is_internal = false) AND (EXISTS ( SELECT 1    FROM lms.support_tickets t   WHERE ((t.id = ticket_messages.ticket_id) AND (t.student_user_id = auth.uid()))))) |  |
| user_roles | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| user_roles | own_roles | SELECT | public | PERMISSIVE | (auth.uid() = user_id) |  |
| vault_ledger | vault_ledger_own_or_admin | SELECT | authenticated | PERMISSIVE | ((student_user_id = auth.uid()) OR ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role)) |  |
| vault_lots | vault_lots_own_or_admin | SELECT | authenticated | PERMISSIVE | ((student_user_id = auth.uid()) OR ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role)) |  |
| vault_review_flags | vault_review_flags_admin | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| vault_settings | vault_settings_admin_select | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| vault_settings_history | vault_settings_history_admin_select | SELECT | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |  |
| vault_vouchers | vault_vouchers_own_or_admin | SELECT | authenticated | PERMISSIVE | ((student_user_id = auth.uid()) OR ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role)) |  |
| vault_walkthrough_seen | Own walkthrough row | SELECT | authenticated | PERMISSIVE | (user_id = auth.uid()) |  |
| whatsapp_blocklist_archive | whatsapp_blocklist_admin | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |
| whatsapp_reminder_dispatch | whatsapp_reminder_dispatch_service | ALL | service_role | PERMISSIVE | true | true |
| whatsapp_templates | lms_admin_all | ALL | authenticated | PERMISSIVE | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) | ( SELECT lms.has_role(auth.uid(), 'admin'::text) AS has_role) |

## portal (71 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| app_settings | Admins can delete app_settings | DELETE | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |  |
| app_settings | Admins can insert app_settings | INSERT | authenticated | PERMISSIVE |  | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |
| app_settings | Admins can read app_settings | SELECT | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |  |
| app_settings | Admins can update app_settings | UPDATE | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |
| app_settings | Managers can insert app_settings | INSERT | authenticated | PERMISSIVE |  | portal.is_manager(auth.uid()) |
| app_settings | Managers can read app_settings | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| app_settings | Managers can update app_settings | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| application_installments | Admins manage installments | ALL | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |
| application_installments | Managers can insert installments | INSERT | authenticated | PERMISSIVE |  | portal.is_manager(auth.uid()) |
| application_installments | Managers can read installments | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| application_installments | Managers can update installments | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| application_installments | Students read own installments | SELECT | authenticated | PERMISSIVE | (EXISTS ( SELECT 1    FROM (applications a      JOIN auth.users u ON ((lower((u.email)::text) = lower(a.student_email))))   WHERE ((a.id = application_installments.application_id) AND (u.id = auth.uid())))) |  |
| captured_leads | Admins can delete captured leads | DELETE | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |  |
| captured_leads | Admins can update captured leads | UPDATE | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |
| captured_leads | Admins can view captured leads | SELECT | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |  |
| captured_leads | Anyone can insert captured leads | INSERT | anon,authenticated | PERMISSIVE |  | ((COALESCE(length(student_email), 0) <= 254) AND (COALESCE(length(parent_email), 0) <= 254) AND (COALESCE(length(student_phone), 0) <= 32) AND (COALESCE(length(parent_phone), 0) <= 32) AND (COALESCE(length(student_name), 0) <= 200) AND (COALESCE(length(parent_name), 0) <= 200) AND (COALESCE(length(school_name), 0) <= 300) AND (COALESCE(length(lead_type), 0) <= 40) AND ((student_email IS NULL) OR (student_email = ''::text) OR (student_email ~* '^[^@[:space:]]+@[^@[:space:]]+\.[^@[:space:]]+$'::text)) AND ((parent_email IS NULL) OR (parent_email = ''::text) OR (parent_email ~* '^[^@[:space:]]+@[^@[:space:]]+\.[^@[:space:]]+$'::text))) |
| captured_leads | Managers can update captured leads | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| captured_leads | Managers can view captured leads | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| email_reminder_settings | Group admins manage email_reminder_settings | ALL | authenticated | PERMISSIVE | (portal.is_lwl_admin(auth.uid()) OR portal.is_admin(auth.uid())) | (portal.is_lwl_admin(auth.uid()) OR portal.is_admin(auth.uid())) |
| email_reminder_settings | Managers can insert email reminder settings | INSERT | authenticated | PERMISSIVE |  | portal.is_manager(auth.uid()) |
| email_reminder_settings | Managers can read email reminder settings | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| email_reminder_settings | Managers can update email reminder settings | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| email_send_log | Group admins view email_send_log | SELECT | authenticated | PERMISSIVE | (portal.is_lwl_admin(auth.uid()) OR portal.is_admin(auth.uid())) |  |
| email_send_log | Managers can view email logs | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| email_templates | Group admins manage email_templates | ALL | authenticated | PERMISSIVE | (portal.is_lwl_admin(auth.uid()) OR portal.is_admin(auth.uid())) | (portal.is_lwl_admin(auth.uid()) OR portal.is_admin(auth.uid())) |
| email_templates | Managers can insert email templates | INSERT | authenticated | PERMISSIVE |  | portal.is_manager(auth.uid()) |
| email_templates | Managers can read email templates | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| email_templates | Managers can update email templates | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| form_templates | Admins can view form templates | SELECT | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |  |
| form_templates | Group admins can manage form templates | ALL | authenticated | PERMISSIVE | portal.is_lwl_admin(auth.uid()) | portal.is_lwl_admin(auth.uid()) |
| form_templates | Managers can insert form templates | INSERT | authenticated | PERMISSIVE |  | portal.is_manager(auth.uid()) |
| form_templates | Managers can read form templates | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| form_templates | Managers can update form templates | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| masterclass_form_templates | Admins can view masterclass form templates | SELECT | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |  |
| masterclass_form_templates | Group admins can manage masterclass form templates | ALL | authenticated | PERMISSIVE | portal.is_lwl_admin(auth.uid()) | portal.is_lwl_admin(auth.uid()) |
| masterclass_form_templates | Managers can insert masterclass form templates | INSERT | authenticated | PERMISSIVE |  | portal.is_manager(auth.uid()) |
| masterclass_form_templates | Managers can read masterclass form templates | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| masterclass_form_templates | Managers can update masterclass form templates | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| mentors | mentors_admin_manage | ALL | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |
| mentors | mentors_public_read | SELECT | public | PERMISSIVE | (is_active = true) |  |
| newsletter_subscribers | Allow public inserts on newsletter_subscribers | INSERT | anon,authenticated | PERMISSIVE |  | ((email IS NOT NULL) AND (email ~* '^[^@\s]+@[^@\s]+\.[^@\s]+$'::text) AND ((length(email) >= 3) AND (length(email) <= 320))) |
| newsletter_subscribers | Group admins can update newsletter_subscribers | UPDATE | authenticated | PERMISSIVE | portal.is_lwl_admin(auth.uid()) |  |
| newsletter_subscribers | Group admins can view newsletter_subscribers | SELECT | authenticated | PERMISSIVE | portal.is_lwl_admin(auth.uid()) |  |
| newsletter_subscribers | Managers can update newsletter subscribers | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| newsletter_subscribers | Managers can view newsletter subscribers | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| newsletter_subscribers | Users can delete own newsletter subscription | DELETE | authenticated | PERMISSIVE | (email = (( SELECT users.email    FROM auth.users   WHERE (users.id = auth.uid())))::text) |  |
| payment_links | Admins manage payment links | ALL | authenticated | PERMISSIVE | (EXISTS ( SELECT 1    FROM portal.user_profiles up   WHERE ((up.user_id = auth.uid()) AND (up.role = ANY (ARRAY['admin'::portal.app_role, 'lwl_admin'::portal.app_role, 'super_admin'::portal.app_role]))))) | (EXISTS ( SELECT 1    FROM portal.user_profiles up   WHERE ((up.user_id = auth.uid()) AND (up.role = ANY (ARRAY['admin'::portal.app_role, 'lwl_admin'::portal.app_role, 'super_admin'::portal.app_role]))))) |
| program_attachments | Group admins manage program_attachments | ALL | authenticated | PERMISSIVE | (portal.is_lwl_admin(auth.uid()) OR portal.is_admin(auth.uid())) | (portal.is_lwl_admin(auth.uid()) OR portal.is_admin(auth.uid())) |
| program_attachments | Managers can insert program attachments | INSERT | authenticated | PERMISSIVE |  | portal.is_manager(auth.uid()) |
| program_attachments | Managers can read program attachments | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| program_attachments | Managers can update program attachments | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| program_email_settings | Admins can read program_email_settings | SELECT | authenticated | PERMISSIVE | (portal.has_role(auth.uid(), 'lwl_admin'::text) OR portal.has_role(auth.uid(), 'admin'::text)) |  |
| program_email_settings | Group admins manage program_email_settings | ALL | authenticated | PERMISSIVE | (portal.is_lwl_admin(auth.uid()) OR portal.is_admin(auth.uid())) | (portal.is_lwl_admin(auth.uid()) OR portal.is_admin(auth.uid())) |
| program_email_settings | Managers can insert program email settings | INSERT | authenticated | PERMISSIVE |  | portal.is_manager(auth.uid()) |
| program_email_settings | Managers can read program email settings | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| program_email_settings | Managers can update program email settings | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| school_program_inquiries | Allow public inserts on school_program_inquiries | INSERT | anon,authenticated | PERMISSIVE |  | ((school_name IS NOT NULL) AND ((length(school_name) >= 1) AND (length(school_name) <= 300)) AND (contact_person_name IS NOT NULL) AND ((length(contact_person_name) >= 1) AND (length(contact_person_name) <= 200)) AND (email IS NOT NULL) AND (email ~* '^[^@\s]+@[^@\s]+\.[^@\s]+$'::text) AND (phone IS NOT NULL) AND ((length(phone) >= 4) AND (length(phone) <= 32)) AND (length(COALESCE(special_requirements, ''::text)) <= 4000)) |
| school_program_inquiries | Group admins can update school_program_inquiries | UPDATE | authenticated | PERMISSIVE | portal.is_lwl_admin(auth.uid()) |  |
| school_program_inquiries | Group admins can view all school_program_inquiries | SELECT | authenticated | PERMISSIVE | portal.is_lwl_admin(auth.uid()) |  |
| school_program_inquiries | Managers can update school inquiries | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| school_program_inquiries | Managers can view school inquiries | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| user_profiles | Admins can delete profiles | DELETE | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |  |
| user_profiles | Admins can insert profiles | INSERT | authenticated | PERMISSIVE |  | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |
| user_profiles | Admins can update all profiles | UPDATE | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |
| user_profiles | Admins can view all profiles | SELECT | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |  |
| user_profiles | Managers can update profiles | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| user_profiles | Managers can view profiles | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| user_profiles | Users can insert own profile | INSERT | authenticated | PERMISSIVE |  | ((user_id = auth.uid()) AND (role = 'student'::portal.app_role)) |
| user_profiles | Users can view own profile | SELECT | authenticated | PERMISSIVE | (user_id = auth.uid()) |  |
| user_profiles | users_update_own_profile_no_escalation | UPDATE | authenticated | PERMISSIVE | (user_id = auth.uid()) | ((user_id = auth.uid()) AND (role = ( SELECT up.role    FROM portal.user_profiles up   WHERE (up.user_id = auth.uid()))) AND (NOT (school_name IS DISTINCT FROM ( SELECT up.school_name    FROM portal.user_profiles up   WHERE (up.user_id = auth.uid()))))) |
| whatsapp_drip_sequences | Admins manage whatsapp drip sequences | ALL | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |

## public (28 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| all_cohorts | all_cohorts_read | SELECT | public | PERMISSIVE | (EXISTS ( SELECT 1    FROM all_programs programme   WHERE ((programme.id = all_cohorts.program_id) AND (programme.is_active = true)))) |  |
| all_masterclasses | all_masterclasses_doctors_admin_write | ALL | authenticated | PERMISSIVE | ((partner_id = ( SELECT premed.partner_id() AS partner_id)) AND (premed.has_role(auth.uid(), 'admin'::premed.app_role) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role))) | ((partner_id = ( SELECT premed.partner_id() AS partner_id)) AND (premed.has_role(auth.uid(), 'admin'::premed.app_role) OR premed.has_role(auth.uid(), 'mentor'::premed.app_role))) |
| all_masterclasses | all_masterclasses_public_read | SELECT | anon | PERMISSIVE | is_published |  |
| all_masterclasses | all_masterclasses_read | SELECT | authenticated | PERMISSIVE | is_published |  |
| all_mentors | all_mentors_lms_admin_write | ALL | authenticated | PERMISSIVE | lms.has_role(auth.uid(), 'admin'::text) | lms.has_role(auth.uid(), 'admin'::text) |
| all_mentors | all_mentors_lwl_admin_insert | INSERT | authenticated | PERMISSIVE |  | is_finance_admin(auth.uid()) |
| all_mentors | all_mentors_lwl_admin_update | UPDATE | authenticated | PERMISSIVE | is_finance_admin(auth.uid()) | is_finance_admin(auth.uid()) |
| all_mentors | all_mentors_onward_read | SELECT | fellowship_writer | PERMISSIVE | true |  |
| all_mentors | all_mentors_public_read | SELECT | anon | PERMISSIVE | is_active |  |
| all_mentors | all_mentors_read | SELECT | authenticated | PERMISSIVE | is_active |  |
| all_partners | all_partners_read | SELECT | authenticated | PERMISSIVE | (is_active OR lms.has_role(( SELECT auth.uid() AS uid), 'admin'::text)) |  |
| all_programs | all_programs_read | SELECT | public | PERMISSIVE | (is_active = true) |  |
| premed_masterclass_regs | doctors_regs_staff_manage | ALL | authenticated | PERMISSIVE | (premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) | (premed.has_role(auth.uid(), 'mentor'::premed.app_role) OR premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| applications | Admins can update all_applications | UPDATE | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |
| applications | Admins can view all_applications | SELECT | authenticated | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |  |
| applications | Allow public inserts on all_applications | INSERT | anon,authenticated | PERMISSIVE |  | ((student_email IS NOT NULL) AND ((length(student_email) >= 3) AND (length(student_email) <= 320)) AND (student_email ~* '^[^@\s]+@[^@\s]+\.[^@\s]+$'::text) AND (student_name IS NOT NULL) AND ((length(student_name) >= 1) AND (length(student_name) <= 200)) AND (payment_status = 'pending'::text)) |
| applications | Managers can update applications | UPDATE | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) | portal.is_manager(auth.uid()) |
| applications | Managers can view applications | SELECT | authenticated | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| applications | School admins can view their school applications | SELECT | authenticated | PERMISSIVE | (portal.is_school_admin(auth.uid()) AND (school_name IS NOT NULL) AND (school_name = portal.get_user_school_name(auth.uid()))) |  |
| applications | Students can view own applications | SELECT | authenticated | PERMISSIVE | ((current_verified_email() IS NOT NULL) AND ((lower(student_email) = current_verified_email()) OR (lower(COALESCE(parent_email, ''::text)) = current_verified_email()))) |  |
| masterclass_regs | Admins can update masterclass regs | UPDATE | public | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |
| masterclass_regs | Admins can view masterclass regs | SELECT | public | PERMISSIVE | (portal.is_admin(auth.uid()) OR portal.is_lwl_admin(auth.uid())) |  |
| masterclass_regs | Allow public inserts on masterclass_regs | INSERT | public | PERMISSIVE |  | ((email IS NOT NULL) AND ((length(email) >= 3) AND (length(email) <= 320)) AND (email ~* '^[^@\s]+@[^@\s]+\.[^@\s]+$'::text) AND (full_name IS NOT NULL) AND ((length(full_name) >= 1) AND (length(full_name) <= 200)) AND (status = 'registered'::text)) |
| masterclass_regs | Managers can view masterclass regs | SELECT | public | PERMISSIVE | portal.is_manager(auth.uid()) |  |
| masterclass_regs | Registrants can view own masterclass regs | SELECT | public | PERMISSIVE | ((lower(email) = lower(COALESCE((auth.jwt() ->> 'email'::text), ''::text))) OR (lower(COALESCE(parent_email, ''::text)) = lower(COALESCE((auth.jwt() ->> 'email'::text), ''::text)))) |  |
| fellowship_masterclass_regs | fellowship_writer_all | ALL | fellowship_writer | PERMISSIVE | true | true |
| testprep_masterclass_regs | Public can submit valid masterclass registration | INSERT | anon,authenticated | PERMISSIVE |  | ((email_sent IS NOT TRUE) AND ((char_length(btrim(COALESCE(full_name, ''::text))) >= 1) AND (char_length(btrim(COALESCE(full_name, ''::text))) <= 200)) AND ((char_length(btrim(COALESCE(email, ''::text))) >= 3) AND (char_length(btrim(COALESCE(email, ''::text))) <= 255)) AND (email ~* '^[^@\s]+@[^@\s]+\.[^@\s]+$'::text) AND ((char_length(btrim(COALESCE(phone, ''::text))) >= 3) AND (char_length(btrim(COALESCE(phone, ''::text))) <= 50)) AND ((char_length(btrim(COALESCE(country, ''::text))) >= 1) AND (char_length(btrim(COALESCE(country, ''::text))) <= 120)) AND ((char_length(btrim(COALESCE(role, ''::text))) >= 1) AND (char_length(btrim(COALESCE(role, ''::text))) <= 80)) AND ((COALESCE(array_length(selected_sessions, 1), 0) >= 1) AND (COALESCE(array_length(selected_sessions, 1), 0) <= 10)) AND ((grade IS NULL) OR (char_length(grade) <= 80)) AND ((school_name IS NULL) OR (char_length(school_name) <= 200)) AND ((parent_name IS NULL) OR (char_length(parent_name) <= 200)) AND ((parent_email IS NULL) OR (parent_email = ''::text) OR ((char_length(parent_email) <= 255) AND (parent_email ~* '^[^@\s]+@[^@\s]+\.[^@\s]+$'::text))) AND ((parent_phone IS NULL) OR (char_length(parent_phone) <= 50)) AND ((student_name IS NULL) OR (char_length(student_name) <= 200)) AND ((student_grade IS NULL) OR (char_length(student_grade) <= 80)) AND ((designation IS NULL) OR (char_length(designation) <= 200))) |
| week_without_walls_inquiries | fellowship_writer_all | ALL | fellowship_writer | PERMISSIVE | true | true |

## testprep (6 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| country_dial_codes | Dial codes are public reference data | SELECT | public | PERMISSIVE | (is_public = true) |  |
| lead_events | Public can insert valid lead events | INSERT | anon,authenticated | PERMISSIVE |  | ((session_id IS NOT NULL) AND (session_id = testprep.current_session_id()) AND ((char_length(session_id) >= 32) AND (char_length(session_id) <= 120)) AND ((page_url IS NULL) OR (char_length(page_url) <= 1000)) AND ((user_agent IS NULL) OR (char_length(user_agent) <= 1000)) AND ((referrer IS NULL) OR (char_length(referrer) <= 1000)) AND ((screen_width IS NULL) OR ((screen_width >= 1) AND (screen_width <= 10000))) AND ((screen_height IS NULL) OR ((screen_height >= 1) AND (screen_height <= 10000)))) |
| leads | Public can insert valid leads | INSERT | anon,authenticated | PERMISSIVE |  | (((email IS NULL) OR ((char_length(email) <= 255) AND (email ~* '^[^@\s]+@[^@\s]+\.[^@\s]+$'::text))) AND ((name IS NULL) OR (char_length(name) <= 200)) AND ((session_id IS NOT NULL) AND (session_id = testprep.current_session_id()) AND ((char_length(session_id) >= 32) AND (char_length(session_id) <= 120))) AND ((referrer_url IS NULL) OR (char_length(referrer_url) <= 1000)) AND ((landing_page IS NULL) OR (char_length(landing_page) <= 500)) AND (status = ANY (ARRAY['partial'::testprep.lead_status, 'complete'::testprep.lead_status])) AND (converted_at IS NULL)) |
| sat_questions | Published questions are publicly viewable | SELECT | anon,authenticated | PERMISSIVE | (published = true) |  |
| school_inquiries | Public can submit valid school inquiries | INSERT | anon,authenticated | PERMISSIVE |  | (((char_length(btrim(COALESCE(school_name, ''::text))) >= 2) AND (char_length(btrim(COALESCE(school_name, ''::text))) <= 200)) AND ((char_length(btrim(COALESCE(contact_name, ''::text))) >= 2) AND (char_length(btrim(COALESCE(contact_name, ''::text))) <= 200)) AND ((char_length(btrim(COALESCE(contact_email, ''::text))) >= 3) AND (char_length(btrim(COALESCE(contact_email, ''::text))) <= 255)) AND (contact_email ~* '^[^@\s]+@[^@\s]+\.[^@\s]+$'::text) AND ((contact_phone IS NULL) OR (char_length(contact_phone) <= 50)) AND ((contact_title IS NULL) OR (char_length(contact_title) <= 200)) AND ((preferred_program IS NULL) OR (char_length(preferred_program) <= 200)) AND ((preferred_dates IS NULL) OR (char_length(preferred_dates) <= 300)) AND ((notes IS NULL) OR (char_length(notes) <= 2500)) AND ((estimated_students IS NULL) OR ((estimated_students >= 1) AND (estimated_students <= 10000))) AND (status = 'new'::text)) |
| session_analytics | Public can insert valid session analytics | INSERT | anon,authenticated | PERMISSIVE |  | ((session_id IS NOT NULL) AND (session_id = testprep.current_session_id()) AND ((char_length(session_id) >= 32) AND (char_length(session_id) <= 120)) AND ((first_page IS NULL) OR (char_length(first_page) <= 500)) AND ((last_page IS NULL) OR (char_length(last_page) <= 500)) AND ((device_type IS NULL) OR (char_length(device_type) <= 50)) AND ((browser IS NULL) OR (char_length(browser) <= 80)) AND (COALESCE(page_count, 0) >= 0) AND (COALESCE(duration_seconds, 0) >= 0)) |

## research (14 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| analytics_events | analytics_events_admin_read | SELECT | authenticated | PERMISSIVE | research.has_role(auth.uid(), 'admin'::research.app_role) |  |
| country_dial_aliases | country_dial_aliases_admin_read | SELECT | authenticated | PERMISSIVE | research.has_role(auth.uid(), 'admin'::research.app_role) |  |
| country_dial_aliases | country_dial_aliases_public_read | SELECT | anon,authenticated | PERMISSIVE | (is_public = true) |  |
| country_dial_codes | country_dial_codes_admin_read | SELECT | authenticated | PERMISSIVE | research.has_role(auth.uid(), 'admin'::research.app_role) |  |
| country_dial_codes | country_dial_codes_public_read | SELECT | anon,authenticated | PERMISSIVE | (is_public = true) |  |
| email_send_log | email_send_log_admin_read | SELECT | authenticated | PERMISSIVE | research.has_role(auth.uid(), 'admin'::research.app_role) |  |
| email_templates | email_templates_admin_read | SELECT | authenticated | PERMISSIVE | research.has_role(auth.uid(), 'admin'::research.app_role) |  |
| outreach_sync_log | outreach_sync_log_admin_read | SELECT | authenticated | PERMISSIVE | research.has_role(auth.uid(), 'admin'::research.app_role) |  |
| program_email_settings | program_email_settings_admin_read | SELECT | authenticated | PERMISSIVE | research.has_role(auth.uid(), 'admin'::research.app_role) |  |
| suppression_list | suppression_list_admin_read | SELECT | authenticated | PERMISSIVE | research.has_role(auth.uid(), 'admin'::research.app_role) |  |
| user_roles | user_roles_self_read | SELECT | authenticated | PERMISSIVE | (user_id = auth.uid()) |  |
| webhook_log | webhook_log_admin_read | SELECT | authenticated | PERMISSIVE | research.has_role(auth.uid(), 'admin'::research.app_role) |  |
| whatsapp_messages | whatsapp_messages_admin_read | SELECT | authenticated | PERMISSIVE | research.has_role(auth.uid(), 'admin'::research.app_role) |  |
| whatsapp_settings | whatsapp_settings_admin_read | SELECT | authenticated | PERMISSIVE | research.has_role(auth.uid(), 'admin'::research.app_role) |  |

## storage (12 policies)

| table | policy | cmd | roles | permissive | USING (qual) | WITH CHECK |
|---|---|---|---|---|---|---|
| objects | announcement_media_auth_delete | DELETE | authenticated | PERMISSIVE | ((bucket_id = 'announcement-media'::text) AND lms.has_role(auth.uid(), 'admin'::text)) |  |
| objects | announcement_media_auth_update | UPDATE | authenticated | PERMISSIVE | ((bucket_id = 'announcement-media'::text) AND lms.has_role(auth.uid(), 'admin'::text)) |  |
| objects | announcement_media_auth_write | INSERT | authenticated | PERMISSIVE |  | ((bucket_id = 'announcement-media'::text) AND lms.has_role(( SELECT auth.uid() AS uid), 'admin'::text)) |
| objects | doctors_attach_delete | DELETE | authenticated | PERMISSIVE | ((bucket_id = 'program-attachments'::text) AND ((storage.foldername(name))[1] = 'premed'::text) AND premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| objects | doctors_attach_read | SELECT | authenticated | PERMISSIVE | ((bucket_id = 'program-attachments'::text) AND ((storage.foldername(name))[1] = 'premed'::text) AND premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| objects | doctors_attach_update | UPDATE | authenticated | PERMISSIVE | ((bucket_id = 'program-attachments'::text) AND ((storage.foldername(name))[1] = 'premed'::text) AND premed.has_role(auth.uid(), 'admin'::premed.app_role)) |  |
| objects | doctors_attach_write | INSERT | authenticated | PERMISSIVE |  | ((bucket_id = 'program-attachments'::text) AND ((storage.foldername(name))[1] = 'premed'::text) AND premed.has_role(auth.uid(), 'admin'::premed.app_role)) |
| objects | lms_submissions_delete_own | DELETE | authenticated | PERMISSIVE | ((bucket_id = 'lms-submissions'::text) AND (lms.has_role(auth.uid(), 'admin'::text) OR (((storage.foldername(name))[3] = (auth.uid())::text) AND (EXISTS ( SELECT 1    FROM lms.enrollments e   WHERE ((e.student_user_id = auth.uid()) AND (e.access_status = 'active'::text) AND ((e.cohort_id)::text = (storage.foldername(objects.name))[1]))))))) |  |
| objects | lms_submissions_insert | INSERT | authenticated | PERMISSIVE |  | ((bucket_id = 'lms-submissions'::text) AND ((storage.foldername(name))[3] = (auth.uid())::text) AND (EXISTS ( SELECT 1    FROM lms.enrollments e   WHERE ((e.student_user_id = auth.uid()) AND (e.access_status = 'active'::text) AND ((e.cohort_id)::text = (storage.foldername(objects.name))[1]))))) |
| objects | lms_submissions_read_own | SELECT | authenticated | PERMISSIVE | ((bucket_id = 'lms-submissions'::text) AND (((storage.foldername(name))[3] = (auth.uid())::text) OR lms.has_role(auth.uid(), 'admin'::text))) |  |
| objects | mentor_photos_admin_write | ALL | authenticated | PERMISSIVE | ((bucket_id = 'mentor-photos'::text) AND lms.has_role(auth.uid(), 'admin'::text)) | ((bucket_id = 'mentor-photos'::text) AND lms.has_role(auth.uid(), 'admin'::text)) |
| objects | ticket attachments upload | INSERT | authenticated | PERMISSIVE |  | ((bucket_id = 'ticket-attachments'::text) AND ((storage.foldername(name))[1] = ( SELECT (auth.uid())::text AS uid))) |

## Observations

- Totals: **385 policies** (affiliate 2, premed 90, finance 3, finedu 24, ops 5, prelaw 6, lms 124, portal 71, public 28, testprep 6, research 14, storage 12). No policies exist in `auth`, `realtime`, `fellowship` or `leaders`. Every policy in the estate is PERMISSIVE; there are no RESTRICTIVE policies.
- **RLS on, zero policies** (so anon/authenticated are denied regardless of GRANTs; access only via `service_role`, SECURITY DEFINER functions, or the `fellowship_writer` login role): all 17 `fellowship` tables; all 22 `leaders` tables; affiliate (8 of 10: admins, announcements, partners, payment_link_links, payouts, promo_code_links, registration_attempts, schools); finance (24 of 27, everything except codes_dropdown, expense_codes, program_codes); premed (concierge_rate_limits, stripe_charge_cache, stripe_reconcile_state); finedu (catalogue_audit, email_send_state, flywire_webhook_events, payment_reminder_log, program_map, unmatched_payments, voucher_redemptions); ops (agent_campaign_calls, agent_campaign_contacts, agent_campaigns, agent_profiles, payment_followup_leads); prelaw (promo_slot_claims); lms (feedback_email_tokens, referral_application_attribution, referral_first_payments, referral_join_rewards, referral_lookup_attempts, referral_program_integrations, referral_program_links, referral_verifications, vault_fx_rates); portal (camp_leads, camp_payments, legacy_form_templates, megan_pending_actions, rate_limit_hits, whatsapp_undeliverable_numbers); public (automation_app_config, automation_video_posts, data_fix_log, finedu_masterclass_regs, prelaw_masterclass_regs, form_responses, master_data, platform_api_clients, promo_codes, promo_redemptions, razorpay_payments, research_masterclass_regs, stripe_payments, leaders_applications); testprep (14 of 20); research (30 of 44).
- Consequence for `public`: anon/authenticated hold broad table GRANTs there (see `shared-estate-grants.md`) but with RLS enabled and no policy the effective access is none. The GRANTs only matter on the tables that do have policies.
- Policies whose role is `public` (= every role, including anon): finedu.assessment_questions read, portal.mentors read, public.all_cohorts / public.all_programs read, the five `public.masterclass_regs` policies, testprep.country_dial_codes read, and in lms: enrollments.own_enrollments, help_messages.help_own, notification_recipients.own_notifications, program_feedback.program_feedback_insert_own / program_feedback_read_own, resources.enrolled_resources, session_mentors.enrolled_session_mentors, student_profiles.own_profile, support_tickets.tickets_student_*, ticket_messages.ticket_msgs_student_*, user_roles.own_roles. These all still require `auth.uid()` or an active enrollment, so anon gets nothing from them except the published catalog views.
- Anonymous INSERT paths (lead capture) are gated by WITH CHECK validators rather than functions: portal.captured_leads, portal.newsletter_subscribers, portal.school_program_inquiries, public.applications (`payment_status = 'pending'` forced), public.testprep_masterclass_regs, finedu.concierge_leads / page_events / page_visits, testprep.leads / lead_events / session_analytics / school_inquiries (bound to `testprep.current_session_id()`), public.masterclass_regs.
- Non-standard roles referenced by policies: `fellowship_writer` (public.all_mentors read; ALL on public.fellowship_masterclass_regs and public.week_without_walls_inquiries) is a LOGIN role used by the Fellowship Platform site's own DB connection; explicit `service_role` policies on ops.contact_suppressions, lms.community_session_reminder_log, lms.whatsapp_reminder_dispatch are redundant (service_role has BYPASSRLS).
- Role-check helpers per platform (all SECURITY DEFINER, STABLE): `premed.has_role(uid, app_role)`, `finedu.is_admin()`, `finance.is_finance_user()` + `public.is_finance_admin(uid)`, `ops.is_admin()`, `prelaw.has_role`, `lms.has_role(uid,'admin')` + `lms.is_cohort_mentor` / `lms.is_cohort_student` / `lms.is_my_catalog_mentor` / `lms.can_view_channel`, `portal.is_admin` / `is_lwl_admin` / `is_manager` / `is_school_admin` / `has_role`, `research.has_role`. lms policies wrap the call as `( SELECT lms.has_role(...) )` so it is evaluated once per statement.
- Student self-service on central tables: `public.applications` "Students can view own applications" uses `public.current_verified_email()` and matches either `student_email` or `parent_email`; `portal.application_installments` "Students read own installments" joins `auth.users` by e-mail.
- research: FORCE ROW LEVEL SECURITY on 41 of 44 tables with only 14 read policies (admin reads, public dial-code lookups, user_roles self-read). All public writes go through SECURITY DEFINER RPCs (`research.submit_public_application`, `research.submit_public_masterclass_registration`) or service_role.
- `public.all_masterclasses`: anon/authenticated can read only `is_published` rows, and because `join_url`/`passcode` are columns on the base table, platforms expose `public.all_masterclasses_public` (view without those columns) to browsers; `lms.all_masterclasses` returns join_url/passcode only when `lms.am_i_registered_for(id)`.
- storage.objects (12 policies): announcement-media (lms admin write), program-attachments/premed/* (premed admin RWD), lms-submissions (insert/read/delete own file under `<cohort_id>/.../<uid>/` while enrollment is active, admins read all), mentor-photos (lms admin ALL), ticket-attachments (insert into own `<uid>/` folder). Other buckets rely on the bucket `public` flag or service_role.

