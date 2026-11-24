# Shared estate columns: premed (40 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### premed.aeo_keywords

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| page_slug | text | NOT NULL |  |
| keyword | text | NOT NULL |  |
| kind | text | NOT NULL | 'secondary'::text |
| intent | text | null |  |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `aeo_keywords_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX aeo_keywords_pkey USING btree (id)
- INDEX aeo_keywords_slug_idx USING btree (page_slug)

### premed.aeo_outputs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| keyword | text | NOT NULL |  |
| intent | text | null |  |
| market | text | null |  |
| funnel_stage | text | null |  |
| grade_band | text | null |  |
| target_slug | text | NOT NULL |  |
| faq_question | text | NOT NULL | ''::text |
| faq_answer | text | NOT NULL | ''::text |
| meta_description | text | NOT NULL | ''::text |
| confidence | integer | NOT NULL | 0 |
| status | text | NOT NULL | 'pending'::text |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `aeo_outputs_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX aeo_outputs_pkey USING btree (id)

### premed.aeo_page_optimizations

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| page_slug | text | NOT NULL |  |
| page_label | text | NOT NULL | ''::text |
| page_type | text | NOT NULL | 'page'::text |
| meta_title | text | null |  |
| meta_description | text | null |  |
| og_title | text | null |  |
| og_description | text | null |  |
| meta_status | text | NOT NULL | 'not_generated'::text |
| content_analysis | jsonb | null |  |
| content_status | text | NOT NULL | 'not_analyzed'::text |
| last_run_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `aeo_page_optimizations_pkey` PRIMARY KEY (id)
- `aeo_page_optimizations_page_slug_key` UNIQUE (page_slug)

Indexes:
- UNIQUE INDEX aeo_page_optimizations_page_slug_key USING btree (page_slug)
- UNIQUE INDEX aeo_page_optimizations_pkey USING btree (id)

### premed.aeo_pages

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| label | text | NOT NULL |  |
| slug | text | NOT NULL |  |
| path | text | NOT NULL |  |
| status | text | NOT NULL | 'draft'::text |
| title | text | null |  |
| meta_description | text | null |  |
| schema | jsonb | null |  |
| meta_tags | jsonb | null |  |
| answer_snippets | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `aeo_pages_pkey` PRIMARY KEY (id)
- `aeo_pages_slug_key` UNIQUE (slug)

Indexes:
- UNIQUE INDEX aeo_pages_pkey USING btree (id)
- UNIQUE INDEX aeo_pages_slug_key USING btree (slug)

### premed.aeo_prompts

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| page_slug | text | NOT NULL |  |
| prompt | text | NOT NULL |  |
| rationale | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `aeo_prompts_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX aeo_prompts_pkey USING btree (id)
- INDEX aeo_prompts_slug_idx USING btree (page_slug)

### premed.aeo_settings

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| entity_context | text | NOT NULL | ''::text |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `aeo_settings_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX aeo_settings_pkey USING btree (id)

### premed.app_settings

| column | type | nullable | default |
|---|---|---|---|
| key | text | NOT NULL |  |
| value | jsonb | NOT NULL | '{}'::jsonb |
| updated_at | timestamp with time zone | NOT NULL | now() |
| updated_by | uuid | null |  |

Constraints:
- `app_settings_pkey` PRIMARY KEY (key)

Indexes:
- UNIQUE INDEX app_settings_pkey USING btree (key)

### premed.captured_leads

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| parent_name | text | null |  |
| email | text | NOT NULL |  |
| phone | text | null |  |
| picture | text | null |  |
| google_sub | text | null |  |
| channel | text | NOT NULL | 'google_one_tap'::text |
| kind | text | NOT NULL | 'visitor'::text |
| source_url | text | null |  |
| user_agent | text | null |  |
| country | text | null |  |
| city | text | null |  |
| timezone | text | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| captured_at | timestamp with time zone | NOT NULL | now() |
| last_emailed_at | timestamp with time zone | null |  |
| emails_sent_count | integer | NOT NULL | 0 |
| nurture_status | text | NOT NULL | 'new'::text |
| converted_application_id | uuid | null |  |
| unsubscribe_token | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| grade | text | null |  |
| interest | text | null |  |
| stage | text | null |  |
| concierge_session_id | text | null |  |
| intent_score | integer | NOT NULL | 0 |
| intent_tier | text | null |  |
| hot_alert_sent_at | timestamp with time zone | null |  |

Constraints:
- `captured_leads_intent_tier_check` CHECK (((intent_tier IS NULL) OR (intent_tier = ANY (ARRAY['hot'::text, 'warm'::text, 'cold'::text]))))
- `captured_leads_pkey` PRIMARY KEY (id)
- `captured_leads_unsubscribe_token_key` UNIQUE (unsubscribe_token)

Indexes:
- INDEX captured_leads_concierge_session_idx USING btree (concierge_session_id)
- INDEX captured_leads_email_idx USING btree (lower(email))
- UNIQUE INDEX captured_leads_email_key USING btree (email)
- INDEX captured_leads_intent_tier_idx USING btree (intent_tier)
- UNIQUE INDEX captured_leads_pkey USING btree (id)
- INDEX captured_leads_status_idx USING btree (nurture_status)
- UNIQUE INDEX captured_leads_unsubscribe_token_key USING btree (unsubscribe_token)

### premed.challenge_quiz_attempts

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| attempt_id | uuid | NOT NULL |  |
| quiz_week | integer | NOT NULL |  |
| student_name | text | NOT NULL |  |
| student_email | text | NOT NULL |  |
| answers | jsonb | NOT NULL | '[]'::jsonb |
| score | integer | NOT NULL |  |
| total | integer | NOT NULL |  |
| time_seconds | integer | null |  |
| rank_title | text | null |  |
| email_status | text | NOT NULL | 'pending'::text |
| email_error | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `challenge_quiz_attempts_pkey` PRIMARY KEY (id)
- `challenge_quiz_attempts_attempt_id_key` UNIQUE (attempt_id)

Indexes:
- UNIQUE INDEX challenge_quiz_attempts_attempt_id_key USING btree (attempt_id)
- INDEX challenge_quiz_attempts_email_idx USING btree (lower(student_email), created_at DESC)
- UNIQUE INDEX challenge_quiz_attempts_pkey USING btree (id)

### premed.cohort_settings

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | NOT NULL |  |
| program_key | text | NOT NULL |  |
| cohort_key | text | NOT NULL |  |
| price_cents | integer | NOT NULL | 0 |
| currency | text | NOT NULL | 'GBP'::text |
| capacity | integer | NOT NULL | 20 |
| payment_link_url | text | null |  |
| mentor_id | uuid | null |  |
| is_active | boolean | NOT NULL | true |
| sort_order | integer | NOT NULL | 0 |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `cohort_settings_cohort_id_fkey` FOREIGN KEY (cohort_id) REFERENCES all_cohorts(id) ON DELETE CASCADE
- `cohort_settings_mentor_id_fkey` FOREIGN KEY (mentor_id) REFERENCES auth.users(id) ON DELETE SET NULL
- `cohort_settings_pkey` PRIMARY KEY (id)
- `cohort_settings_cohort_id_key` UNIQUE (cohort_id)

Indexes:
- UNIQUE INDEX cohort_settings_cohort_id_key USING btree (cohort_id)
- UNIQUE INDEX cohort_settings_cohort_key_idx USING btree (cohort_key)
- UNIQUE INDEX cohort_settings_pkey USING btree (id)

### premed.concierge_ai_usage

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| usage_day | date | NOT NULL |  |
| calls | integer | NOT NULL | 0 |
| prompt_tokens | bigint | NOT NULL | 0 |
| completion_tokens | bigint | NOT NULL | 0 |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `concierge_ai_usage_pkey` PRIMARY KEY (id)
- `concierge_ai_usage_day_unique` UNIQUE (usage_day)

Indexes:
- UNIQUE INDEX concierge_ai_usage_day_unique USING btree (usage_day)
- UNIQUE INDEX concierge_ai_usage_pkey USING btree (id)

### premed.concierge_conversations

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | text | NOT NULL |  |
| messages | jsonb | NOT NULL | '[]'::jsonb |
| flagged | boolean | NOT NULL | false |
| flag_reason | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| user_id | uuid | null |  |
| tier | text | NOT NULL | 'public'::text |
| feedback | jsonb | NOT NULL | '{}'::jsonb |
| recap_sent_at | timestamp with time zone | null |  |
| profile | jsonb | NOT NULL | '{}'::jsonb |

Constraints:
- `concierge_conversations_pkey` PRIMARY KEY (id)

Indexes:
- INDEX concierge_conversations_created_idx USING btree (created_at DESC)
- UNIQUE INDEX concierge_conversations_pkey USING btree (id)
- UNIQUE INDEX concierge_conversations_session_idx USING btree (session_id)
- INDEX concierge_conversations_user_id_idx USING btree (user_id, updated_at DESC)

### premed.concierge_faqs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| question | text | NOT NULL |  |
| answer | text | NOT NULL |  |
| source_label | text | null |  |
| asked_count | integer | NOT NULL | 0 |
| sort_order | integer | NOT NULL | 100 |
| is_published | boolean | NOT NULL | false |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `concierge_faqs_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX concierge_faqs_pkey USING btree (id)
- INDEX concierge_faqs_published_idx USING btree (is_published, sort_order, created_at)

### premed.concierge_rate_limits

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| client_key | text | NOT NULL |  |
| window_start | timestamp with time zone | NOT NULL |  |
| request_count | integer | NOT NULL | 1 |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `concierge_rate_limits_pkey` PRIMARY KEY (id)

Indexes:
- INDEX concierge_rate_limits_key_idx USING btree (client_key)
- UNIQUE INDEX concierge_rate_limits_key_window_idx USING btree (client_key, window_start)
- UNIQUE INDEX concierge_rate_limits_pkey USING btree (id)

### premed.email_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| subject | text | NOT NULL |  |
| html_body | text | NOT NULL |  |
| body_text | text | null |  |
| description | text | null |  |
| variables | text[] | null |  |
| trigger_type | text | NOT NULL | 'manual'::text |
| sender_name | text | NOT NULL | 'Pre-Med Platform'::text |
| is_active | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `email_templates_pkey` PRIMARY KEY (id)
- `email_templates_name_key` UNIQUE (name)

Indexes:
- UNIQUE INDEX email_templates_name_key USING btree (name)
- UNIQUE INDEX email_templates_pkey USING btree (id)

### premed.fellowship_projects

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| title | text | NOT NULL |  |
| track | text | NOT NULL | 'research_study'::text |
| description | text | null |  |
| student_name | text | NOT NULL | ''::text |
| cohort_name | text | null |  |
| year | integer | NOT NULL | 2026 |
| project_url | text | null |  |
| mentor_notes | text | null |  |
| is_featured | boolean | NOT NULL | false |
| is_active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `fellowship_projects_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX fellowship_projects_pkey USING btree (id)

### premed.goal_reviews

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| mentor_id | uuid | null |  |
| period_start | date | NOT NULL | ((now())::date - 30) |
| period_end | date | NOT NULL | (now())::date |
| summary | text | NOT NULL |  |
| strengths | text | null |  |
| focus_areas | text | null |  |
| mentor_rating | integer | null |  |
| is_published | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `goal_reviews_mentor_id_fkey` FOREIGN KEY (mentor_id) REFERENCES auth.users(id) ON DELETE SET NULL
- `goal_reviews_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `goal_reviews_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX goal_reviews_pkey USING btree (id)
- INDEX goal_reviews_user_idx USING btree (user_id, period_end DESC)

### premed.intake_raw

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| form_type | text | NOT NULL |  |
| idempotency_key | text | NOT NULL |  |
| payload | jsonb | NOT NULL |  |
| status | text | NOT NULL | 'received'::text |
| application_id | uuid | null |  |
| error | text | null |  |
| ip | text | null |  |
| user_agent | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `intake_raw_pkey` PRIMARY KEY (id)
- `intake_raw_idempotency_key_key` UNIQUE (idempotency_key)

Indexes:
- UNIQUE INDEX intake_raw_idempotency_key_key USING btree (idempotency_key)
- UNIQUE INDEX intake_raw_pkey USING btree (id)
- INDEX intake_raw_status_idx USING btree (status, created_at)

### premed.knowledge_entries

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| domain | text | NOT NULL |  |
| subdomain | text | NOT NULL |  |
| region | text | null |  |
| school | text | null |  |
| pathway | text | null |  |
| claim | text | NOT NULL |  |
| claim_type | text | NOT NULL | 'EXACT'::text |
| value | text | null |  |
| value_low | numeric | null |  |
| value_high | numeric | null |  |
| unit | text | null |  |
| confidence | text | NOT NULL | 'MEDIUM'::text |
| source_type | text | null |  |
| source_name | text | null |  |
| source_url | text | null |  |
| verified_by | text | null |  |
| verified_date | date | null |  |
| cycle | text | null |  |
| expires | date | NOT NULL | (CURRENT_DATE + 365) |
| tags | text[] | null |  |
| grade_phases | text[] | null |  |
| display_text | text | NOT NULL |  |
| counselor_note | text | null |  |
| contradictions | text[] | null |  |
| search_vector | tsvector | null |  |
| active | boolean | null | true |
| created_at | timestamp with time zone | null | now() |
| updated_at | timestamp with time zone | null | now() |
| audience | text | null | 'student_only'::text |

Constraints:
- `knowledge_entries_audience_chk` CHECK (((audience IS NULL) OR (audience = ANY (ARRAY['public'::text, 'student_only'::text]))))
- `knowledge_entries_pkey` PRIMARY KEY (id)

Indexes:
- INDEX knowledge_entries_audience_idx USING btree (audience)
- INDEX knowledge_entries_filter_idx USING btree (domain, region, pathway) WHERE active
- UNIQUE INDEX knowledge_entries_pkey USING btree (id)
- INDEX knowledge_entries_search_idx USING gin (search_vector)

### premed.learning_assessments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| assessment_type | text | NOT NULL | 'module'::text |
| module_id | text | null |  |
| module_title | text | null |  |
| score | integer | null |  |
| max_score | integer | NOT NULL | 100 |
| answers | jsonb | null | '[]'::jsonb |
| time_taken_seconds | integer | null |  |
| submitted_at | timestamp with time zone | NOT NULL | now() |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `learning_assessments_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `learning_assessments_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX learning_assessments_pkey USING btree (id)
- INDEX learning_assessments_user_idx USING btree (user_id, submitted_at DESC)

### premed.live_class_rsvps

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| session_id | uuid | NOT NULL |  |
| status | text | NOT NULL | 'going'::text |
| joined_at | timestamp with time zone | null |  |
| left_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `live_class_rsvps_session_id_fkey` FOREIGN KEY (session_id) REFERENCES all_masterclasses(id) ON DELETE CASCADE
- `live_class_rsvps_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `live_class_rsvps_pkey` PRIMARY KEY (id)
- `live_class_rsvps_user_id_session_id_key` UNIQUE (user_id, session_id)

Indexes:
- UNIQUE INDEX live_class_rsvps_pkey USING btree (id)
- UNIQUE INDEX live_class_rsvps_user_id_session_id_key USING btree (user_id, session_id)

### premed.masterclass_series_overrides

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| masterclass_id | uuid | NOT NULL |  |
| series_name | text | NOT NULL |  |
| created_at | timestamp with time zone | null | now() |
| updated_at | timestamp with time zone | null | now() |

Constraints:
- `masterclass_series_overrides_masterclass_id_fkey` FOREIGN KEY (masterclass_id) REFERENCES all_masterclasses(id) ON DELETE CASCADE
- `masterclass_series_overrides_pkey` PRIMARY KEY (id)
- `masterclass_series_overrides_masterclass_id_key` UNIQUE (masterclass_id)

Indexes:
- UNIQUE INDEX masterclass_series_overrides_masterclass_id_key USING btree (masterclass_id)
- UNIQUE INDEX masterclass_series_overrides_pkey USING btree (id)

### premed.mmi_coach_sessions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| station_id | text | null |  |
| scenario_question | text | NOT NULL |  |
| scenario_category | text | null |  |
| scenario_difficulty | text | null |  |
| conversation | jsonb | NOT NULL | '[]'::jsonb |
| coach_chat | jsonb | NOT NULL | '[]'::jsonb |
| report | jsonb | null |  |
| status | text | NOT NULL | 'in_progress'::text |
| duration_seconds | integer | null |  |
| remaining_seconds | integer | null |  |
| tavus_conversation_id | text | null |  |
| tavus_conversation_url | text | null |  |
| tavus_started_at | timestamp with time zone | null |  |
| tavus_ended_at | timestamp with time zone | null |  |
| started_at | timestamp with time zone | NOT NULL | now() |
| completed_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `mmi_coach_sessions_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `mmi_coach_sessions_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX mmi_coach_sessions_pkey USING btree (id)
- INDEX mmi_coach_sessions_user_idx USING btree (user_id, started_at DESC)

### premed.mmi_practice_attempts

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| question_id | text | null |  |
| self_rating | integer | null |  |
| response_text | text | null |  |
| ethics_score | numeric | null |  |
| communication_score | numeric | null |  |
| motivation_score | numeric | null |  |
| knowledge_score | numeric | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `mmi_practice_attempts_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `mmi_practice_attempts_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX mmi_practice_attempts_pkey USING btree (id)
- INDEX mmi_practice_attempts_user_idx USING btree (user_id, created_at DESC)

### premed.parent_reports

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | null |  |
| report_type | text | NOT NULL |  |
| attendance_summary | jsonb | null |  |
| assessment_summary | jsonb | null |  |
| project_summary | jsonb | null |  |
| mentor_comments | text | null |  |
| strengths | jsonb | null |  |
| areas_for_growth | jsonb | null |  |
| overall_rating | integer | null |  |
| generated_at | timestamp with time zone | NOT NULL | now() |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `parent_reports_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `parent_reports_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX parent_reports_pkey USING btree (id)
- INDEX parent_reports_user_idx USING btree (user_id, generated_at DESC)

### premed.program_attachments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_key | text | NOT NULL |  |
| file_path | text | NOT NULL |  |
| file_name | text | NOT NULL |  |
| output_filename_template | text | NOT NULL | '{name}.pdf'::text |
| name_page | integer | NOT NULL | 1 |
| name_x | double precision | NOT NULL | 0 |
| name_y | double precision | NOT NULL | 0 |
| font_family | text | NOT NULL | 'Helvetica'::text |
| font_storage_path | text | null |  |
| font_size | integer | NOT NULL | 24 |
| font_color | text | NOT NULL | '#000000'::text |
| name_prefix | text | NOT NULL | ''::text |
| name_suffix | text | NOT NULL | ''::text |
| x_offset | double precision | NOT NULL | 0 |
| y_offset | double precision | NOT NULL | 0 |
| is_active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_attachments_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX program_attachments_pkey USING btree (id)
- INDEX program_attachments_program_idx USING btree (program_key, is_active)

### premed.program_email_settings

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| category | text | NOT NULL |  |
| application_emails_enabled | boolean | NOT NULL | true |
| payment_emails_enabled | boolean | NOT NULL | true |
| reminder_emails_enabled | boolean | NOT NULL | true |
| letter_of_acceptance_enabled | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_email_settings_pkey` PRIMARY KEY (id)
- `program_email_settings_category_key` UNIQUE (category)

Indexes:
- UNIQUE INDEX program_email_settings_category_key USING btree (category)
- UNIQUE INDEX program_email_settings_pkey USING btree (id)

### premed.registration_attempts

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| ip_hash | text | NOT NULL |  |
| email_hash | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `registration_attempts_pkey` PRIMARY KEY (id)

Indexes:
- INDEX registration_attempts_email_hash_created_at_idx USING btree (email_hash, created_at)
- INDEX registration_attempts_ip_hash_created_at_idx USING btree (ip_hash, created_at)
- UNIQUE INDEX registration_attempts_pkey USING btree (id)

### premed.saved_universities

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_id | uuid | NOT NULL |  |
| university_id | uuid | null |  |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `saved_universities_student_id_fkey` FOREIGN KEY (student_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `saved_universities_university_id_fkey` FOREIGN KEY (university_id) REFERENCES premed.universities(id) ON DELETE CASCADE
- `saved_universities_pkey` PRIMARY KEY (id)
- `saved_universities_student_id_university_id_key` UNIQUE (student_id, university_id)

Indexes:
- UNIQUE INDEX saved_universities_pkey USING btree (id)
- UNIQUE INDEX saved_universities_student_id_university_id_key USING btree (student_id, university_id)

### premed.sent_emails

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| template_name | text | NOT NULL |  |
| recipient_email | text | NOT NULL |  |
| cc_emails | text[] | null |  |
| subject | text | NOT NULL |  |
| application_id | uuid | null |  |
| status | text | NOT NULL |  |
| error_message | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `sent_emails_pkey` PRIMARY KEY (id)

Indexes:
- INDEX sent_emails_application_idx USING btree (application_id)
- UNIQUE INDEX sent_emails_pkey USING btree (id)
- INDEX sent_emails_recipient_idx USING btree (lower(recipient_email))
- INDEX sent_emails_template_idx USING btree (template_name, created_at DESC)

### premed.session_attendance

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| session_date | date | NOT NULL |  |
| attended | boolean | NOT NULL | false |
| participation_score | integer | null |  |
| mentor_notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `session_attendance_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `session_attendance_pkey` PRIMARY KEY (id)
- `session_attendance_user_id_session_date_key` UNIQUE (user_id, session_date)

Indexes:
- UNIQUE INDEX session_attendance_pkey USING btree (id)
- UNIQUE INDEX session_attendance_user_id_session_date_key USING btree (user_id, session_date)

### premed.side_effects_queue

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| kind | text | NOT NULL |  |
| application_id | uuid | null |  |
| payload | jsonb | NOT NULL | '{}'::jsonb |
| status | text | NOT NULL | 'pending'::text |
| attempts | integer | NOT NULL | 0 |
| max_attempts | integer | NOT NULL | 5 |
| next_run_at | timestamp with time zone | NOT NULL | now() |
| last_error | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| completed_at | timestamp with time zone | null |  |

Constraints:
- `side_effects_queue_pkey` PRIMARY KEY (id)

Indexes:
- INDEX side_effects_queue_due_idx USING btree (status, next_run_at)
- UNIQUE INDEX side_effects_queue_pkey USING btree (id)

### premed.stripe_charge_cache

| column | type | nullable | default |
|---|---|---|---|
| charge_id | text | NOT NULL |  |
| payment_intent_id | text | null |  |
| product_names | jsonb | NOT NULL | '[]'::jsonb |
| program_key | text | null |  |
| is_ours | boolean | NOT NULL | false |
| amount_cents | bigint | null |  |
| currency | text | null |  |
| email | text | null |  |
| paid_at | timestamp with time zone | null |  |
| seen_count | integer | NOT NULL | 1 |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `stripe_charge_cache_pkey` PRIMARY KEY (charge_id)

Indexes:
- INDEX stripe_charge_cache_ours_idx USING btree (is_ours, paid_at DESC)
- INDEX stripe_charge_cache_pi_idx USING btree (payment_intent_id)
- UNIQUE INDEX stripe_charge_cache_pkey USING btree (charge_id)

### premed.stripe_reconcile_state

| column | type | nullable | default |
|---|---|---|---|
| application_id | uuid | NOT NULL |  |
| last_checked_at | timestamp with time zone | NOT NULL | now() |
| checks | integer | NOT NULL | 1 |
| last_result | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `stripe_reconcile_state_pkey` PRIMARY KEY (application_id)

Indexes:
- INDEX stripe_reconcile_state_checked_idx USING btree (last_checked_at)
- UNIQUE INDEX stripe_reconcile_state_pkey USING btree (application_id)

### premed.student_goals

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| category | text | NOT NULL |  |
| title | text | NOT NULL |  |
| description | text | null |  |
| due_date | date | null |  |
| progress | integer | NOT NULL | 0 |
| status | text | NOT NULL | 'active'::text |
| sub_tasks | jsonb | NOT NULL | '[]'::jsonb |
| sort_order | integer | NOT NULL | 0 |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `student_goals_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `student_goals_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX student_goals_pkey USING btree (id)
- INDEX student_goals_user_idx USING btree (user_id, sort_order)

### premed.student_notifications

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| category | text | NOT NULL |  |
| tone | text | NOT NULL | 'navy'::text |
| title | text | NOT NULL |  |
| body | text | null |  |
| link_url | text | null |  |
| is_read | boolean | NOT NULL | false |
| read_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `student_notifications_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `student_notifications_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX student_notifications_pkey USING btree (id)
- INDEX student_notifications_user_idx USING btree (user_id, created_at DESC)

### premed.student_university_applications

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_id | uuid | NOT NULL |  |
| university_id | uuid | null |  |
| app_status | text | NOT NULL | 'planning'::text |
| applied_date | date | null |  |
| decision_date | date | null |  |
| documents_submitted | boolean | NOT NULL | false |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `student_university_applications_student_id_fkey` FOREIGN KEY (student_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `student_university_applications_university_id_fkey` FOREIGN KEY (university_id) REFERENCES premed.universities(id) ON DELETE SET NULL
- `student_university_applications_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX student_university_applications_pkey USING btree (id)
- INDEX sua_student_idx USING btree (student_id)

### premed.universities

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| program_name | text | NOT NULL |  |
| country | text | NOT NULL |  |
| city | text | NOT NULL |  |
| tuition_annual | integer | NOT NULL |  |
| currency | text | NOT NULL | 'GBP'::text |
| hero_image_url | text | null |  |
| entry_requirements | text | null |  |
| website_url | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `universities_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX universities_pkey USING btree (id)

### premed.user_profiles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| role | premed.app_role | NOT NULL | 'student'::premed.app_role |
| email | text | NOT NULL |  |
| full_name | text | null |  |
| phone | text | null |  |
| mentor_name | text | null |  |
| current_grade | text | null |  |
| curriculum | premed.curriculum_type | null |  |
| predicted_grades | text | null |  |
| science_subjects | text[] | null |  |
| exam_preference | premed.exam_preference | null |  |
| work_experience | premed.work_experience_status | null |  |
| budget_min | integer | null |  |
| budget_max | integer | null |  |
| preferred_countries | text[] | null |  |
| profile_completed | boolean | null | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `user_profiles_user_id_fkey` FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- `user_profiles_pkey` PRIMARY KEY (id)
- `user_profiles_user_id_key` UNIQUE (user_id)

Indexes:
- INDEX user_profiles_email_idx USING btree (lower(email))
- UNIQUE INDEX user_profiles_pkey USING btree (id)
- INDEX user_profiles_role_idx USING btree (role)
- UNIQUE INDEX user_profiles_user_id_key USING btree (user_id)

### premed.whatsapp_messages

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| template_name | text | NOT NULL |  |
| recipient_phone | text | NOT NULL |  |
| recipient_name | text | null |  |
| application_id | uuid | null |  |
| kind | text | NOT NULL | 'transactional'::text |
| parameters | jsonb | NOT NULL | '{}'::jsonb |
| status | text | NOT NULL | 'sent'::text |
| error_message | text | null |  |
| provider_message_id | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `whatsapp_messages_pkey` PRIMARY KEY (id)

Indexes:
- INDEX whatsapp_messages_app_idx USING btree (application_id)
- INDEX whatsapp_messages_created_idx USING btree (created_at DESC)
- UNIQUE INDEX whatsapp_messages_pkey USING btree (id)
