# Shared estate columns: research (44 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### research.analytics_events

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| page_path | text | NOT NULL |  |
| event_type | text | NOT NULL | 'page_view'::text |
| referrer | text | null |  |
| user_agent | text | null |  |
| device_type | text | null |  |
| session_id | text | null |  |
| metadata | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| country | text | null |  |
| city | text | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |

Constraints:
- `analytics_events_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX analytics_events_pkey USING btree (id)
- INDEX idx_analytics_events_country USING btree (country)
- INDEX idx_analytics_events_created_at USING btree (created_at DESC)
- INDEX idx_analytics_events_event_type USING btree (event_type)
- INDEX idx_analytics_events_page_path USING btree (page_path)
- INDEX idx_analytics_events_utm_source USING btree (utm_source)

### research.captured_leads

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| source | text | NOT NULL | 'one_tap'::text |
| email | text | NOT NULL |  |
| full_name | text | null |  |
| picture_url | text | null |  |
| google_sub | text | null |  |
| country | text | null |  |
| city | text | null |  |
| region | text | null |  |
| ip | text | null |  |
| page_url | text | null |  |
| referrer | text | null |  |
| user_agent | text | null |  |
| language | text | null |  |
| timezone | text | null |  |
| screen | text | null |  |
| viewport | text | null |  |
| dwell_seconds | integer | null |  |
| session_started_at | timestamp with time zone | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| raw_data | jsonb | null | '{}'::jsonb |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `captured_leads_pkey` PRIMARY KEY (id)

Indexes:
- INDEX captured_leads_created_at_idx USING btree (created_at DESC)
- UNIQUE INDEX captured_leads_email_lower_idx USING btree (lower(email))
- UNIQUE INDEX captured_leads_pkey USING btree (id)

### research.chat_conversations

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | text | NOT NULL |  |
| parent_email | text | null |  |
| parent_name | text | null |  |
| child_grade | text | null |  |
| recommended_program | text | null |  |
| messages_count | integer | NOT NULL | 0 |
| converted | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `chat_conversations_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX chat_conversations_pkey USING btree (id)

### research.citations

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| project_id | uuid | NOT NULL |  |
| user_id | uuid | NOT NULL |  |
| title | text | NOT NULL |  |
| authors | text | null |  |
| publication | text | null |  |
| year | text | null |  |
| url | text | null |  |
| citation_type | text | NOT NULL | 'article'::text |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `citations_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX citations_pkey USING btree (id)

### research.country_dial_aliases

| column | type | nullable | default |
|---|---|---|---|
| alias | text | NOT NULL |  |
| iso2 | text | NOT NULL |  |
| is_public | boolean | NOT NULL | false |

Constraints:
- `country_dial_aliases_pkey` PRIMARY KEY (alias)

Indexes:
- UNIQUE INDEX country_dial_aliases_pkey USING btree (alias)

### research.country_dial_codes

| column | type | nullable | default |
|---|---|---|---|
| iso2 | text | NOT NULL |  |
| name | text | NOT NULL |  |
| dial | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| is_public | boolean | NOT NULL | false |

Constraints:
- `country_dial_codes_pkey` PRIMARY KEY (iso2)

Indexes:
- UNIQUE INDEX country_dial_codes_pkey USING btree (iso2)

### research.email_send_log

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| message_id | text | null |  |
| template_name | text | null |  |
| recipient_email | text | NOT NULL |  |
| application_id | uuid | null |  |
| status | text | NOT NULL | 'sent'::text |
| error_message | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `email_send_log_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX email_send_log_pkey USING btree (id)
- INDEX idx_email_send_log_created USING btree (created_at DESC)
- INDEX idx_email_send_log_status USING btree (status)

### research.email_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| subject | text | NOT NULL |  |
| html_body | text | NOT NULL |  |
| body_text | text | null |  |
| sender_name | text | NOT NULL | 'Sir Luther Center'::text |
| trigger_type | text | NOT NULL |  |
| program_key | text | null |  |
| is_active | boolean | NOT NULL | false |
| description | text | null |  |
| variables | jsonb | null | '[]'::jsonb |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `email_templates_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX email_templates_pkey USING btree (id)
- INDEX idx_email_templates_program USING btree (program_key)
- INDEX idx_email_templates_trigger_active USING btree (trigger_type, is_active)

### research.generated_proposals

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| subject | text | NOT NULL |  |
| grade_level | text | NOT NULL |  |
| interests | text | null |  |
| constraints | text | null |  |
| research_style | text | null |  |
| journal_tier | text | null |  |
| topic_data | jsonb | null |  |
| proposal_data | jsonb | null |  |
| selected_topic_index | integer | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `generated_proposals_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX generated_proposals_pkey USING btree (id)

### research.import_batches

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| source_label | text | NOT NULL |  |
| audience | text | NOT NULL |  |
| row_count | integer | NOT NULL | 0 |
| success_count | integer | NOT NULL | 0 |
| skipped_count | integer | NOT NULL | 0 |
| error_count | integer | NOT NULL | 0 |
| uploaded_by | uuid | null |  |
| notes | jsonb | null | '{}'::jsonb |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `import_batches_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_import_batches_created_at USING btree (created_at DESC)
- UNIQUE INDEX import_batches_pkey USING btree (id)

### research.masterclass_attendees

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| event_id | uuid | NOT NULL |  |
| full_name | text | null |  |
| email | text | NOT NULL |  |
| school_name | text | null |  |
| country | text | null |  |
| grade | text | null |  |
| joined_at | timestamp with time zone | null |  |
| converted_application_id | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| target_university | text | null |  |
| applying_this_year | boolean | null |  |
| subject_interest | text | null |  |
| poll_answers | jsonb | null | '{}'::jsonb |

Constraints:
- `masterclass_attendees_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_masterclass_attendees_email USING btree (lower(email))
- INDEX idx_masterclass_attendees_event USING btree (event_id)
- UNIQUE INDEX masterclass_attendees_pkey USING btree (id)
- UNIQUE INDEX uq_masterclass_attendee_event_email USING btree (event_id, lower(email))

### research.masterclass_events

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| title | text | NOT NULL |  |
| event_date | date | NOT NULL |  |
| host | text | null |  |
| source_label | text | null |  |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `masterclass_events_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX masterclass_events_pkey USING btree (id)

### research.mentor_questions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| name | text | NOT NULL |  |
| email | text | NOT NULL |  |
| phone | text | null |  |
| school_name | text | null |  |
| grade | text | null |  |
| topic | text | NOT NULL |  |
| question_text | text | NOT NULL |  |
| context_text | text | null |  |
| urgency | text | NOT NULL | 'either'::text |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| status | text | NOT NULL | 'new'::text |

Constraints:
- `mentor_questions_status_check` CHECK ((status = ANY (ARRAY['new'::text, 'answered'::text, 'archived'::text])))
- `mentor_questions_urgency_check` CHECK ((urgency = ANY (ARRAY['priority'::text, 'standard'::text, 'either'::text])))
- `mentor_questions_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_mentor_questions_status_created USING btree (status, created_at DESC)
- UNIQUE INDEX mentor_questions_pkey USING btree (id)

### research.module_assignments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| teacher_id | uuid | NOT NULL |  |
| student_id | uuid | NOT NULL |  |
| project_id | uuid | NOT NULL |  |
| module_number | integer | NOT NULL |  |
| due_date | timestamp with time zone | null |  |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `module_assignments_pkey` PRIMARY KEY (id)
- `module_assignments_student_id_project_id_module_number_key` UNIQUE (student_id, project_id, module_number)

Indexes:
- UNIQUE INDEX module_assignments_pkey USING btree (id)
- UNIQUE INDEX module_assignments_student_id_project_id_module_number_key USING btree (student_id, project_id, module_number)

### research.module_progress

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| project_id | uuid | null |  |
| module_number | integer | NOT NULL |  |
| completed | boolean | NOT NULL | false |
| checkpoint_passed | boolean | NOT NULL | false |
| completed_at | timestamp with time zone | null |  |

Constraints:
- `module_progress_pkey` PRIMARY KEY (id)
- `module_progress_user_id_module_number_key` UNIQUE (user_id, module_number)

Indexes:
- UNIQUE INDEX module_progress_pkey USING btree (id)
- UNIQUE INDEX module_progress_user_id_module_number_key USING btree (user_id, module_number)

### research.outreach_forecast

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| snapshot_date | date | NOT NULL | CURRENT_DATE |
| target_students | integer | NOT NULL | 5000 |
| students_committed | integer | NOT NULL | 0 |
| students_predicted_30d | numeric | NOT NULL | 0 |
| students_predicted_90d | numeric | NOT NULL | 0 |
| schools_contacted | integer | NOT NULL | 0 |
| schools_responded | integer | NOT NULL | 0 |
| schools_partnered | integer | NOT NULL | 0 |
| pace_status | text | NOT NULL | 'on_track'::text |
| weekly_required_outreach | integer | NOT NULL | 0 |
| notes | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `outreach_forecast_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_outreach_forecast_date USING btree (snapshot_date DESC)
- UNIQUE INDEX outreach_forecast_pkey USING btree (id)

### research.outreach_segment_performance

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| segment_key | text | NOT NULL |  |
| region | text | null |  |
| tuition_tier | text | null |  |
| school_type | text | null |  |
| schools_total | integer | NOT NULL | 0 |
| schools_contacted | integer | NOT NULL | 0 |
| schools_responded | integer | NOT NULL | 0 |
| schools_partnered | integer | NOT NULL | 0 |
| response_rate | numeric | NOT NULL | 0 |
| partner_rate | numeric | NOT NULL | 0 |
| avg_students_per_partner | numeric | NOT NULL | 0 |
| last_updated | timestamp with time zone | NOT NULL | now() |

Constraints:
- `outreach_segment_performance_pkey` PRIMARY KEY (id)
- `outreach_segment_performance_segment_key_key` UNIQUE (segment_key)

Indexes:
- INDEX idx_segment_performance_key USING btree (segment_key)
- UNIQUE INDEX outreach_segment_performance_pkey USING btree (id)
- UNIQUE INDEX outreach_segment_performance_segment_key_key USING btree (segment_key)

### research.outreach_sync_log

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| contact_id | uuid | null |  |
| audience | text | NOT NULL |  |
| campaign_id | text | null |  |
| provider | text | NOT NULL | 'smartlead'::text |
| action | text | NOT NULL |  |
| status | text | NOT NULL |  |
| payload | jsonb | null |  |
| response | jsonb | null |  |
| error_message | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `outreach_sync_log_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_outreach_sync_log_contact USING btree (contact_id)
- INDEX idx_outreach_sync_log_created USING btree (created_at DESC)
- UNIQUE INDEX outreach_sync_log_pkey USING btree (id)

### research.profiles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| full_name | text | null |  |
| grade_level | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| school_name | text | null |  |
| country | text | null |  |
| subject_preferences | text[] | null | '{}'::text[] |
| research_interests | text | null |  |
| onboarding_completed | boolean | null | false |

Constraints:
- `profiles_pkey` PRIMARY KEY (id)
- `profiles_user_id_key` UNIQUE (user_id)

Indexes:
- UNIQUE INDEX profiles_pkey USING btree (id)
- UNIQUE INDEX profiles_user_id_key USING btree (user_id)

### research.program_email_settings

| column | type | nullable | default |
|---|---|---|---|
| cohort_key | text | NOT NULL |  |
| application_emails_enabled | boolean | NOT NULL | true |
| payment_confirmation_enabled | boolean | NOT NULL | true |
| payment_reminder_enabled | boolean | NOT NULL | true |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_email_settings_pkey` PRIMARY KEY (cohort_key)

Indexes:
- UNIQUE INDEX program_email_settings_pkey USING btree (cohort_key)

### research.published_papers

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| title | text | NOT NULL |  |
| author_name | text | NOT NULL |  |
| school_name | text | null |  |
| grade_level | text | null |  |
| subject_area | text | NOT NULL |  |
| journal_name | text | NOT NULL |  |
| publication_year | integer | NOT NULL |  |
| publication_url | text | null |  |
| abstract | text | null |  |
| country | text | null |  |
| featured | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `published_papers_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_published_papers_featured USING btree (featured)
- INDEX idx_published_papers_subject USING btree (subject_area)
- INDEX idx_published_papers_year USING btree (publication_year)
- UNIQUE INDEX published_papers_pkey USING btree (id)

### research.questionnaire_responses

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| created_at | timestamp with time zone | NOT NULL | now() |
| student_name | text | null |  |
| student_email | text | null |  |
| parent_email | text | null |  |
| student_grade | text | null |  |
| page_url | text | null |  |
| utm_source | text | NOT NULL | 'unknown'::text |
| utm_medium | text | NOT NULL | 'unknown'::text |
| utm_campaign | text | NOT NULL | 'unknown'::text |
| enjoy_subjects_top3 | text | null |  |
| enjoy_subject_one | text | null |  |
| rating_experiments | smallint | null |  |
| rating_reading_articles | smallint | null |  |
| rating_arguing_position | smallint | null |  |
| rating_building_coding | smallint | null |  |
| rating_interviewing_people | smallint | null |  |
| rating_numbers_charts | smallint | null |  |
| subject_outside_class | text | null |  |
| noticed_problem | text | null |  |
| last_curiosity_lookup | text | null |  |
| question_type | text | null |  |
| community_problem | text | null |  |
| help_audience | text | null |  |
| writing_comfort | smallint | null |  |
| read_research_paper | text | null |  |
| tools_used | text | null |  |
| work_preference | text | null |  |
| hours_per_week | text | null |  |
| program_goals | text | null |  |
| prior_research | text | null |  |
| existing_work | text | null |  |
| topic_certainty | smallint | null |  |

Constraints:
- `questionnaire_responses_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX questionnaire_responses_pkey USING btree (id)

### research.referrals

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| referrer_application_id | uuid | null |  |
| referrer_email | text | null |  |
| referred_email | text | NOT NULL |  |
| referred_name | text | null |  |
| utm_campaign | text | null |  |
| utm_source | text | null | 'referral'::text |
| converted_application_id | uuid | null |  |
| converted_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `referrals_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_referrals_converted USING btree (converted_application_id)
- INDEX idx_referrals_referrer USING btree (referrer_application_id)
- UNIQUE INDEX referrals_pkey USING btree (id)

### research.replication_log

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| table_name | text | NOT NULL |  |
| op | text | NOT NULL |  |
| row_id | uuid | null |  |
| request_id | bigint | null |  |
| status | text | NOT NULL | 'pending'::text |
| status_code | integer | null |  |
| response | jsonb | null |  |
| error_message | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| reconciled_at | timestamp with time zone | null |  |

Constraints:
- `replication_log_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX replication_log_pkey USING btree (id)
- INDEX replication_log_request_idx USING btree (request_id)
- INDEX replication_log_status_idx USING btree (status, created_at DESC)

### research.research_projects

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| title | text | NOT NULL | 'Untitled Project'::text |
| topic | text | null |  |
| hypothesis | text | null |  |
| abstract | text | null |  |
| introduction | text | null |  |
| methodology | text | null |  |
| results | text | null |  |
| discussion | text | null |  |
| conclusion | text | null |  |
| references_text | text | null |  |
| status | text | NOT NULL | 'in_progress'::text |
| current_module | integer | NOT NULL | 1 |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `research_projects_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX research_projects_pkey USING btree (id)

### research.school_leads

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| school_name | text | NOT NULL |  |
| city | text | null |  |
| state | text | null |  |
| website_url | text | null |  |
| school_type | text | null | 'general'::text |
| estimated_enrollment | text | null |  |
| contact_email | text | null |  |
| contact_name | text | null |  |
| contact_title | text | null |  |
| outreach_status | text | NOT NULL | 'new'::text |
| notes | text | null |  |
| email_sent_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| data_source | text | NOT NULL | 'ai_generated'::text |
| website_verified | boolean | null | false |
| verified_at | timestamp with time zone | null |  |
| enrollment_band | text | null |  |
| tuition_tier | text | null |  |
| has_ap_ib | boolean | null |  |
| has_research_program | boolean | null |  |
| college_strength_tier | text | null |  |
| counselor_count | integer | null |  |
| region | text | null |  |
| timezone | text | null |  |
| icp_fit_score | integer | null |  |
| geo_score | integer | null |  |
| signal_score | integer | null |  |
| conversion_likelihood | numeric | null |  |
| predicted_paid_students | numeric | null |  |
| priority_score | integer | null |  |
| score_explanation | jsonb | null |  |
| best_contact_role | text | null |  |
| best_template_id | text | null |  |
| best_send_window | text | null |  |
| last_scored_at | timestamp with time zone | null |  |
| intelligence_version | integer | null | 1 |
| ai_summary | text | null |  |
| research_signals | text[] | null |  |
| enriched_at | timestamp with time zone | null |  |
| enrichment_status | text | null | 'pending'::text |
| penetration_score | numeric | null | 0 |
| enrolled_student_count | integer | null | 0 |
| partner_status | text | null | 'awareness'::text |
| counsellor_attended_masterclass | boolean | null | false |
| counsellor_state | text | null | 'not_contacted'::text |
| counsellor_state_changed_at | timestamp with time zone | null |  |
| programs_with_enrollments | text[] | null | '{}'::text[] |
| curriculum | text | null |  |

Constraints:
- `school_leads_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_school_leads_counsellor_state USING btree (counsellor_state)
- INDEX idx_school_leads_enrichment_status USING btree (enrichment_status)
- INDEX idx_school_leads_last_scored_at USING btree (last_scored_at)
- INDEX idx_school_leads_partner_status USING btree (partner_status)
- INDEX idx_school_leads_priority_score USING btree (priority_score DESC NULLS LAST)
- INDEX idx_school_leads_region USING btree (region)
- INDEX idx_school_leads_state USING btree (state)
- INDEX idx_school_leads_status USING btree (outreach_status)
- UNIQUE INDEX school_leads_pkey USING btree (id)

### research.school_nomination_students

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| nomination_id | uuid | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| student_position | integer | NOT NULL |  |
| student_name | text | NOT NULL |  |
| student_grade | text | NOT NULL |  |
| contact_email | text | NOT NULL |  |

Constraints:
- `school_nomination_students_nomination_id_fkey` FOREIGN KEY (nomination_id) REFERENCES research.school_nominations(id) ON DELETE CASCADE
- `school_nomination_students_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX school_nomination_students_pkey USING btree (id)

### research.school_nominations

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| created_at | timestamp with time zone | NOT NULL | now() |
| school_name | text | NOT NULL |  |
| city | text | NOT NULL |  |
| country | text | NOT NULL |  |
| nominator_name | text | NOT NULL |  |
| nominator_role | text | NOT NULL |  |
| nominator_email | text | NOT NULL |  |
| nominator_phone | text | null |  |
| student_count | integer | NOT NULL | 0 |
| program_name | text | NOT NULL | 'Sir Luther Center \| Research Publication Fellowship'::text |
| page_url | text | null |  |
| utm_source | text | NOT NULL | 'unknown'::text |
| utm_medium | text | NOT NULL | 'unknown'::text |
| utm_campaign | text | NOT NULL | 'unknown'::text |

Constraints:
- `school_nominations_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX school_nominations_pkey USING btree (id)

### research.school_partners

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| school_lead_id | uuid | null |  |
| contact_title | text | null |  |
| active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `school_partners_pkey` PRIMARY KEY (id)
- `school_partners_user_id_key` UNIQUE (user_id)

Indexes:
- UNIQUE INDEX school_partners_pkey USING btree (id)
- UNIQUE INDEX school_partners_user_id_key USING btree (user_id)

### research.school_teacher_contacts

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| school_lead_id | uuid | NOT NULL |  |
| full_name | text | null |  |
| first_name | text | null |  |
| last_name | text | null |  |
| email | text | NOT NULL |  |
| title | text | null |  |
| subject_taught | text | null |  |
| discovery_source | text | NOT NULL | 'scrape'::text |
| source_url | text | null |  |
| email_verified | boolean | NOT NULL | false |
| verification_status | text | NOT NULL | 'unknown'::text |
| outreach_status | text | NOT NULL | 'new'::text |
| smartlead_lead_id | text | null |  |
| last_contacted_at | timestamp with time zone | null |  |
| replied_at | timestamp with time zone | null |  |
| bounced_at | timestamp with time zone | null |  |
| unsubscribed_at | timestamp with time zone | null |  |
| paid_at | timestamp with time zone | null |  |
| paid_amount | numeric | null |  |
| paid_currency | text | null |  |
| notes | jsonb | null | '{}'::jsonb |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| audience | text | NOT NULL | 'cold'::text |
| import_batch_id | uuid | null |  |
| external_ref | jsonb | null | '{}'::jsonb |
| assigned_payment_link_id | uuid | null |  |
| segment_locked | boolean | NOT NULL | false |
| segment_reason | text | null |  |

Constraints:
- `school_teacher_contacts_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_stc_audience USING btree (audience)
- INDEX idx_stc_created_at USING btree (created_at DESC)
- INDEX idx_stc_discovery_source USING btree (discovery_source)
- UNIQUE INDEX idx_stc_email_audience_unique USING btree (lower(email), audience)
- INDEX idx_stc_email_verified USING btree (email_verified)
- INDEX idx_stc_import_batch USING btree (import_batch_id)
- INDEX idx_stc_outreach_status USING btree (outreach_status)
- INDEX idx_stc_school_lead_id USING btree (school_lead_id)
- INDEX idx_stc_segment_locked USING btree (segment_locked) WHERE (segment_locked = false)
- UNIQUE INDEX school_teacher_contacts_pkey USING btree (id)

### research.site_leads

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| full_name | text | null |  |
| email | text | null |  |
| phone | text | null |  |
| source | text | NOT NULL |  |
| page_path | text | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| parent_or_student | text | null |  |
| child_grade | text | null |  |
| child_name | text | null |  |
| message | text | null |  |
| metadata | jsonb | null |  |
| email_sent_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| wa_number | text | null |  |
| interest_tags | text[] | null | '{}'::text[] |
| program_recommended | text | null |  |
| nurture_stage | text | null | 'new'::text |
| is_return_visitor | boolean | null | false |
| chat_summary | text | null |  |
| asset_name | text | null |  |
| first_touch_at | timestamp with time zone | null | now() |

Constraints:
- `site_leads_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_site_leads_created_at USING btree (created_at DESC)
- INDEX idx_site_leads_email USING btree (email)
- INDEX idx_site_leads_nurture_stage USING btree (nurture_stage)
- INDEX idx_site_leads_source USING btree (source)
- UNIQUE INDEX site_leads_pkey USING btree (id)

### research.staff_invitations

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| email | text | NOT NULL |  |
| role | research.app_role | NOT NULL |  |
| invited_by | uuid | null |  |
| token | uuid | NOT NULL | gen_random_uuid() |
| school_lead_id | uuid | null |  |
| display_name | text | null |  |
| expires_at | timestamp with time zone | NOT NULL | (now() + '7 days'::interval) |
| accepted_at | timestamp with time zone | null |  |
| accepted_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `staff_invitations_pkey` PRIMARY KEY (id)

Indexes:
- INDEX staff_invitations_email_idx USING btree (lower(email))
- INDEX staff_invitations_pending_idx USING btree (email) WHERE (accepted_at IS NULL)
- UNIQUE INDEX staff_invitations_pkey USING btree (id)
- UNIQUE INDEX staff_invitations_token_idx USING btree (token)

### research.student_outcomes

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_name | text | NOT NULL |  |
| published_paper_id | uuid | null |  |
| graduation_year | integer | NOT NULL |  |
| outcome_type | text | NOT NULL | 'university_acceptance'::text |
| university_name | text | null |  |
| university_tier | text | null |  |
| acceptance_type | text | null |  |
| major | text | null |  |
| scholarship_name | text | null |  |
| scholarship_amount | numeric(10,2) | null |  |
| scholarship_type | text | null |  |
| awarding_organization | text | null |  |
| award_name | text | null |  |
| award_description | text | null |  |
| verification_status | text | NOT NULL | 'pending'::text |
| verification_notes | text | null |  |
| verified_at | timestamp with time zone | null |  |
| verified_by | uuid | null |  |
| featured | boolean | NOT NULL | false |
| display_on_website | boolean | NOT NULL | false |
| student_consent | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| created_by | uuid | null |  |

Constraints:
- `student_outcomes_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_student_outcomes_type USING btree (outcome_type)
- INDEX idx_student_outcomes_verified USING btree (verification_status, display_on_website)
- INDEX idx_student_outcomes_year USING btree (graduation_year)
- UNIQUE INDEX student_outcomes_pkey USING btree (id)

### research.suppression_list

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| email | text | NOT NULL |  |
| reason | text | NOT NULL | 'unsubscribed'::text |
| source | text | null |  |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `suppression_list_pkey` PRIMARY KEY (id)
- `suppression_list_email_key` UNIQUE (email)

Indexes:
- INDEX idx_suppression_email USING btree (lower(email))
- UNIQUE INDEX suppression_list_email_key USING btree (email)
- UNIQUE INDEX suppression_list_pkey USING btree (id)

### research.teacher_bookmarks

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| teacher_lead_id | uuid | NOT NULL |  |
| resource_type | text | NOT NULL |  |
| resource_id | text | NOT NULL |  |
| resource_title | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `teacher_bookmarks_pkey` PRIMARY KEY (id)
- `teacher_bookmarks_teacher_lead_id_resource_type_resource_id_key` UNIQUE (teacher_lead_id, resource_type, resource_id)

Indexes:
- INDEX idx_teacher_bookmarks_lead USING btree (teacher_lead_id)
- UNIQUE INDEX teacher_bookmarks_pkey USING btree (id)
- UNIQUE INDEX teacher_bookmarks_teacher_lead_id_resource_type_resource_id_key USING btree (teacher_lead_id, resource_type, resource_id)

### research.teacher_feedback

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| teacher_id | uuid | NOT NULL |  |
| student_id | uuid | NOT NULL |  |
| project_id | uuid | NOT NULL |  |
| section | text | NOT NULL |  |
| feedback | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `teacher_feedback_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX teacher_feedback_pkey USING btree (id)

### research.teacher_leads

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| email | text | NOT NULL |  |
| full_name | text | null |  |
| school_organization | text | null |  |
| grade_levels | text[] | null |  |
| downloaded_at | timestamp with time zone | null | now() |
| source | text | null | 'teacher-resources-page'::text |
| email_opt_in | boolean | null | true |
| created_at | timestamp with time zone | null | now() |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |

Constraints:
- `teacher_leads_pkey` PRIMARY KEY (id)
- `teacher_leads_email_key` UNIQUE (email)

Indexes:
- UNIQUE INDEX teacher_leads_email_key USING btree (email)
- UNIQUE INDEX teacher_leads_pkey USING btree (id)

### research.teacher_resource_access

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| teacher_lead_id | uuid | NOT NULL |  |
| resource_type | text | NOT NULL |  |
| resource_id | text | NOT NULL |  |
| resource_title | text | NOT NULL |  |
| accessed_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `teacher_resource_access_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_teacher_resource_access_lead USING btree (teacher_lead_id)
- UNIQUE INDEX teacher_resource_access_pkey USING btree (id)

### research.teacher_students

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| teacher_id | uuid | NOT NULL |  |
| student_id | uuid | null |  |
| student_email | text | NOT NULL |  |
| status | text | NOT NULL | 'pending'::text |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `teacher_students_status_check` CHECK ((status = ANY (ARRAY['pending'::text, 'accepted'::text, 'declined'::text])))
- `teacher_students_pkey` PRIMARY KEY (id)
- `teacher_students_teacher_id_student_email_key` UNIQUE (teacher_id, student_email)

Indexes:
- UNIQUE INDEX teacher_students_pkey USING btree (id)
- UNIQUE INDEX teacher_students_teacher_id_student_email_key USING btree (teacher_id, student_email)

### research.topic_finder_leads

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| full_name | text | null |  |
| email | text | NOT NULL |  |
| school_organization | text | null |  |
| grade_level | text | null |  |
| email_opt_in | boolean | null | true |
| step1_answers | jsonb | null |  |
| step2_question | text | null |  |
| pico_data | jsonb | null |  |
| source | text | null | 'topic-finder'::text |
| created_at | timestamp with time zone | null | now() |
| wants_session | boolean | null | false |
| referrer_lead_id | uuid | null |  |
| share_count | integer | NOT NULL | 0 |
| step3_full_results | jsonb | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |

Constraints:
- `topic_finder_leads_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_topic_finder_leads_referrer USING btree (referrer_lead_id)
- UNIQUE INDEX topic_finder_leads_pkey USING btree (id)

### research.user_roles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| role | research.app_role | NOT NULL |  |
| mentor_id | uuid | null |  |

Constraints:
- `user_roles_mentor_id_fkey` FOREIGN KEY (mentor_id) REFERENCES all_mentors(id) ON DELETE SET NULL
- `user_roles_pkey` PRIMARY KEY (id)
- `user_roles_user_id_role_key` UNIQUE (user_id, role)

Indexes:
- INDEX user_roles_mentor_id_idx USING btree (mentor_id)
- UNIQUE INDEX user_roles_pkey USING btree (id)
- UNIQUE INDEX user_roles_user_id_role_key USING btree (user_id, role)

### research.webhook_log

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| event_type | text | NOT NULL |  |
| payload | jsonb | null |  |
| destination | text | null |  |
| status | text | NOT NULL | 'pending'::text |
| status_code | integer | null |  |
| error_message | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `webhook_log_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_webhook_log_created_at USING btree (created_at DESC)
- INDEX idx_webhook_log_event_type USING btree (event_type)
- UNIQUE INDEX webhook_log_pkey USING btree (id)

### research.whatsapp_messages

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| application_id | uuid | null |  |
| flow | text | NOT NULL |  |
| audience | text | NOT NULL |  |
| to_phone | text | NOT NULL |  |
| template_name | text | null |  |
| broadcast_name | text | null |  |
| status | text | NOT NULL |  |
| error | text | null |  |
| provider_response | jsonb | null |  |
| idempotency_key | text | null |  |
| sent_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `whatsapp_messages_pkey` PRIMARY KEY (id)
- `whatsapp_messages_idempotency_key_key` UNIQUE (idempotency_key)

Indexes:
- INDEX whatsapp_messages_application_idx USING btree (application_id)
- UNIQUE INDEX whatsapp_messages_idempotency_key_key USING btree (idempotency_key)
- UNIQUE INDEX whatsapp_messages_pkey USING btree (id)
- INDEX whatsapp_messages_sent_at_idx USING btree (sent_at DESC)

### research.whatsapp_settings

| column | type | nullable | default |
|---|---|---|---|
| key | text | NOT NULL |  |
| value | jsonb | NOT NULL | '{}'::jsonb |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `whatsapp_settings_pkey` PRIMARY KEY (key)

Indexes:
- UNIQUE INDEX whatsapp_settings_pkey USING btree (key)
