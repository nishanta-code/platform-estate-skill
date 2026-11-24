# Shared estate columns: testprep (20 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### testprep.captured_leads

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| email | text | NOT NULL |  |
| name | text | null |  |
| picture | text | null |  |
| google_sub | text | null |  |
| country | text | null |  |
| city | text | null |  |
| region | text | null |  |
| ip | text | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| page_url | text | null |  |
| referrer | text | null |  |
| user_agent | text | null |  |
| language | text | null |  |
| timezone | text | null |  |
| screen | text | null |  |
| viewport | text | null |  |
| dwell_seconds | integer | null |  |
| session_started_at | timestamp with time zone | null |  |
| captured_at | timestamp with time zone | NOT NULL | now() |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| utm_content | text | null | 'unknown'::text |
| utm_term | text | null | 'unknown'::text |

Constraints:
- `captured_leads_pkey` PRIMARY KEY (id)

Indexes:
- INDEX captured_leads_created_at_idx USING btree (created_at DESC)
- UNIQUE INDEX captured_leads_email_unique USING btree (lower(email))
- UNIQUE INDEX captured_leads_pkey USING btree (id)

### testprep.country_dial_codes

| column | type | nullable | default |
|---|---|---|---|
| country_name | text | NOT NULL |  |
| dial_code | text | NOT NULL |  |
| is_public | boolean | NOT NULL | false |

Constraints:
- `country_dial_codes_pkey` PRIMARY KEY (country_name)

Indexes:
- UNIQUE INDEX country_dial_codes_pkey USING btree (country_name)

### testprep.drill_responses

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | uuid | NOT NULL |  |
| user_id | uuid | NOT NULL |  |
| question_id | uuid | NOT NULL |  |
| selected_answer_index | integer | NOT NULL |  |
| is_correct | boolean | NOT NULL |  |
| trap_type_hit | testprep.trap_type | null |  |
| time_taken_ms | integer | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `drill_responses_question_id_fkey` FOREIGN KEY (question_id) REFERENCES testprep.questions(id) ON DELETE CASCADE
- `drill_responses_session_id_fkey` FOREIGN KEY (session_id) REFERENCES testprep.drill_sessions(id) ON DELETE CASCADE
- `drill_responses_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX drill_responses_pkey USING btree (id)
- INDEX idx_drill_responses_user USING btree (user_id, created_at DESC)

### testprep.drill_sessions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| started_at | timestamp with time zone | NOT NULL | now() |
| ended_at | timestamp with time zone | null |  |
| total_questions | integer | NOT NULL | 0 |
| correct_answers | integer | NOT NULL | 0 |

Constraints:
- `drill_sessions_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX drill_sessions_pkey USING btree (id)
- INDEX idx_drill_sessions_user USING btree (user_id, started_at DESC)

### testprep.email_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| subject | text | NOT NULL |  |
| html_body | text | NOT NULL |  |
| body_text | text | null |  |
| description | text | null |  |
| trigger_type | text | NOT NULL |  |
| sender_name | text | NOT NULL | 'SAT Instinct'::text |
| is_active | boolean | NOT NULL | false |
| variables | jsonb | null | '[]'::jsonb |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `email_templates_pkey` PRIMARY KEY (id)
- `email_templates_name_key` UNIQUE (name)

Indexes:
- UNIQUE INDEX email_templates_name_key USING btree (name)
- UNIQUE INDEX email_templates_pkey USING btree (id)

### testprep.homework_submissions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| week | integer | NOT NULL |  |
| assignment_type | text | NOT NULL |  |
| content | jsonb | null |  |
| submitted_at | timestamp with time zone | NOT NULL | now() |
| reviewed_at | timestamp with time zone | null |  |
| feedback | text | null |  |
| score | integer | null |  |

Constraints:
- `homework_submissions_score_check` CHECK (((score >= 0) AND (score <= 100)))
- `homework_submissions_week_check` CHECK (((week >= 1) AND (week <= 4)))
- `homework_submissions_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX homework_submissions_pkey USING btree (id)
- INDEX idx_homework_user_week USING btree (user_id, week)

### testprep.lead_events

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| lead_id | uuid | null |  |
| session_id | text | NOT NULL |  |
| event_type | testprep.lead_event_type | NOT NULL |  |
| event_data | jsonb | null | '{}'::jsonb |
| page_url | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| user_agent | text | null |  |
| referrer | text | null |  |
| screen_width | integer | null |  |
| screen_height | integer | null |  |

Constraints:
- `lead_events_lead_id_fkey` FOREIGN KEY (lead_id) REFERENCES testprep.leads(id) ON DELETE SET NULL
- `lead_events_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_lead_events_created_at USING btree (created_at)
- INDEX idx_lead_events_event_type USING btree (event_type)
- INDEX idx_lead_events_lead_id USING btree (lead_id)
- INDEX idx_lead_events_session_id USING btree (session_id)
- UNIQUE INDEX lead_events_pkey USING btree (id)

### testprep.leads

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| email | text | null |  |
| name | text | null |  |
| lead_type | testprep.lead_type | null |  |
| status | testprep.lead_status | NOT NULL | 'partial'::testprep.lead_status |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| referrer_url | text | null |  |
| landing_page | text | null |  |
| session_id | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| converted_at | timestamp with time zone | null |  |
| utm_content | text | null | 'unknown'::text |
| utm_term | text | null | 'unknown'::text |

Constraints:
- `leads_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_leads_created_at USING btree (created_at DESC)
- INDEX idx_leads_email USING btree (email)
- INDEX idx_leads_session_id USING btree (session_id)
- INDEX idx_leads_status USING btree (status)
- UNIQUE INDEX leads_pkey USING btree (id)

### testprep.program_email_settings

| column | type | nullable | default |
|---|---|---|---|
| category | text | NOT NULL |  |
| application_emails_enabled | boolean | NOT NULL | true |
| payment_emails_enabled | boolean | NOT NULL | true |
| reminder_emails_enabled | boolean | NOT NULL | true |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_email_settings_pkey` PRIMARY KEY (category)

Indexes:
- UNIQUE INDEX program_email_settings_pkey USING btree (category)

### testprep.questions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| subject | testprep.question_subject | NOT NULL |  |
| stem | text | NOT NULL |  |
| choices | jsonb | NOT NULL |  |
| correct_answer_index | integer | NOT NULL |  |
| difficulty | integer | NOT NULL | 1 |
| trap_types | testprep.trap_type[] | NOT NULL |  |
| instinct_script | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `questions_difficulty_check` CHECK (((difficulty >= 1) AND (difficulty <= 3)))
- `questions_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX questions_pkey USING btree (id)

### testprep.sat_questions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| slug | text | NOT NULL |  |
| question | text | NOT NULL |  |
| short_answer | text | NOT NULL |  |
| full_answer | text | NOT NULL |  |
| category | text | NOT NULL | 'general'::text |
| related_trap | text | null |  |
| meta_title | text | null |  |
| meta_description | text | null |  |
| keywords | text[] | null | '{}'::text[] |
| search_volume | integer | null | 0 |
| published | boolean | NOT NULL | false |
| published_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `sat_questions_pkey` PRIMARY KEY (id)
- `sat_questions_slug_key` UNIQUE (slug)

Indexes:
- INDEX idx_sat_questions_category USING btree (category)
- INDEX idx_sat_questions_published USING btree (published)
- INDEX idx_sat_questions_slug USING btree (slug)
- UNIQUE INDEX sat_questions_pkey USING btree (id)
- UNIQUE INDEX sat_questions_slug_key USING btree (slug)

### testprep.school_inquiries

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| school_name | text | NOT NULL |  |
| contact_name | text | NOT NULL |  |
| contact_title | text | null |  |
| contact_email | text | NOT NULL |  |
| contact_phone | text | null |  |
| estimated_students | integer | null |  |
| preferred_dates | text | null |  |
| notes | text | null |  |
| status | text | NOT NULL | 'new'::text |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| preferred_program | text | null |  |
| utm_source | text | null | 'unknown'::text |
| utm_medium | text | null | 'unknown'::text |
| utm_campaign | text | null | 'unknown'::text |
| utm_content | text | null | 'unknown'::text |
| utm_term | text | null | 'unknown'::text |

Constraints:
- `school_inquiries_pkey` PRIMARY KEY (id)

Indexes:
- INDEX school_inquiries_created_at_idx USING btree (created_at DESC)
- UNIQUE INDEX school_inquiries_pkey USING btree (id)

### testprep.sent_emails

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| template_name | text | NOT NULL |  |
| recipient_email | text | NOT NULL |  |
| cc_emails | text[] | null |  |
| subject | text | NOT NULL |  |
| reference_id | uuid | null |  |
| reference_table | text | null |  |
| status | text | NOT NULL | 'sent'::text |
| error_message | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `sent_emails_pkey` PRIMARY KEY (id)

Indexes:
- INDEX sent_emails_created_at_idx USING btree (created_at DESC)
- UNIQUE INDEX sent_emails_pkey USING btree (id)
- INDEX sent_emails_reference_idx USING btree (reference_id, template_name, status)

### testprep.session_analytics

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | text | NOT NULL |  |
| first_page | text | null |  |
| last_page | text | null |  |
| page_count | integer | null | 1 |
| started_at | timestamp with time zone | null | now() |
| ended_at | timestamp with time zone | null |  |
| duration_seconds | integer | null |  |
| device_type | text | null |  |
| browser | text | null |  |
| country | text | null |  |
| applied | boolean | null | false |
| converted | boolean | null | false |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| created_at | timestamp with time zone | null | now() |

Constraints:
- `session_analytics_pkey` PRIMARY KEY (id)
- `session_analytics_session_id_key` UNIQUE (session_id)

Indexes:
- INDEX idx_session_analytics_started_at USING btree (started_at)
- UNIQUE INDEX session_analytics_pkey USING btree (id)
- UNIQUE INDEX session_analytics_session_id_key USING btree (session_id)

### testprep.student_potential_snapshots

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| potential_score | integer | NOT NULL | 0 |
| accuracy_score | integer | NOT NULL | 0 |
| speed_score | integer | NOT NULL | 0 |
| consistency_score | integer | NOT NULL | 0 |
| mastery_score | integer | NOT NULL | 0 |
| questions_answered_total | integer | NOT NULL | 0 |
| traps_mastered_count | integer | NOT NULL | 0 |
| growth_velocity | numeric | NOT NULL | 0 |
| snapshot_date | timestamp with time zone | NOT NULL | now() |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `student_potential_snapshots_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_potential_snapshots_user_date USING btree (user_id, snapshot_date DESC)
- UNIQUE INDEX student_potential_snapshots_pkey USING btree (id)

### testprep.user_roles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| role | testprep.app_role | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| display_name | text | null |  |
| enrollment_status | text | null | 'pre'::text |
| program_week | integer | null | 1 |
| enrolled_at | timestamp with time zone | null |  |
| completed_at | timestamp with time zone | null |  |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `user_roles_pkey` PRIMARY KEY (id)
- `user_roles_user_id_key` UNIQUE (user_id)

Indexes:
- UNIQUE INDEX user_roles_pkey USING btree (id)
- UNIQUE INDEX user_roles_user_id_key USING btree (user_id)

### testprep.user_trap_stats

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| trap_type | testprep.trap_type | NOT NULL |  |
| total_encounters | integer | NOT NULL | 0 |
| times_fallen | integer | NOT NULL | 0 |
| current_streak | integer | NOT NULL | 0 |
| best_streak | integer | NOT NULL | 0 |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `user_trap_stats_pkey` PRIMARY KEY (id)
- `user_trap_stats_user_id_trap_type_key` UNIQUE (user_id, trap_type)

Indexes:
- UNIQUE INDEX user_trap_stats_pkey USING btree (id)
- UNIQUE INDEX user_trap_stats_user_id_trap_type_key USING btree (user_id, trap_type)

### testprep.whatsapp_custom_variables

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| value | text | NOT NULL | ''::text |
| description | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `whatsapp_custom_variables_pkey` PRIMARY KEY (id)
- `whatsapp_custom_variables_name_key` UNIQUE (name)

Indexes:
- UNIQUE INDEX whatsapp_custom_variables_name_key USING btree (name)
- UNIQUE INDEX whatsapp_custom_variables_pkey USING btree (id)

### testprep.whatsapp_sends

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| template_key | text | NOT NULL |  |
| wati_template_name | text | null |  |
| recipient_phone | text | NOT NULL |  |
| recipient_type | text | NOT NULL |  |
| recipient_name | text | null |  |
| application_id | uuid | null |  |
| status | text | NOT NULL | 'pending'::text |
| error | text | null |  |
| wati_response | jsonb | null |  |
| parameters | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `whatsapp_sends_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_wa_sends_application USING btree (application_id)
- INDEX idx_wa_sends_created_at USING btree (created_at DESC)
- UNIQUE INDEX whatsapp_sends_pkey USING btree (id)

### testprep.whatsapp_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| key | text | NOT NULL |  |
| label | text | NOT NULL |  |
| description | text | null |  |
| wati_template_name | text | null |  |
| language | text | null | 'en'::text |
| body_preview | text | null |  |
| parameters | jsonb | NOT NULL | '[]'::jsonb |
| enabled | boolean | NOT NULL | true |
| auto_send | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `whatsapp_templates_pkey` PRIMARY KEY (id)
- `whatsapp_templates_key_key` UNIQUE (key)

Indexes:
- UNIQUE INDEX whatsapp_templates_key_key USING btree (key)
- UNIQUE INDEX whatsapp_templates_pkey USING btree (id)
