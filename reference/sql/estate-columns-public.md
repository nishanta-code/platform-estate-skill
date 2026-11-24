# Shared estate columns: public (25 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### public.all_cohorts

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_id | uuid | NOT NULL |  |
| cohort_key | text | null |  |
| program_code | text | null |  |
| start_date | date | null |  |
| end_date | date | null |  |
| application_deadline | date | null |  |
| seats_total | integer | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| cohort_label | text | null |  |

Constraints:
- `all_cohorts_dates_sane` CHECK (((end_date IS NULL) OR (start_date IS NULL) OR (end_date >= start_date)))
- `all_cohorts_program_id_fkey` FOREIGN KEY (program_id) REFERENCES all_programs(id) ON DELETE CASCADE
- `all_cohorts_pkey` PRIMARY KEY (id)
- `all_cohorts_cohort_key_key` UNIQUE (cohort_key)
- `all_cohorts_program_code_key` UNIQUE (program_code)
- `all_cohorts_program_id_uk` UNIQUE (program_id, id)

Indexes:
- UNIQUE INDEX all_cohorts_cohort_key_key USING btree (cohort_key)
- UNIQUE INDEX all_cohorts_pkey USING btree (id)
- UNIQUE INDEX all_cohorts_program_code_key USING btree (program_code)
- UNIQUE INDEX all_cohorts_program_id_uk USING btree (program_id, id)
- INDEX all_cohorts_program_idx USING btree (program_id)
- INDEX all_cohorts_start_idx USING btree (start_date)

### public.all_masterclasses

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| partner_id | uuid | NOT NULL |  |
| program_id | uuid | null |  |
| cohort_id | uuid | null |  |
| slug | text | NOT NULL |  |
| title | text | NOT NULL |  |
| description | text | null |  |
| series | text | null |  |
| session_number | integer | null |  |
| audience | text | NOT NULL | 'general'::text |
| scheduled_at | timestamp with time zone | null |  |
| duration_minutes | integer | NOT NULL | 60 |
| display_timezone | text | NOT NULL | 'UTC'::text |
| host_mentor_id | uuid | null |  |
| host_name | text | null |  |
| learning_outcomes | jsonb | NOT NULL | '[]'::jsonb |
| join_url | text | null |  |
| passcode | text | null |  |
| apply_url | text | null |  |
| capacity | integer | null |  |
| is_published | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| page_copy | jsonb | null |  |

Constraints:
- `all_masterclasses_audience_check` CHECK ((audience = ANY (ARRAY['general'::text, 'senior'::text, 'junior'::text])))
- `all_masterclasses_host_present` CHECK (((host_mentor_id IS NOT NULL) OR (host_name IS NOT NULL)))
- `all_masterclasses_host_mentor_id_fkey` FOREIGN KEY (host_mentor_id) REFERENCES all_mentors(id) ON DELETE SET NULL
- `all_masterclasses_partner_id_fkey` FOREIGN KEY (partner_id) REFERENCES all_partners(id) ON DELETE RESTRICT
- `all_masterclasses_partner_program_fk` FOREIGN KEY (partner_id, program_id) REFERENCES all_programs(partner_id, id) ON DELETE SET NULL
- `all_masterclasses_program_cohort_fk` FOREIGN KEY (program_id, cohort_id) REFERENCES all_cohorts(program_id, id) ON DELETE SET NULL
- `all_masterclasses_pkey` PRIMARY KEY (id)
- `all_masterclasses_slug_key` UNIQUE (slug)

Indexes:
- INDEX all_masterclasses_cohort_idx USING btree (cohort_id)
- INDEX all_masterclasses_partner_idx USING btree (partner_id)
- UNIQUE INDEX all_masterclasses_pkey USING btree (id)
- INDEX all_masterclasses_program_idx USING btree (program_id)
- INDEX all_masterclasses_scheduled_idx USING btree (scheduled_at)
- UNIQUE INDEX all_masterclasses_slug_key USING btree (slug)

### public.all_mentors

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| full_name | text | NOT NULL |  |
| bio | text | null |  |
| university | text | null |  |
| image_url | text | null |  |
| is_active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| email | text | null |  |
| country | text | null |  |
| updated_by_platform | text | null |  |

Constraints:
- `all_mentors_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX all_mentors_email_unique USING btree (lower(email)) WHERE (email IS NOT NULL)
- UNIQUE INDEX all_mentors_pkey USING btree (id)

### public.all_partners

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| partner_slug | text | NOT NULL |  |
| org_name | text | NOT NULL |  |
| description | text | null |  |
| website | text | null |  |
| logo_url | text | null |  |
| country | text | null |  |
| contact_name | text | null |  |
| contact_phone | text | null |  |
| offerings | text[] | NOT NULL | '{}'::text[] |
| approval_status | text | NOT NULL | 'pending'::text |
| approved_at | timestamp with time zone | null |  |
| is_active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `all_partners_approval_status_check` CHECK ((approval_status = ANY (ARRAY['pending'::text, 'approved'::text, 'rejected'::text])))
- `all_partners_pkey` PRIMARY KEY (id)
- `all_partners_partner_slug_key` UNIQUE (partner_slug)

Indexes:
- INDEX all_partners_offerings_idx USING gin (offerings)
- UNIQUE INDEX all_partners_partner_slug_key USING btree (partner_slug)
- UNIQUE INDEX all_partners_pkey USING btree (id)

### public.all_programs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| partner_id | uuid | NOT NULL |  |
| program_name | text | NOT NULL |  |
| short_description | text | null |  |
| description | text | null |  |
| categories | text[] | NOT NULL | '{}'::text[] |
| format | text | null |  |
| location | text | null |  |
| duration | text | null |  |
| eligibility_grades | text | null |  |
| sticker_price | integer | NOT NULL |  |
| early_bird_price | integer | null |  |
| price_currency | character(3) | NOT NULL | 'USD'::bpchar |
| cover_image_url | text | null |  |
| program_url | text | null |  |
| apply_url | text | null |  |
| is_active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| pay_slug | text | null |  |
| alt_currency | jsonb | null |  |

Constraints:
- `all_programs_partner_id_fkey` FOREIGN KEY (partner_id) REFERENCES all_partners(id) ON DELETE RESTRICT
- `all_programs_pkey` PRIMARY KEY (id)
- `all_programs_partner_id_uk` UNIQUE (partner_id, id)
- `all_programs_pay_slug_key` UNIQUE (pay_slug)

Indexes:
- INDEX all_programs_active_idx USING btree (is_active)
- INDEX all_programs_cats_idx USING gin (categories)
- UNIQUE INDEX all_programs_partner_id_uk USING btree (partner_id, id)
- INDEX all_programs_partner_idx USING btree (partner_id)
- UNIQUE INDEX all_programs_pay_slug_key USING btree (pay_slug)
- UNIQUE INDEX all_programs_pkey USING btree (id)

### public.automation_app_config

| column | type | nullable | default |
|---|---|---|---|
| key | text | NOT NULL |  |
| value | text | NOT NULL |  |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `automation_app_config_pkey` PRIMARY KEY (key)

Indexes:
- UNIQUE INDEX automation_app_config_pkey USING btree (key)

### public.automation_video_posts

| column | type | nullable | default |
|---|---|---|---|
| drive_file_id | text | NOT NULL |  |
| drive_name | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| posted_date | date | null |  |
| yt_status | text | NOT NULL | 'pending'::text |
| yt_video_id | text | null |  |
| yt_error | text | null |  |
| ig_status | text | NOT NULL | 'pending'::text |
| ig_media_id | text | null |  |
| ig_error | text | null |  |
| status | text | NOT NULL | 'processing'::text |
| yt_title | text | null |  |
| ig_caption | text | null |  |
| storage_url | text | null |  |
| md5_checksum | text | null |  |

Constraints:
- `automation_video_posts_pkey` PRIMARY KEY (drive_file_id)

Indexes:
- INDEX automation_video_posts_md5_idx USING btree (md5_checksum) WHERE (md5_checksum IS NOT NULL)
- UNIQUE INDEX automation_video_posts_pkey USING btree (drive_file_id)
- INDEX automation_video_posts_posted_date_idx USING btree (posted_date DESC)
- INDEX automation_video_posts_status_idx USING btree (status)

### public.data_fix_log

| column | type | nullable | default |
|---|---|---|---|
| id | bigint | NOT NULL | nextval('data_fix_log_id_seq'::regclass) |
| fixed_at | timestamp with time zone | NOT NULL | now() |
| batch | text | NOT NULL |  |
| table_name | text | NOT NULL |  |
| row_id | text | NOT NULL |  |
| column_name | text | NOT NULL |  |
| old_value | text | null |  |
| new_value | text | null |  |
| old_row | jsonb | NOT NULL |  |
| reason | text | NOT NULL |  |

Constraints:
- `data_fix_log_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX data_fix_log_pkey USING btree (id)

### public.premed_masterclass_regs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| sessions | text[] | NOT NULL | '{}'::text[] |
| masterclass_ids | uuid[] | NOT NULL | '{}'::uuid[] |
| category | text | NOT NULL | 'future_doctors'::text |
| attendee_type | text | null |  |
| full_name | text | NOT NULL |  |
| email | text | NOT NULL |  |
| phone | text | null |  |
| designation | text | null |  |
| student_name | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| school | text | null |  |
| grade | text | null |  |
| country | text | null |  |
| consent | boolean | NOT NULL | false |
| status | text | NOT NULL | 'registered'::text |
| utm_source | text | NOT NULL | 'direct'::text |
| utm_medium | text | NOT NULL | 'website'::text |
| utm_campaign | text | NOT NULL | 'direct'::text |
| meta | jsonb | NOT NULL | '{}'::jsonb |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `premed_masterclass_regs_pkey` PRIMARY KEY (id)

Indexes:
- INDEX premed_masterclass_regs_category_idx USING btree (category)
- INDEX premed_masterclass_regs_created_at_idx USING btree (created_at DESC)
- INDEX premed_masterclass_regs_email_idx USING btree (lower(email))
- INDEX premed_masterclass_regs_mc_ids_idx USING gin (masterclass_ids)
- UNIQUE INDEX premed_masterclass_regs_pkey USING btree (id)

### public.finedu_masterclass_regs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| selected_sessions | text | NOT NULL |  |
| session_key | text | null |  |
| session_keys | text[] | NOT NULL | '{}'::text[] |
| full_name | text | NOT NULL |  |
| email | text | NOT NULL |  |
| phone | text | null |  |
| country | text | null |  |
| role | text | NOT NULL |  |
| grade | text | null |  |
| school_name | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| student_name | text | null |  |
| student_grade | text | null |  |
| designation | text | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `finedu_masterclass_regs_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX finedu_masterclass_regs_pkey USING btree (id)
- INDEX idx_finedu_masterclass_regs_created USING btree (created_at DESC)
- INDEX idx_finedu_masterclass_regs_email USING btree (email)

### public.prelaw_masterclass_regs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| created_at | timestamp with time zone | NOT NULL | now() |
| sessions | text[] | NOT NULL |  |
| attendee_type | text | NOT NULL |  |
| full_name | text | NOT NULL |  |
| email | text | NOT NULL |  |
| phone | text | null |  |
| country | text | null |  |
| state | text | null |  |
| city | text | null |  |
| grade | text | null |  |
| school | text | null |  |
| hear_about | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| student_name | text | null |  |
| student_grade | text | null |  |
| designation | text | null |  |
| consent | boolean | NOT NULL | false |
| source_url | text | null |  |
| referrer | text | null |  |
| user_agent | text | null |  |
| email_sent_at | timestamp with time zone | null |  |
| utm_source | text | NOT NULL | 'unknown'::text |
| utm_medium | text | NOT NULL | 'unknown'::text |
| utm_campaign | text | NOT NULL | 'unknown'::text |
| promotes_program_code | text | null |  |
| masterclass_ids | uuid[] | null | '{}'::uuid[] |

Constraints:
- `prelaw_masterclass_regs_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_lmr_created_at USING btree (created_at DESC)
- INDEX idx_lmr_email USING btree (email)
- INDEX idx_lmr_email_lower USING btree (lower(email))
- INDEX idx_lmr_ppc USING btree (promotes_program_code)
- UNIQUE INDEX prelaw_masterclass_regs_pkey USING btree (id)

### public.applications

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_id | text | null | portal.generate_student_id() |
| created_at | timestamp with time zone | NOT NULL | now() |
| program_code | text | null |  |
| program_name | text | null |  |
| program_category | text | null |  |
| student_name | text | NOT NULL |  |
| student_email | text | NOT NULL |  |
| student_mobile | text | null |  |
| student_dob | date | null |  |
| grade | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| school_name | text | null |  |
| country | text | null |  |
| state | text | null |  |
| city | text | null |  |
| counsellor_name | text | null |  |
| counsellor_email | text | null |  |
| visa_status | text | null |  |
| admin_notes | text | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| payment_status | text | NOT NULL | 'pending'::text |
| amount | numeric | null |  |
| currency | text | null | 'USD'::text |
| payment_intent_id | text | null |  |
| payment_date | timestamp with time zone | null |  |
| stripe_customer_id | text | null |  |
| submission_token | uuid | NOT NULL | gen_random_uuid() |
| loa_sent_at | timestamp with time zone | null |  |
| last_reminder_stage | text | null |  |
| last_reminder_sent_at | timestamp with time zone | null |  |
| payment_mode | text | null |  |
| form_template_id | uuid | null |  |
| source_flow | text | NOT NULL | 'application_first'::text |
| registration_status | text | NOT NULL | 'complete'::text |
| registration_email_sent_at | timestamp with time zone | null |  |
| registration_reminder_count | integer | NOT NULL | 0 |
| registration_last_reminder_at | timestamp with time zone | null |  |
| payment_link_id | uuid | null |  |
| student_gender | text | null |  |
| scholarship_code | text | null |  |
| form_data | jsonb | NOT NULL | '{}'::jsonb |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `applications_form_data_object` CHECK ((jsonb_typeof(form_data) = 'object'::text))
- `applications_payment_mode_check` CHECK ((payment_mode = ANY (ARRAY['full'::text, 'installments'::text])))
- `applications_registration_status_check` CHECK ((registration_status = ANY (ARRAY['complete'::text, 'pending_application'::text, 'withdrawn'::text])))
- `applications_source_flow_check` CHECK ((source_flow = ANY (ARRAY['application_first'::text, 'payment_first'::text, 'b2b_link'::text])))
- `applications_form_template_id_fkey` FOREIGN KEY (form_template_id) REFERENCES portal.form_templates(id) ON DELETE SET NULL
- `all_applications_new_pkey` PRIMARY KEY (id)

Indexes:
- INDEX all_applications_application_id_idx USING btree (student_id)
- UNIQUE INDEX all_applications_new_pkey USING btree (id)
- INDEX all_applications_payment_intent_id_idx USING btree (payment_intent_id) WHERE (payment_intent_id IS NOT NULL)
- INDEX all_applications_payment_status_created_idx USING btree (payment_status, created_at DESC)
- UNIQUE INDEX all_applications_pending_email_program_uidx USING btree (lower(student_email), COALESCE(program_code, ''::text)) WHERE (payment_status = 'pending'::text)
- UNIQUE INDEX all_applications_submission_token_uidx USING btree (submission_token)
- INDEX idx_all_applications_created_at USING btree (created_at DESC)
- INDEX idx_all_applications_program_code USING btree (program_code)
- INDEX idx_all_applications_student_email USING btree (lower(student_email))
- INDEX idx_all_apps_loa_pending USING btree (created_at) WHERE ((loa_sent_at IS NULL) AND (payment_status = 'pending'::text))
- INDEX idx_all_apps_reminder_pending USING btree (program_code, last_reminder_stage) WHERE (payment_status = 'pending'::text)
- INDEX idx_applications_registration_status USING btree (registration_status) WHERE (registration_status = 'pending_application'::text)
- INDEX applications_form_template_id_idx USING btree (form_template_id)

### public.form_responses

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| form_name | text | NOT NULL |  |
| data | jsonb | NOT NULL | '{}'::jsonb |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `form_responses_pkey` PRIMARY KEY (id)

Indexes:
- INDEX form_responses_email_idx USING btree (((data ->> 'form_slug'::text)), lower((data ->> 'respondent_email'::text)))
- INDEX form_responses_form_name_idx USING btree (form_name)
- UNIQUE INDEX form_responses_pkey USING btree (id)
- INDEX form_responses_slug_idx USING btree (((data ->> 'form_slug'::text)))

### public.masterclass_regs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| masterclass_id | uuid | null |  |
| partner_id | uuid | null |  |
| full_name | text | NOT NULL |  |
| email | text | NOT NULL |  |
| phone | text | null |  |
| country | text | null |  |
| attendee_type | text | NOT NULL | 'student'::text |
| grade | text | null |  |
| school_name | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| student_name | text | null |  |
| student_grade | text | null |  |
| consent | boolean | NOT NULL | false |
| status | text | NOT NULL | 'registered'::text |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| form_data | jsonb | NOT NULL | '{}'::jsonb |
| email_sent_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| migrated_from_table | text | null |  |
| migrated_from_id | uuid | null |  |
| migration_batch | text | null |  |
| platform_name | text | null |  |
| session_name | text | null |  |
| designation | text | null |  |

Constraints:
- `masterclass_regs_attendee_type_check` CHECK ((attendee_type = ANY (ARRAY['student'::text, 'parent'::text, 'educator'::text, 'other'::text])))
- `masterclass_regs_form_data_check` CHECK ((jsonb_typeof(form_data) = 'object'::text))
- `masterclass_regs_status_check` CHECK ((status = ANY (ARRAY['registered'::text, 'attended'::text, 'cancelled'::text, 'no_show'::text])))
- `masterclass_regs_masterclass_id_fkey` FOREIGN KEY (masterclass_id) REFERENCES all_masterclasses(id)
- `masterclass_regs_partner_id_fkey` FOREIGN KEY (partner_id) REFERENCES all_partners(id)
- `masterclass_regs_pkey` PRIMARY KEY (id)

Indexes:
- INDEX masterclass_regs_lower_idx USING btree (lower(email))
- UNIQUE INDEX masterclass_regs_masterclass_email_key USING btree (masterclass_id, lower(email))
- INDEX masterclass_regs_masterclass_id_idx USING btree (masterclass_id)
- INDEX masterclass_regs_partner_id_idx USING btree (partner_id)
- UNIQUE INDEX masterclass_regs_pkey USING btree (id)

### public.master_data

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| first_name | text | NOT NULL | ''::text |
| last_name | text | NOT NULL | ''::text |
| title | text | null | ''::text |
| company_name | text | NOT NULL | ''::text |
| email | text | null | ''::text |
| email_status | text | null | ''::text |
| primary_email_source | text | null | ''::text |
| corporate_phone | text | null | ''::text |
| industry | text | null | ''::text |
| keywords | text | null | ''::text |
| person_linkedin_url | text | null | ''::text |
| website | text | null | ''::text |
| company_linkedin_url | text | null | ''::text |
| city | text | null | ''::text |
| state | text | null | ''::text |
| country | text | null | ''::text |
| data_user | uuid | null |  |
| last_email_sent_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `master_data_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX master_data_email_unique USING btree (email) WHERE ((email IS NOT NULL) AND (email <> ''::text))
- UNIQUE INDEX master_data_pkey USING btree (id)

### public.fellowship_masterclass_regs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_key | text | NOT NULL | 'aischool_masterclass'::text |
| full_name | text | NOT NULL |  |
| email | text | NOT NULL |  |
| phone | text | null |  |
| country | text | null |  |
| role | text | NOT NULL |  |
| grade | text | null |  |
| school_name | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| child_name | text | null |  |
| child_grade | text | null |  |
| designation | text | null |  |
| sessions | text[] | NOT NULL | '{}'::text[] |
| email_sent | boolean | NOT NULL | false |
| email_sent_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| synced_at | timestamp with time zone | NOT NULL | now() |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| questions | text | null |  |
| utm_content | text | null |  |
| utm_term | text | null |  |

Constraints:
- `fellowship_masterclass_regs_pkey` PRIMARY KEY (id)

Indexes:
- INDEX fellowship_masterclass_regs_created_at_idx USING btree (created_at DESC)
- INDEX fellowship_masterclass_regs_email_idx USING btree (email)
- UNIQUE INDEX fellowship_masterclass_regs_pkey USING btree (id)
- INDEX fellowship_masterclass_regs_program_key_idx USING btree (program_key)

### public.platform_api_clients

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| platform_slug | text | NOT NULL |  |
| name | text | NOT NULL |  |
| key_hash | text | NOT NULL |  |
| scopes | text[] | NOT NULL | '{}'::text[] |
| is_active | boolean | NOT NULL | true |
| last_used_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `platform_api_clients_pkey` PRIMARY KEY (id)
- `platform_api_clients_key_hash_key` UNIQUE (key_hash)
- `platform_api_clients_platform_slug_key` UNIQUE (platform_slug)

Indexes:
- UNIQUE INDEX platform_api_clients_key_hash_key USING btree (key_hash)
- UNIQUE INDEX platform_api_clients_pkey USING btree (id)
- UNIQUE INDEX platform_api_clients_platform_slug_key USING btree (platform_slug)

### public.promo_codes

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| code | text | NOT NULL |  |
| program_id | uuid | NOT NULL |  |
| program_name | text | null |  |
| discount_type | text | NOT NULL | 'fixed'::text |
| discount_value | numeric | NOT NULL |  |
| currency | text | null |  |
| max_redemptions | integer | null |  |
| redemption_count | integer | NOT NULL | 0 |
| starts_at | timestamp with time zone | null |  |
| expires_at | timestamp with time zone | null |  |
| is_active | boolean | NOT NULL | true |
| note | text | null |  |
| created_by_platform | text | null |  |
| created_by_email | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `promo_codes_code_format` CHECK (((code = upper(code)) AND (code ~ '^[A-Z0-9_-]{3,32}$'::text)))
- `promo_codes_discount_type_check` CHECK ((discount_type = ANY (ARRAY['fixed'::text, 'percent'::text])))
- `promo_codes_discount_value_check` CHECK ((discount_value > (0)::numeric))
- `promo_codes_fixed_currency` CHECK (((discount_type <> 'fixed'::text) OR (currency IS NOT NULL)))
- `promo_codes_max_redemptions_check` CHECK (((max_redemptions IS NULL) OR (max_redemptions > 0)))
- `promo_codes_percent_max` CHECK (((discount_type <> 'percent'::text) OR (discount_value <= (100)::numeric)))
- `promo_codes_program_id_fkey` FOREIGN KEY (program_id) REFERENCES all_programs(id) ON DELETE CASCADE
- `promo_codes_pkey` PRIMARY KEY (id)
- `promo_codes_code_key` UNIQUE (code)

Indexes:
- INDEX promo_codes_active_idx USING btree (is_active)
- UNIQUE INDEX promo_codes_code_key USING btree (code)
- UNIQUE INDEX promo_codes_pkey USING btree (id)
- INDEX promo_codes_program_idx USING btree (program_id)

### public.promo_redemptions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| promo_code_id | uuid | null |  |
| promo_code | text | NOT NULL |  |
| platform_slug | text | null |  |
| program_code | text | null |  |
| cohort_code | text | null |  |
| customer_email | text | null |  |
| application_id | text | null |  |
| payment_intent_id | text | NOT NULL |  |
| amount_before | numeric | null |  |
| amount_after | numeric | null |  |
| discount_amount | numeric | null |  |
| currency | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `promo_redemptions_promo_code_id_fkey` FOREIGN KEY (promo_code_id) REFERENCES promo_codes(id) ON DELETE SET NULL
- `promo_redemptions_pkey` PRIMARY KEY (id)
- `promo_redemptions_payment_intent_id_key` UNIQUE (payment_intent_id)

Indexes:
- INDEX promo_redemptions_code_idx USING btree (promo_code)
- INDEX promo_redemptions_email_idx USING btree (lower(customer_email))
- UNIQUE INDEX promo_redemptions_payment_intent_id_key USING btree (payment_intent_id)
- UNIQUE INDEX promo_redemptions_pkey USING btree (id)

### public.razorpay_payments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| created_date | timestamp without time zone | null |  |
| payment_id | text | NOT NULL |  |
| amount | numeric(12,2) | null |  |
| currency | text | null |  |
| status | text | null |  |
| method | text | null |  |
| email | text | null |  |
| contact | text | null |  |
| description | text | null |  |
| order_id | text | null |  |
| fee | numeric(12,2) | null |  |
| tax | numeric(12,2) | null |  |
| international | text | null |  |
| refund_status | text | null |  |
| amount_refunded | numeric(12,2) | null |  |
| bank | text | null |  |
| wallet | text | null |  |
| vpa | text | null |  |
| notes | jsonb | null | '{}'::jsonb |
| synced_at | timestamp with time zone | null | now() |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |

Constraints:
- `razorpay_payments_pkey` PRIMARY KEY (id)
- `razorpay_payments_payment_id_key` UNIQUE (payment_id)

Indexes:
- INDEX idx_razorpay_payments_created USING btree (created_date)
- INDEX idx_razorpay_payments_email USING btree (email)
- INDEX idx_razorpay_payments_order_id USING btree (order_id)
- INDEX idx_razorpay_payments_status USING btree (status)
- UNIQUE INDEX razorpay_payments_payment_id_key USING btree (payment_id)
- UNIQUE INDEX razorpay_payments_pkey USING btree (id)

### public.testprep_masterclass_regs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| full_name | text | NOT NULL |  |
| email | text | NOT NULL |  |
| phone | text | NOT NULL |  |
| country | text | NOT NULL |  |
| role | text | NOT NULL |  |
| selected_sessions | text[] | NOT NULL | '{}'::text[] |
| grade | text | null |  |
| school_name | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| student_name | text | null |  |
| student_grade | text | null |  |
| designation | text | null |  |
| utm_source | text | null | 'direct'::text |
| utm_medium | text | null | 'website'::text |
| utm_campaign | text | null | 'direct'::text |
| email_sent | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| utm_content | text | null | 'unknown'::text |
| utm_term | text | null | 'unknown'::text |

Constraints:
- `testprep_masterclass_regs_role_check` CHECK ((role = ANY (ARRAY['Student'::text, 'Parent'::text, 'Educator'::text])))
- `testprep_masterclass_regs_pkey` PRIMARY KEY (id)
- `testprep_masterclass_regs_email_key` UNIQUE (email)

Indexes:
- INDEX idx_testprep_masterclass_regs_created_at USING btree (created_at DESC)
- UNIQUE INDEX testprep_masterclass_regs_email_key USING btree (email)
- UNIQUE INDEX testprep_masterclass_regs_pkey USING btree (id)

### public.research_masterclass_regs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| full_name | text | NOT NULL |  |
| email | text | NOT NULL |  |
| phone | text | null |  |
| country | text | null |  |
| grade | text | null |  |
| role | text | NOT NULL |  |
| selected_sessions | text[] | NOT NULL | '{}'::text[] |
| school_name | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| student_name | text | null |  |
| student_grade | text | null |  |
| designation | text | null |  |
| utm_source | text | null | 'direct'::text |
| utm_medium | text | null | 'website'::text |
| utm_campaign | text | null | 'direct'::text |
| created_at | timestamp with time zone | NOT NULL | now() |
| category | text | null |  |

Constraints:
- `research_masterclass_regs_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_research_masterclass_regs_created_at USING btree (created_at DESC)
- INDEX idx_research_masterclass_regs_email USING btree (email)
- UNIQUE INDEX research_masterclass_regs_pkey USING btree (id)

### public.stripe_payments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL |  |
| created_date_utc | timestamp without time zone | null |  |
| customer_email | text | null |  |
| amount | numeric(12,2) | null |  |
| currency | text | null |  |
| checkout_line_item_summary | text | null |  |
| payment_intent_id | text | null |  |
| status | text | null |  |
| card_name | text | null |  |
| plink | text | null |  |
| student_name | text | null |  |
| student_email | text | null |  |
| school_name | text | null |  |
| program_code | text | null |  |
| metadata | jsonb | null | '{}'::jsonb |
| processing_status | text | null |  |
| reference | text | null |  |
| payout_id | text | null |  |
| export_status | text | null |  |
| program_type | text | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |

Constraints:
- `stripe_payments_pkey` PRIMARY KEY (id)
- `stripe_payments_pi_unique` UNIQUE (payment_intent_id)

Indexes:
- INDEX idx_sp_program_type USING btree (program_type)
- INDEX idx_sp_utm_campaign USING btree (utm_campaign) WHERE ((utm_campaign IS NOT NULL) AND (utm_campaign <> ''::text))
- INDEX idx_sp_utm_source USING btree (utm_source) WHERE ((utm_source IS NOT NULL) AND (utm_source <> ''::text))
- UNIQUE INDEX stripe_payments_pi_unique USING btree (payment_intent_id)
- UNIQUE INDEX stripe_payments_pkey USING btree (id)

### public.leaders_applications

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| full_name | text | NOT NULL |  |
| parent_details | text | null |  |
| school | text | null |  |
| grade | text | null |  |
| city_country | text | null |  |
| email | text | NOT NULL |  |
| phone | text | null |  |
| path | text | NOT NULL |  |
| path_reason | text | NOT NULL |  |
| declaration_confirmed | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| student_dob | date | null |  |
| country | text | null |  |
| state | text | null |  |
| city | text | null |  |
| parent_name | text | null |  |
| parent_phone | text | null |  |
| parent_email | text | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |

Constraints:
- `applications_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX applications_pkey USING btree (id)

### public.week_without_walls_inquiries

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| created_at | timestamp with time zone | null |  |
| updated_at | timestamp with time zone | null |  |
| synced_at | timestamp with time zone | null |  |
| school_name | text | null |  |
| contact_name | text | null |  |
| contact_role | text | null |  |
| email | text | null |  |
| phone | text | null |  |
| country | text | null |  |
| city | text | null |  |
| programs | text[] | null |  |
| student_count | integer | null |  |
| grade_levels | text[] | null |  |
| preferred_dates | text | null |  |
| backup_dates | text | null |  |
| notes | text | null |  |
| source | text | null |  |
| email_sent | boolean | NOT NULL | false |
| email_sent_at | timestamp with time zone | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| utm_content | text | null |  |
| utm_term | text | null |  |

Constraints:
- `week_without_walls_inquiries_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX week_without_walls_inquiries_pkey USING btree (id)
