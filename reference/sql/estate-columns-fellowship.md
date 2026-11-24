# Shared estate columns: fellowship (17 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### fellowship.cohort_settings

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_id | uuid | NOT NULL |  |
| program_key | text | NOT NULL |  |
| cohort_label | text | NOT NULL | 'Default'::text |
| is_active | boolean | NOT NULL | false |
| payment_link_url | text | null |  |
| price_usd | numeric | null |  |
| sort_order | integer | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_cohorts_cohort_id_fkey` FOREIGN KEY (cohort_id) REFERENCES all_cohorts(id) ON DELETE RESTRICT
- `program_cohorts_program_key_fkey` FOREIGN KEY (program_key) REFERENCES fellowship.program_map(program_key) ON UPDATE CASCADE
- `program_cohorts_pkey` PRIMARY KEY (id)
- `program_cohorts_cohort_id_key` UNIQUE (cohort_id)

Indexes:
- INDEX idx_onward_program_cohorts_key USING btree (program_key)
- UNIQUE INDEX program_cohorts_cohort_id_key USING btree (cohort_id)
- UNIQUE INDEX program_cohorts_pkey USING btree (id)

### fellowship.edge_rate_limits

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| bucket | text | NOT NULL |  |
| key | text | NOT NULL |  |
| window_start | timestamp with time zone | NOT NULL |  |
| count | integer | NOT NULL | 0 |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `edge_rate_limits_pkey` PRIMARY KEY (id)
- `edge_rate_limits_bucket_key_window_start_key` UNIQUE (bucket, key, window_start)

Indexes:
- UNIQUE INDEX edge_rate_limits_bucket_key_window_start_key USING btree (bucket, key, window_start)
- UNIQUE INDEX edge_rate_limits_pkey USING btree (id)

### fellowship.email_templates

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
| sender_name | text | NOT NULL | 'Fellowship Programme'::text |
| is_active | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `email_templates_pkey` PRIMARY KEY (id)
- `email_templates_name_key` UNIQUE (name)

Indexes:
- UNIQUE INDEX email_templates_name_key USING btree (name)
- UNIQUE INDEX email_templates_pkey USING btree (id)

### fellowship.interest_signups

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| email | text | NOT NULL |  |
| phone | text | null |  |
| user_type | text | NOT NULL |  |
| created_at | timestamp with time zone | null | now() |
| school_type | text | null |  |
| referral_source | text | null |  |
| referral_code | text | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| utm_content | text | null |  |
| utm_term | text | null |  |

Constraints:
- `interest_signups_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX interest_signups_pkey USING btree (id)

### fellowship.profiles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL |  |
| full_name | text | null |  |
| avatar_url | text | null |  |
| grade_level | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `profiles_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX profiles_pkey USING btree (id)

### fellowship.program_attachments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_key | text | NOT NULL |  |
| file_path | text | NOT NULL |  |
| file_name | text | NOT NULL |  |
| output_filename_template | text | NOT NULL | '{{student_name}}.pdf'::text |
| name_page | integer | NOT NULL | 1 |
| name_x | numeric | NOT NULL | 79 |
| name_y | numeric | NOT NULL | 622 |
| font_family | text | NOT NULL | 'Montserrat-Bold'::text |
| font_storage_path | text | null |  |
| font_size | numeric | NOT NULL | 12 |
| font_color | text | NOT NULL | '#1a1a1a'::text |
| name_prefix | text | NOT NULL | ''::text |
| name_suffix | text | NOT NULL | ','::text |
| is_active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| x_offset | numeric | NOT NULL | 0 |
| y_offset | numeric | NOT NULL | 0 |

Constraints:
- `program_attachments_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX program_attachments_pkey USING btree (id)

### fellowship.program_email_settings

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| category | text | NOT NULL |  |
| application_emails_enabled | boolean | NOT NULL | true |
| payment_emails_enabled | boolean | NOT NULL | true |
| reminder_emails_enabled | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| letter_of_acceptance_enabled | boolean | NOT NULL | false |

Constraints:
- `program_email_settings_pkey` PRIMARY KEY (id)
- `program_email_settings_category_key` UNIQUE (category)

Indexes:
- UNIQUE INDEX program_email_settings_category_key USING btree (category)
- UNIQUE INDEX program_email_settings_pkey USING btree (id)

### fellowship.program_entry_tiers

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_key | text | NOT NULL |  |
| entry_key | text | NOT NULL |  |
| label | text | NOT NULL |  |
| price_usd | numeric | null |  |
| stripe_price_id | text | null |  |
| payment_link_url | text | null |  |
| sort_order | integer | NOT NULL | 0 |
| is_active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_entry_tiers_pkey` PRIMARY KEY (id)
- `program_entry_tiers_program_key_entry_key_key` UNIQUE (program_key, entry_key)

Indexes:
- UNIQUE INDEX program_entry_tiers_pkey USING btree (id)
- UNIQUE INDEX program_entry_tiers_program_key_entry_key_key USING btree (program_key, entry_key)

### fellowship.program_feedback

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| created_at | timestamp with time zone | NOT NULL | now() |
| program_slug | text | null |  |
| cohort | text | null |  |
| feedback_date | date | NOT NULL | ((now() AT TIME ZONE 'utc'::text))::date |
| day_number | integer | NOT NULL |  |
| student_name | text | null |  |
| mood | integer | NOT NULL |  |
| likert_enjoyed | integer | NOT NULL |  |
| likert_learned | integer | NOT NULL |  |
| likert_engaged | integer | NOT NULL |  |
| likert_pace | integer | NOT NULL |  |
| likert_comfortable | integer | NOT NULL |  |
| likert_supported | integer | NOT NULL |  |
| likert_clarity | integer | NOT NULL |  |
| likert_tomorrow | integer | NOT NULL |  |
| nps | integer | NOT NULL |  |
| nps_category | text | null |  |
| best_part | text | null |  |
| learned_text | text | null |  |
| improve | text | null |  |
| more_of | text | null |  |
| other | text | null |  |
| user_agent | text | null |  |
| ip_hash | text | null |  |

Constraints:
- `program_feedback_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_onward_feedback_cohort USING btree (cohort)
- UNIQUE INDEX program_feedback_pkey USING btree (id)

### fellowship.program_feedback_mentors

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| feedback_id | uuid | NOT NULL |  |
| mentor_id | uuid | null |  |
| mentor_name | text | NOT NULL |  |
| rating_overall | integer | null |  |
| likert_helpful | integer | null |  |
| likert_clarity | integer | null |  |
| likert_curriculum | integer | null |  |
| comment | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_feedback_mentors_feedback_id_fkey` FOREIGN KEY (feedback_id) REFERENCES fellowship.program_feedback(id) ON DELETE CASCADE
- `program_feedback_mentors_mentor_id_fkey` FOREIGN KEY (mentor_id) REFERENCES all_mentors(id) ON DELETE SET NULL
- `program_feedback_mentors_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX program_feedback_mentors_pkey USING btree (id)

### fellowship.program_map

| column | type | nullable | default |
|---|---|---|---|
| program_key | text | NOT NULL |  |
| program_id | uuid | NOT NULL |  |
| program_label | text | NOT NULL |  |
| code_prefix | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_map_program_id_fkey` FOREIGN KEY (program_id) REFERENCES all_programs(id) ON DELETE RESTRICT
- `program_map_pkey` PRIMARY KEY (program_key)

Indexes:
- UNIQUE INDEX program_map_pkey USING btree (program_key)

### fellowship.promo_slot_claims

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| code | text | NOT NULL |  |
| holder | text | NOT NULL |  |
| succeeded | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `promo_slot_claims_pkey` PRIMARY KEY (id)
- `promo_slot_claims_code_holder_key` UNIQUE (code, holder)

Indexes:
- UNIQUE INDEX promo_slot_claims_code_holder_key USING btree (code, holder)
- UNIQUE INDEX promo_slot_claims_pkey USING btree (id)

### fellowship.sent_emails

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| template_name | text | NOT NULL |  |
| recipient_email | text | NOT NULL |  |
| cc_emails | text[] | null |  |
| subject | text | NOT NULL |  |
| application_id | uuid | null |  |
| status | text | NOT NULL | 'pending'::text |
| error_message | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `sent_emails_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_onward_sent_emails_app USING btree (application_id)
- UNIQUE INDEX sent_emails_pkey USING btree (id)

### fellowship.unmatched_payments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| payment_intent_id | text | null |  |
| email | text | null |  |
| customer_name | text | null |  |
| amount | numeric | null |  |
| currency | text | null |  |
| cohort | text | null |  |
| program_code | text | null |  |
| paid_at | timestamp with time zone | null |  |
| status | text | null | 'unmatched'::text |
| raw | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `unmatched_payments_pkey` PRIMARY KEY (id)
- `unmatched_payments_payment_intent_id_key` UNIQUE (payment_intent_id)

Indexes:
- UNIQUE INDEX unmatched_payments_payment_intent_id_key USING btree (payment_intent_id)
- UNIQUE INDEX unmatched_payments_pkey USING btree (id)

### fellowship.user_roles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| role | fellowship.app_role | NOT NULL |  |
| school_id | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `user_roles_pkey` PRIMARY KEY (id)
- `user_roles_user_id_role_key` UNIQUE (user_id, role)

Indexes:
- INDEX idx_onward_user_roles_user_id USING btree (user_id)
- UNIQUE INDEX user_roles_pkey USING btree (id)
- UNIQUE INDEX user_roles_user_id_role_key USING btree (user_id, role)

### fellowship.whatsapp_send_log

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| application_id | uuid | null |  |
| event_key | text | NOT NULL |  |
| template_name | text | null |  |
| recipient_role | text | NOT NULL |  |
| phone | text | null |  |
| status | text | NOT NULL |  |
| wati_response | jsonb | null |  |
| error | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `whatsapp_send_log_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_onward_wa_log_app USING btree (application_id)
- UNIQUE INDEX whatsapp_send_log_pkey USING btree (id)

### fellowship.whatsapp_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| event_key | text | NOT NULL |  |
| template_name | text | NOT NULL | ''::text |
| broadcast_name | text | NOT NULL | ''::text |
| variables | jsonb | NOT NULL | '[]'::jsonb |
| requires_payment_link | boolean | NOT NULL | false |
| is_active | boolean | NOT NULL | false |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| recipient_role | text | NOT NULL |  |
| contact_attributes | jsonb | NOT NULL | '[]'::jsonb |

Constraints:
- `whatsapp_templates_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX whatsapp_templates_pkey USING btree (id)
