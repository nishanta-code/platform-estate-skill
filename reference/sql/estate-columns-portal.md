# Shared estate columns: portal (22 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### portal.app_settings

| column | type | nullable | default |
|---|---|---|---|
| key | text | NOT NULL |  |
| value | jsonb | NOT NULL | '{}'::jsonb |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `app_settings_pkey` PRIMARY KEY (key)

Indexes:
- UNIQUE INDEX app_settings_pkey USING btree (key)

### portal.application_installments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| application_id | uuid | NOT NULL |  |
| plan_index | integer | NOT NULL |  |
| label | text | NOT NULL |  |
| stripe_price_id | text | NOT NULL |  |
| amount | numeric | null |  |
| currency | text | null |  |
| status | text | NOT NULL | 'pending'::text |
| due_date | date | null |  |
| payment_intent_id | text | null |  |
| paid_at | timestamp with time zone | null |  |
| last_reminder_sent_at | timestamp with time zone | null |  |
| reminder_count | integer | NOT NULL | 0 |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| autopay_status | text | null |  |
| autopay_payment_method_id | text | null |  |
| autopay_customer_id | text | null |  |
| autopay_intent_id | text | null |  |
| autopay_attempted_at | timestamp with time zone | null |  |
| autopay_failure_reason | text | null |  |
| autopay_email_sent_at | timestamp with time zone | null |  |

Constraints:
- `application_installments_application_id_fkey` FOREIGN KEY (application_id) REFERENCES applications(id) ON DELETE CASCADE
- `application_installments_pkey` PRIMARY KEY (id)
- `application_installments_application_id_plan_index_key` UNIQUE (application_id, plan_index)

Indexes:
- UNIQUE INDEX application_installments_application_id_plan_index_key USING btree (application_id, plan_index)
- INDEX application_installments_autopay_idx USING btree (autopay_status, due_date) WHERE (autopay_status = 'armed'::text)
- UNIQUE INDEX application_installments_pkey USING btree (id)
- INDEX idx_application_installments_app USING btree (application_id, plan_index)
- INDEX idx_application_installments_pending USING btree (status, due_date) WHERE (status = 'pending'::text)

### portal.camp_leads

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| camp_key | text | NOT NULL |  |
| source_table | text | NOT NULL |  |
| source_row_id | text | NOT NULL |  |
| student_name | text | null |  |
| student_email | text | null |  |
| student_phone | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| grade | text | null |  |
| school_name | text | null |  |
| country | text | null |  |
| first_seen_at | timestamp with time zone | NOT NULL | now() |
| thankyou_sent_at | timestamp with time zone | null |  |
| offer_sent_at | timestamp with time zone | null |  |
| reminders_sent | integer | NOT NULL | 0 |
| last_reminder_at | timestamp with time zone | null |  |
| paid | boolean | NOT NULL | false |
| paid_at | timestamp with time zone | null |  |
| stopped | boolean | NOT NULL | false |
| stop_reason | text | null |  |
| raw | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| paid_amount | numeric | null |  |
| paid_currency | text | null |  |
| paid_email | text | null |  |
| payment_ref | text | null |  |
| paid_source | text | null |  |
| flag_channel | text | null |  |
| flag_reason | text | null |  |
| flag_detail | text | null |  |
| flagged_at | timestamp with time zone | null |  |
| flag_hard | boolean | NOT NULL | false |
| flag_attempts | integer | NOT NULL | 0 |
| flag_retry_after | timestamp with time zone | null |  |
| finals_sent | integer | NOT NULL | 0 |
| last_final_at | timestamp with time zone | null |  |
| wa_last_sent_at | timestamp with time zone | null |  |

Constraints:
- `camp_leads_pkey` PRIMARY KEY (id)
- `camp_leads_source_table_source_row_id_key` UNIQUE (source_table, source_row_id)

Indexes:
- INDEX camp_leads_camp_idx USING btree (camp_key)
- INDEX camp_leads_email_idx USING btree (lower(student_email))
- INDEX camp_leads_flagged_idx USING btree (flagged_at) WHERE (flag_reason IS NOT NULL)
- INDEX camp_leads_payment_ref_idx USING btree (payment_ref)
- UNIQUE INDEX camp_leads_pkey USING btree (id)
- UNIQUE INDEX camp_leads_source_table_source_row_id_key USING btree (source_table, source_row_id)

### portal.camp_payments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| camp_key | text | NOT NULL |  |
| price_id | text | null |  |
| payment_intent_id | text | null |  |
| checkout_session_id | text | null |  |
| amount | numeric | null |  |
| currency | text | null |  |
| customer_email | text | null |  |
| matched_lead_id | uuid | null |  |
| match_method | text | null |  |
| raw | jsonb | null |  |
| paid_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `camp_payments_matched_lead_id_fkey` FOREIGN KEY (matched_lead_id) REFERENCES portal.camp_leads(id) ON DELETE SET NULL
- `camp_payments_pkey` PRIMARY KEY (id)
- `camp_payments_payment_intent_id_key` UNIQUE (payment_intent_id)

Indexes:
- UNIQUE INDEX camp_payments_payment_intent_id_key USING btree (payment_intent_id)
- UNIQUE INDEX camp_payments_pkey USING btree (id)

### portal.captured_leads

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| lead_type | text | NOT NULL | 'form'::text |
| source | text | null |  |
| status | text | NOT NULL | 'new'::text |
| converted | boolean | NOT NULL | false |
| converted_at | timestamp with time zone | null |  |
| student_name | text | null |  |
| student_email | text | null |  |
| student_phone | text | null |  |
| student_age | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| grade | text | null |  |
| school_name | text | null |  |
| city | text | null |  |
| state | text | null |  |
| country | text | null |  |
| program_code | text | null |  |
| program_name | text | null |  |
| program_category | text | null |  |
| camp_interest | text | null |  |
| interested_programs | text[] | null |  |
| message | text | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| landing_page | text | null |  |
| device_type | text | null |  |
| raw_data | jsonb | NOT NULL | '{}'::jsonb |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `captured_leads_pkey` PRIMARY KEY (id)

Indexes:
- INDEX captured_leads_lead_type_created_idx USING btree (lead_type, created_at DESC)
- INDEX captured_leads_parent_email_idx USING btree (lower(parent_email))
- UNIQUE INDEX captured_leads_pkey USING btree (id)
- INDEX captured_leads_program_code_idx USING btree (program_code)
- INDEX captured_leads_source_idx USING btree (source)
- INDEX captured_leads_student_email_idx USING btree (lower(student_email))

### portal.email_reminder_settings

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| auto_reminders_enabled | boolean | NOT NULL | true |
| last_run_at | timestamp with time zone | null |  |
| updated_at | timestamp with time zone | NOT NULL | now() |
| installment_reminder_last_run_at | timestamp with time zone | null |  |

Constraints:
- `email_reminder_settings_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX email_reminder_settings_pkey USING btree (id)

### portal.email_send_log

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| student_id | text | null |  |
| template_slug | text | null |  |
| recipient_email | text | NOT NULL |  |
| status | text | NOT NULL |  |
| error_message | text | null |  |
| triggered_by | text | NOT NULL | 'manual'::text |
| triggered_by_user | uuid | null |  |
| sent_at | timestamp with time zone | NOT NULL | now() |
| subject | text | null |  |
| cc_emails | text[] | null |  |

Constraints:
- `email_send_log_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX email_send_log_pkey USING btree (id)
- INDEX idx_email_send_log_app USING btree (student_id, sent_at DESC)
- INDEX idx_email_send_log_recipient_template USING btree (recipient_email, template_slug)
- INDEX idx_email_send_log_sent_at USING btree (sent_at DESC)

### portal.email_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| slug | text | NOT NULL |  |
| name | text | NOT NULL |  |
| subject | text | NOT NULL |  |
| html_body | text | NOT NULL |  |
| program_code | text | null |  |
| is_active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| body_text | text | null |  |
| description | text | null |  |
| variables | text[] | null | '{}'::text[] |
| trigger_type | text | NOT NULL | 'manual'::text |
| sender_name | text | NOT NULL | 'the Group'::text |
| category | text | null |  |

Constraints:
- `email_templates_pkey` PRIMARY KEY (id)
- `email_templates_slug_key` UNIQUE (slug)

Indexes:
- UNIQUE INDEX email_templates_pkey USING btree (id)
- UNIQUE INDEX email_templates_slug_key USING btree (slug)
- UNIQUE INDEX email_templates_trigger_category_uidx USING btree (trigger_type, slug, category) WHERE ((program_code IS NULL) AND (category IS NOT NULL))
- INDEX idx_email_templates_trigger_type USING btree (trigger_type, is_active)

### portal.form_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| slug | text | NOT NULL |  |
| program_code | text | null |  |
| schema | jsonb | NOT NULL | '[]'::jsonb |
| settings | jsonb | NOT NULL | '{}'::jsonb |
| is_active | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| program_name_override | text | null |  |

Constraints:
- `form_templates_pkey` PRIMARY KEY (id)
- `form_templates_slug_key` UNIQUE (slug)

Indexes:
- UNIQUE INDEX form_templates_pkey USING btree (id)
- UNIQUE INDEX form_templates_slug_key USING btree (slug)

### portal.legacy_form_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| slug | text | NOT NULL |  |
| status | text | NOT NULL | 'draft'::text |
| schema | jsonb | NOT NULL | '[]'::jsonb |
| settings | jsonb | NOT NULL | '{}'::jsonb |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `legacy_form_templates_status_chk` CHECK ((status = ANY (ARRAY['draft'::text, 'live'::text, 'closed'::text])))
- `legacy_form_templates_pkey` PRIMARY KEY (id)
- `legacy_form_templates_slug_key` UNIQUE (slug)

Indexes:
- UNIQUE INDEX legacy_form_templates_name_unique USING btree (lower(btrim(name)))
- UNIQUE INDEX legacy_form_templates_pkey USING btree (id)
- UNIQUE INDEX legacy_form_templates_slug_key USING btree (slug)

### portal.masterclass_form_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| masterclass_id | uuid | NOT NULL |  |
| partner_id | uuid | NOT NULL |  |
| name | text | NOT NULL |  |
| slug | text | NOT NULL |  |
| status | text | NOT NULL | 'draft'::text |
| schema | jsonb | NOT NULL | '[]'::jsonb |
| base_fields | jsonb | NOT NULL | '{"school_name": true, "attendee_type": true, "parent_details": false}'::jsonb |
| settings | jsonb | NOT NULL | '{}'::jsonb |
| allow_multi_session | boolean | NOT NULL | false |
| linked_session_ids | uuid[] | NOT NULL | '{}'::uuid[] |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `masterclass_form_templates_status_check` CHECK ((status = ANY (ARRAY['draft'::text, 'live'::text, 'closed'::text])))
- `masterclass_form_templates_masterclass_id_fkey` FOREIGN KEY (masterclass_id) REFERENCES all_masterclasses(id)
- `masterclass_form_templates_partner_id_fkey` FOREIGN KEY (partner_id) REFERENCES all_partners(id)
- `masterclass_form_templates_pkey` PRIMARY KEY (id)
- `masterclass_form_templates_slug_key` UNIQUE (slug)

Indexes:
- INDEX masterclass_form_templates_masterclass_id_idx USING btree (masterclass_id)
- INDEX masterclass_form_templates_partner_id_idx USING btree (partner_id)
- UNIQUE INDEX masterclass_form_templates_pkey USING btree (id)
- UNIQUE INDEX masterclass_form_templates_slug_key USING btree (slug)

### portal.megan_pending_actions

| column | type | nullable | default |
|---|---|---|---|
| call_id | text | NOT NULL |  |
| path | text | NOT NULL |  |
| reason | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `megan_pending_actions_pkey` PRIMARY KEY (call_id)

Indexes:
- UNIQUE INDEX megan_pending_actions_pkey USING btree (call_id)

### portal.mentors

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| full_name | text | NOT NULL |  |
| bio | text | null |  |
| image_url | text | null |  |
| university | text | null |  |
| program_associations | text[] | null | '{}'::text[] |
| is_active | boolean | null | true |
| display_order | integer | null | 0 |
| created_at | timestamp with time zone | null | now() |
| updated_at | timestamp with time zone | null | now() |
| user_id | uuid | null |  |

Constraints:
- `mentors_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX mentors_pkey USING btree (id)
- INDEX mentors_user_id_idx USING btree (user_id)

### portal.newsletter_subscribers

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| email | text | NOT NULL |  |
| subscribed_at | timestamp with time zone | null | now() |
| source | text | null | 'footer'::text |
| status | text | null | 'active'::text |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |

Constraints:
- `newsletter_subscribers_pkey` PRIMARY KEY (id)
- `newsletter_subscribers_email_unique` UNIQUE (email)

Indexes:
- UNIQUE INDEX newsletter_subscribers_email_unique USING btree (email)
- UNIQUE INDEX newsletter_subscribers_pkey USING btree (id)

### portal.payment_links

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| slug | text | NOT NULL |  |
| name | text | NOT NULL |  |
| form_template_id | uuid | null |  |
| program_code | text | null |  |
| currency | text | NOT NULL | 'USD'::text |
| full_amount | numeric | null |  |
| installments | jsonb | NOT NULL | '[]'::jsonb |
| installments_enabled | boolean | NOT NULL | false |
| note | text | null |  |
| is_active | boolean | NOT NULL | true |
| expires_at | timestamp with time zone | null |  |
| max_uses | integer | null |  |
| uses_count | integer | NOT NULL | 0 |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| program_id | uuid | null |  |
| partner_id | uuid | null |  |
| apply_url | text | null |  |
| installments_autopay_enabled | boolean | NOT NULL | false |
| deal_type | text | NOT NULL | 'b2c'::text |
| cohort_code | text | null |  |
| alt_currency | jsonb | null |  |

Constraints:
- `payment_links_deal_type_chk` CHECK ((deal_type = ANY (ARRAY['b2c'::text, 'b2b'::text])))
- `payment_links_form_template_id_fkey` FOREIGN KEY (form_template_id) REFERENCES portal.form_templates(id) ON DELETE CASCADE
- `payment_links_partner_id_fkey` FOREIGN KEY (partner_id) REFERENCES all_partners(id) ON DELETE SET NULL
- `payment_links_program_id_fkey` FOREIGN KEY (program_id) REFERENCES all_programs(id) ON DELETE SET NULL
- `payment_links_pkey` PRIMARY KEY (id)
- `payment_links_slug_key` UNIQUE (slug)

Indexes:
- UNIQUE INDEX payment_links_pkey USING btree (id)
- INDEX payment_links_program_id_idx USING btree (program_id)
- UNIQUE INDEX payment_links_slug_key USING btree (slug)
- INDEX payment_links_template_idx USING btree (form_template_id)

### portal.program_attachments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_code | text | NOT NULL |  |
| attachment_type | text | NOT NULL | 'letter_of_acceptance'::text |
| file_path | text | NOT NULL |  |
| file_name | text | NOT NULL |  |
| output_filename_template | text | NOT NULL | '{{student_name}} - Letter of Acceptance.pdf'::text |
| font_family | text | NOT NULL | 'Montserrat-Bold'::text |
| font_size | numeric | NOT NULL | 14 |
| font_color | text | NOT NULL | '#1a1a1a'::text |
| name_prefix | text | NOT NULL | ''::text |
| name_suffix | text | NOT NULL | ''::text |
| x_offset | numeric | NOT NULL | 0 |
| y_offset | numeric | NOT NULL | 0 |
| is_active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_attachments_pkey` PRIMARY KEY (id)
- `program_attachments_program_code_key` UNIQUE (program_code)

Indexes:
- UNIQUE INDEX program_attachments_pkey USING btree (id)
- UNIQUE INDEX program_attachments_program_code_key USING btree (program_code)

### portal.program_email_settings

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_code | text | NOT NULL |  |
| program_name | text | null |  |
| application_confirmation_enabled | boolean | NOT NULL | true |
| payment_confirmation_enabled | boolean | NOT NULL | true |
| letter_of_acceptance_enabled | boolean | NOT NULL | false |
| reminder_emails_enabled | boolean | NOT NULL | true |
| start_date | date | null |  |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_email_settings_pkey` PRIMARY KEY (id)
- `program_email_settings_program_code_key` UNIQUE (program_code)

Indexes:
- UNIQUE INDEX program_email_settings_pkey USING btree (id)
- UNIQUE INDEX program_email_settings_program_code_key USING btree (program_code)

### portal.rate_limit_hits

| column | type | nullable | default |
|---|---|---|---|
| id | bigint | NOT NULL | nextval('portal.rate_limit_hits_id_seq'::regclass) |
| key | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `rate_limit_hits_pkey` PRIMARY KEY (id)

Indexes:
- INDEX rate_limit_hits_key_created_idx USING btree (key, created_at DESC)
- UNIQUE INDEX rate_limit_hits_pkey USING btree (id)

### portal.school_program_inquiries

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| school_name | text | NOT NULL |  |
| contact_person_name | text | NOT NULL |  |
| contact_role | text | NOT NULL |  |
| email | text | NOT NULL |  |
| phone | text | NOT NULL |  |
| website | text | null |  |
| city | text | NOT NULL |  |
| country | text | NOT NULL |  |
| selected_programs | text[] | NOT NULL | '{}'::text[] |
| preferred_date_ranges | jsonb | NOT NULL | '[]'::jsonb |
| expected_students | text | NOT NULL |  |
| request_custom_quote | boolean | NOT NULL | false |
| special_requirements | text | null |  |
| hear_about_us | text | null |  |
| status | text | NOT NULL | 'new'::text |
| admin_notes | text | null |  |
| estimated_total_price | numeric | null |  |
| curriculum_type | text | null |  |
| student_gender | text | null | 'co_ed'::text |
| assigned_to | uuid | null |  |
| follow_up_date | date | null |  |
| archived_at | timestamp with time zone | null |  |
| archived_reason | text | null |  |

Constraints:
- `valid_status` CHECK ((status = ANY (ARRAY['new'::text, 'contacted'::text, 'scheduled'::text, 'confirmed'::text, 'completed'::text, 'declined'::text])))
- `school_program_inquiries_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_school_inquiries_created_at USING btree (created_at DESC)
- INDEX idx_school_inquiries_status USING btree (status)
- UNIQUE INDEX school_program_inquiries_pkey USING btree (id)

### portal.user_profiles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| role | portal.app_role | NOT NULL | 'student'::portal.app_role |
| full_name | text | null |  |
| first_name | text | NOT NULL | ''::text |
| last_name | text | NOT NULL | ''::text |
| email | text | null |  |
| phone_number | text | null |  |
| country | text | null |  |
| date_of_birth | date | null |  |
| school_name | text | null |  |
| school_email | text | null |  |
| grade_level | text | null |  |
| parent_name | text | null |  |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| bio | text | null |  |
| avatar_url | text | null |  |
| profile_completed | boolean | NOT NULL | false |
| is_active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| middle_name | text | null | ''::text |

Constraints:
- `user_profiles_pkey` PRIMARY KEY (id)
- `user_profiles_user_id_key` UNIQUE (user_id)

Indexes:
- UNIQUE INDEX user_profiles_pkey USING btree (id)
- UNIQUE INDEX user_profiles_user_id_key USING btree (user_id)

### portal.whatsapp_drip_sequences

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| description | text | null |  |
| trigger_event | text | NOT NULL | 'application_created'::text |
| audience | text | NOT NULL | 'student'::text |
| program_codes | text[] | NOT NULL | '{}'::text[] |
| steps | jsonb | NOT NULL | '[]'::jsonb |
| enabled | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `whatsapp_drip_sequences_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX whatsapp_drip_sequences_pkey USING btree (id)

### portal.whatsapp_undeliverable_numbers

| column | type | nullable | default |
|---|---|---|---|
| phone | text | NOT NULL |  |
| country_code | text | null |  |
| national_number | text | NOT NULL |  |
| country | text | NOT NULL |  |
| last_failure_date | date | NOT NULL |  |
| times_failed | integer | NOT NULL | 1 |
| last_campaign | text | null |  |
| source_file | text | NOT NULL |  |
| first_imported_at | timestamp with time zone | NOT NULL | now() |
| last_imported_at | timestamp with time zone | NOT NULL | now() |
| blocked | boolean | NOT NULL | true |
| unblocked_at | timestamp with time zone | null |  |
| unblocked_by | uuid | null |  |
| unblock_reason | text | null |  |
| last_action_at | timestamp with time zone | NOT NULL | now() |
| last_action_by | uuid | null |  |
| lms_block_reason | text | NOT NULL | 'undelivered'::text |

Constraints:
- `whatsapp_undeliverable_numbers_lms_block_reason_check` CHECK ((lms_block_reason = ANY (ARRAY['blocked'::text, 'undelivered'::text])))
- `whatsapp_undeliverable_numbers_phone_check` CHECK ((phone ~ '^[0-9]{1,32}$'::text))
- `whatsapp_undeliverable_numbers_times_failed_check` CHECK ((times_failed > 0))
- `whatsapp_undeliverable_numbers_pkey` PRIMARY KEY (phone)

Indexes:
- INDEX whatsapp_undeliverable_blocked_idx USING btree (blocked, last_failure_date DESC, phone)
- UNIQUE INDEX whatsapp_undeliverable_numbers_pkey USING btree (phone)
