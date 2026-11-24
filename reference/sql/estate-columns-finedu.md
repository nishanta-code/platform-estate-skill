# Shared estate columns: finedu (22 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### finedu.ai_agent_logs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| agent_name | text | NOT NULL |  |
| action_taken | text | NOT NULL |  |
| status | text | NOT NULL |  |
| metadata | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `ai_agent_logs_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX ai_agent_logs_pkey USING btree (id)
- INDEX idx_ai_agent_logs_created_at USING btree (created_at DESC)

### finedu.app_settings

| column | type | nullable | default |
|---|---|---|---|
| key | text | NOT NULL |  |
| value | jsonb | null |  |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `app_settings_pkey` PRIMARY KEY (key)

Indexes:
- UNIQUE INDEX app_settings_pkey USING btree (key)

### finedu.assessment_questions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| category | text | NOT NULL |  |
| prompt | text | NOT NULL |  |
| follow_up_prompt | text | null |  |
| max_score | integer | NOT NULL | 20 |
| active | boolean | NOT NULL | true |
| sort_order | integer | NOT NULL | 0 |
| created_at | timestamp with time zone | NOT NULL | now() |
| choices | jsonb | null |  |
| choice_kind | text | null |  |
| min_picks | integer | null | 1 |
| max_picks | integer | null | 1 |

Constraints:
- `assessment_questions_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX assessment_questions_pkey USING btree (id)

### finedu.catalogue_audit

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| entity | text | NOT NULL |  |
| entity_id | uuid | null |  |
| action | text | NOT NULL |  |
| actor | text | null |  |
| changes | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `catalogue_audit_pkey` PRIMARY KEY (id)

Indexes:
- INDEX catalogue_audit_entity_idx USING btree (entity, entity_id, created_at DESC)
- UNIQUE INDEX catalogue_audit_pkey USING btree (id)

### finedu.concierge_leads

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| channel | finedu.lead_channel | NOT NULL | 'web'::finedu.lead_channel |
| parent_email | text | null |  |
| parent_phone | text | null |  |
| parent_first_name | text | null |  |
| child_age | integer | null |  |
| child_interest | text | null |  |
| raw_transcript | jsonb | null |  |
| score | integer | null |  |
| score_reason | text | null |  |
| status | finedu.lead_status | NOT NULL | 'new'::finedu.lead_status |
| utm | jsonb | null |  |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| voucher_amount_usd | integer | null |  |
| voucher_code | text | null |  |
| voucher_expires_at | timestamp with time zone | null |  |
| voucher_redeemed_at | timestamp with time zone | null |  |
| assessment_completed_at | timestamp with time zone | null |  |
| assessment_strength_label | text | null |  |
| student_first_name | text | null |  |

Constraints:
- `concierge_leads_pkey` PRIMARY KEY (id)
- `concierge_leads_voucher_code_key` UNIQUE (voucher_code)

Indexes:
- UNIQUE INDEX concierge_leads_pkey USING btree (id)
- UNIQUE INDEX concierge_leads_voucher_code_key USING btree (voucher_code)
- INDEX idx_concierge_leads_channel USING btree (channel)
- INDEX idx_concierge_leads_created_at USING btree (created_at DESC)
- INDEX idx_concierge_leads_status USING btree (status)

### finedu.email_attachments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_key | text | NOT NULL |  |
| trigger_type | text | NOT NULL |  |
| file_path | text | NOT NULL |  |
| display_name | text | NOT NULL |  |
| is_personalized | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `email_attachments_pkey` PRIMARY KEY (id)
- `email_attachments_cohort_key_trigger_type_file_path_key` UNIQUE (cohort_key, trigger_type, file_path)

Indexes:
- UNIQUE INDEX email_attachments_cohort_key_trigger_type_file_path_key USING btree (cohort_key, trigger_type, file_path)
- UNIQUE INDEX email_attachments_pkey USING btree (id)

### finedu.email_send_log

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| message_id | text | null |  |
| template_name | text | NOT NULL |  |
| recipient_email | text | NOT NULL |  |
| status | text | NOT NULL |  |
| error_message | text | null |  |
| metadata | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| application_id | uuid | null |  |
| cc_emails | text[] | null |  |

Constraints:
- `email_send_log_status_check` CHECK ((status = ANY (ARRAY['pending'::text, 'sent'::text, 'suppressed'::text, 'failed'::text, 'bounced'::text, 'complained'::text, 'dlq'::text])))
- `email_send_log_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX email_send_log_masterclass_uniq USING btree (application_id, template_name) WHERE (template_name = 'masterclass_confirmation'::text)
- UNIQUE INDEX email_send_log_pkey USING btree (id)
- INDEX idx_email_send_log_created USING btree (created_at DESC)
- INDEX idx_email_send_log_message USING btree (message_id)
- UNIQUE INDEX idx_email_send_log_message_sent_unique USING btree (message_id) WHERE (status = 'sent'::text)
- INDEX idx_email_send_log_recipient USING btree (recipient_email)

### finedu.email_send_state

| column | type | nullable | default |
|---|---|---|---|
| id | integer | NOT NULL | 1 |
| retry_after_until | timestamp with time zone | null |  |
| batch_size | integer | NOT NULL | 10 |
| send_delay_ms | integer | NOT NULL | 200 |
| auth_email_ttl_minutes | integer | NOT NULL | 15 |
| transactional_email_ttl_minutes | integer | NOT NULL | 60 |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `email_send_state_id_check` CHECK ((id = 1))
- `email_send_state_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX email_send_state_pkey USING btree (id)

### finedu.email_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| subject | text | NOT NULL |  |
| html_body | text | NOT NULL |  |
| body_text | text | null |  |
| sender_name | text | NOT NULL | 'Financial Literacy Fellowship'::text |
| trigger_type | text | NOT NULL | 'manual'::text |
| is_active | boolean | NOT NULL | false |
| variables | jsonb | null |  |
| description | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| program_key | text | null |  |

Constraints:
- `email_templates_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX email_templates_active_trigger_program_key USING btree (trigger_type, COALESCE(program_key, ''::text)) WHERE (is_active = true)
- UNIQUE INDEX email_templates_name_key USING btree (name)
- UNIQUE INDEX email_templates_pkey USING btree (id)
- INDEX idx_email_templates_trigger_program USING btree (trigger_type, program_key)

### finedu.email_unsubscribe_tokens

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| token | text | NOT NULL |  |
| email | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| used_at | timestamp with time zone | null |  |

Constraints:
- `email_unsubscribe_tokens_pkey` PRIMARY KEY (id)
- `email_unsubscribe_tokens_email_key` UNIQUE (email)
- `email_unsubscribe_tokens_token_key` UNIQUE (token)

Indexes:
- UNIQUE INDEX email_unsubscribe_tokens_email_key USING btree (email)
- UNIQUE INDEX email_unsubscribe_tokens_pkey USING btree (id)
- UNIQUE INDEX email_unsubscribe_tokens_token_key USING btree (token)
- INDEX idx_unsubscribe_tokens_token USING btree (token)

### finedu.flywire_webhook_events

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| event_type | text | null |  |
| payment_id | text | null |  |
| reference | text | null |  |
| payload | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `flywire_webhook_events_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX flywire_webhook_events_pkey USING btree (id)

### finedu.page_events

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| session_id | text | null |  |
| path | text | null |  |
| event | text | NOT NULL |  |
| props | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `page_events_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_page_events_created_at USING btree (created_at DESC)
- INDEX idx_page_events_event USING btree (event)
- UNIQUE INDEX page_events_pkey USING btree (id)

### finedu.page_visits

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| path | text | NOT NULL |  |
| session_id | text | null |  |
| referrer | text | null |  |
| utm_source | text | null |  |
| utm_medium | text | null |  |
| utm_campaign | text | null |  |
| utm_term | text | null |  |
| utm_content | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `page_visits_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_page_visits_created_at USING btree (created_at DESC)
- UNIQUE INDEX page_visits_pkey USING btree (id)

### finedu.payment_reminder_log

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| application_id | uuid | NOT NULL |  |
| cohort | text | null |  |
| reminder_no | integer | NOT NULL | 1 |
| channel | text | NOT NULL | 'email'::text |
| recipient | text | null |  |
| trigger_source | text | NOT NULL | 'cron'::text |
| sent_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `payment_reminder_log_pkey` PRIMARY KEY (id)

Indexes:
- INDEX payment_reminder_log_app_idx USING btree (application_id)
- UNIQUE INDEX payment_reminder_log_pkey USING btree (id)
- INDEX payment_reminder_log_sent_idx USING btree (sent_at DESC)

### finedu.program_email_settings

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| cohort_key | text | NOT NULL |  |
| application_emails_enabled | boolean | NOT NULL | true |
| payment_confirmation_enabled | boolean | NOT NULL | true |
| payment_reminder_enabled | boolean | NOT NULL | true |
| letter_of_acceptance_enabled | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| complete_application_enabled | boolean | NOT NULL | true |

Constraints:
- `program_email_settings_pkey` PRIMARY KEY (id)
- `program_email_settings_cohort_key_key` UNIQUE (cohort_key)

Indexes:
- UNIQUE INDEX program_email_settings_cohort_key_key USING btree (cohort_key)
- UNIQUE INDEX program_email_settings_pkey USING btree (id)

### finedu.program_map

| column | type | nullable | default |
|---|---|---|---|
| program_key | text | NOT NULL |  |
| program_id | uuid | NOT NULL |  |
| program_label | text | NOT NULL |  |
| code_prefix | text | NOT NULL |  |
| duration_days | integer | NOT NULL | 56 |

Constraints:
- `program_map_program_id_fkey` FOREIGN KEY (program_id) REFERENCES all_programs(id) ON DELETE CASCADE
- `program_map_pkey` PRIMARY KEY (program_key)
- `program_map_program_id_key` UNIQUE (program_id)

Indexes:
- UNIQUE INDEX program_map_pkey USING btree (program_key)
- UNIQUE INDEX program_map_program_id_key USING btree (program_id)

### finedu.school_proposals

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| slug | text | NOT NULL |  |
| school_name | text | NOT NULL |  |
| contact_name | text | null |  |
| contact_title | text | null |  |
| contact_email | text | null |  |
| city | text | null |  |
| seats | integer | NOT NULL | 30 |
| price_per_student_usd | numeric | NOT NULL | 299 |
| original_price_usd | numeric | NOT NULL | 499 |
| cohort_key | text | null |  |
| custom_inclusions | text[] | NOT NULL | '{}'::text[] |
| notes | text | null |  |
| valid_until | date | null |  |
| stripe_payment_link | text | null |  |
| stripe_payment_link_id | text | null |  |
| status | text | NOT NULL | 'draft'::text |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `school_proposals_pkey` PRIMARY KEY (id)
- `school_proposals_slug_key` UNIQUE (slug)

Indexes:
- UNIQUE INDEX school_proposals_pkey USING btree (id)
- INDEX school_proposals_slug_idx USING btree (slug)
- UNIQUE INDEX school_proposals_slug_key USING btree (slug)
- INDEX school_proposals_status_idx USING btree (status)

### finedu.suppressed_emails

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| email | text | NOT NULL |  |
| reason | text | NOT NULL |  |
| metadata | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `suppressed_emails_reason_check` CHECK ((reason = ANY (ARRAY['unsubscribe'::text, 'bounce'::text, 'complaint'::text])))
- `suppressed_emails_pkey` PRIMARY KEY (id)
- `suppressed_emails_email_key` UNIQUE (email)

Indexes:
- INDEX idx_suppressed_emails_email USING btree (email)
- UNIQUE INDEX suppressed_emails_email_key USING btree (email)
- UNIQUE INDEX suppressed_emails_pkey USING btree (id)

### finedu.unmatched_payments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| payment_intent_id | text | null |  |
| stripe_session_id | text | null |  |
| email | text | null |  |
| customer_name | text | null |  |
| amount | numeric | null |  |
| currency | text | null |  |
| cohort | text | null |  |
| program_code | text | null |  |
| paid_at | timestamp with time zone | null |  |
| status | text | NOT NULL | 'unmatched'::text |
| matched_application_id | uuid | null |  |
| matched_at | timestamp with time zone | null |  |
| reminder_count | integer | NOT NULL | 0 |
| last_reminder_at | timestamp with time zone | null |  |
| raw | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `unmatched_payments_pkey` PRIMARY KEY (id)
- `unmatched_payments_payment_intent_id_key` UNIQUE (payment_intent_id)

Indexes:
- UNIQUE INDEX unmatched_payments_payment_intent_id_key USING btree (payment_intent_id)
- UNIQUE INDEX unmatched_payments_pkey USING btree (id)

### finedu.user_roles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| role | finedu.app_role | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `user_roles_pkey` PRIMARY KEY (id)
- `user_roles_user_id_role_key` UNIQUE (user_id, role)

Indexes:
- UNIQUE INDEX user_roles_pkey USING btree (id)
- UNIQUE INDEX user_roles_user_id_role_key USING btree (user_id, role)

### finedu.voucher_redemptions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| lead_id | uuid | NOT NULL |  |
| application_id | uuid | null |  |
| voucher_code | text | NOT NULL |  |
| amount_usd | integer | NOT NULL |  |
| redeemed_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `voucher_redemptions_lead_id_fkey` FOREIGN KEY (lead_id) REFERENCES finedu.concierge_leads(id) ON DELETE CASCADE
- `voucher_redemptions_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX voucher_redemptions_pkey USING btree (id)

### finedu.whatsapp_messages

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| phone | text | NOT NULL |  |
| direction | text | NOT NULL |  |
| body | text | null |  |
| message_sid | text | null |  |
| lead_id | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| flow | text | null |  |
| audience | text | null |  |
| template_name | text | null |  |
| template_slug | text | null |  |
| application_id | uuid | null |  |
| status | text | null |  |
| error_message | text | null |  |
| triggered_by | text | null |  |
| wati_response | jsonb | null |  |

Constraints:
- `whatsapp_messages_direction_check` CHECK ((direction = ANY (ARRAY['in'::text, 'out'::text])))
- `whatsapp_messages_lead_id_fkey` FOREIGN KEY (lead_id) REFERENCES finedu.concierge_leads(id) ON DELETE SET NULL
- `whatsapp_messages_pkey` PRIMARY KEY (id)

Indexes:
- INDEX idx_whatsapp_messages_application USING btree (application_id)
- INDEX idx_whatsapp_messages_phone USING btree (phone)
- INDEX idx_whatsapp_messages_template_slug_phone USING btree (template_slug, phone, status)
- UNIQUE INDEX whatsapp_messages_pkey USING btree (id)
