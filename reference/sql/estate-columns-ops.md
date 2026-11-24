# Shared estate columns: ops (8 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### ops.agent_campaign_calls

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| campaign_id | uuid | NOT NULL |  |
| contact_id | uuid | null |  |
| is_test | boolean | NOT NULL | false |
| phone_e164 | text | NOT NULL |  |
| bland_call_id | text | null |  |
| status | text | NOT NULL | 'queued'::text |
| duration_seconds | integer | null |  |
| transcript | text | null |  |
| summary | text | null |  |
| recording_url | text | null |  |
| analysis | jsonb | null |  |
| email_sent_at | timestamp with time zone | null |  |
| email_to | text | null |  |
| handoff_sent_at | timestamp with time zone | null |  |
| error_message | text | null |  |
| initiated_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| email_delivery_status | text | NOT NULL | 'pending'::text |
| email_delivery_error | text | null |  |
| email_cc | text | null |  |
| whatsapp_delivery_status | text | NOT NULL | 'pending'::text |
| whatsapp_to | text | null |  |
| whatsapp_message_id | text | null |  |
| whatsapp_sent_at | timestamp with time zone | null |  |
| whatsapp_delivery_error | text | null |  |
| provider | text | null |  |
| provider_call_id | text | null |  |

Constraints:
- `agent_campaign_calls_whatsapp_delivery_status_check` CHECK ((whatsapp_delivery_status = ANY (ARRAY['pending'::text, 'sending'::text, 'accepted'::text, 'review_needed'::text])))
- `agent_campaign_calls_campaign_id_fkey` FOREIGN KEY (campaign_id) REFERENCES ops.agent_campaigns(id) ON DELETE CASCADE
- `agent_campaign_calls_contact_id_fkey` FOREIGN KEY (contact_id) REFERENCES ops.agent_campaign_contacts(id) ON DELETE SET NULL
- `agent_campaign_calls_pkey` PRIMARY KEY (id)
- `agent_campaign_calls_provider_call_id_key` UNIQUE (provider_call_id)

Indexes:
- INDEX agent_campaign_calls_campaign_idx USING btree (campaign_id, created_at DESC)
- UNIQUE INDEX agent_campaign_calls_pkey USING btree (id)
- UNIQUE INDEX agent_campaign_calls_provider_call_id_key USING btree (provider_call_id)

### ops.agent_campaign_contacts

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| campaign_id | uuid | NOT NULL |  |
| organisation_name | text | null |  |
| contact_name | text | null |  |
| contact_role | text | null |  |
| phone_e164 | text | NOT NULL |  |
| email | text | null |  |
| notes | text | null |  |
| status | text | NOT NULL | 'queued'::text |
| attempts | integer | NOT NULL | 0 |
| last_call_id | uuid | null |  |
| outcome | jsonb | null |  |
| do_not_call | boolean | NOT NULL | false |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| next_attempt_at | timestamp with time zone | null |  |
| last_error | text | null |  |
| retry_started_at | timestamp with time zone | null |  |

Constraints:
- `agent_campaign_contacts_campaign_id_fkey` FOREIGN KEY (campaign_id) REFERENCES ops.agent_campaigns(id) ON DELETE CASCADE
- `agent_campaign_contacts_pkey` PRIMARY KEY (id)
- `agent_campaign_contacts_campaign_id_phone_e164_key` UNIQUE (campaign_id, phone_e164)

Indexes:
- UNIQUE INDEX agent_campaign_contacts_campaign_id_phone_e164_key USING btree (campaign_id, phone_e164)
- UNIQUE INDEX agent_campaign_contacts_pkey USING btree (id)
- INDEX agent_campaign_contacts_queue_idx USING btree (campaign_id, status, next_attempt_at)

### ops.agent_campaigns

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| agent_name | text | NOT NULL | 'Hana'::text |
| organisation | text | NOT NULL |  |
| voice | text | null |  |
| language | text | NOT NULL | 'en'::text |
| status | text | NOT NULL | 'draft'::text |
| first_sentence | text | null |  |
| script | text | NOT NULL |  |
| knowledge | text | null |  |
| email_from_name | text | null |  |
| email_reply_to | text | null |  |
| email_subject | text | null |  |
| email_body_html | text | null |  |
| handoff_email | text | null |  |
| max_duration | integer | NOT NULL | 8 |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| dialer_running | boolean | NOT NULL | false |
| call_timezone | text | NOT NULL | 'Asia/Tokyo'::text |
| window_start_hour | integer | NOT NULL | 9 |
| window_end_hour | integer | NOT NULL | 17 |
| weekdays_only | boolean | NOT NULL | true |
| cooldown_seconds | integer | NOT NULL | 90 |
| max_concurrent | integer | NOT NULL | 1 |
| max_attempts | integer | NOT NULL | 3 |
| retry_gap_minutes | integer | NOT NULL | 240 |
| last_dial_at | timestamp with time zone | null |  |
| email_enabled | boolean | NOT NULL | true |
| whatsapp_config | jsonb | NOT NULL | '{"enabled": false, "mappings": [], "auto_send": false, "template_name": "", "allow_test_calls": false}'::jsonb |
| agent_id | uuid | null |  |
| apply_url | text | null |  |
| utm_campaign | text | null |  |
| discount_code | text | null |  |
| window_start_minute | integer | NOT NULL | 0 |
| session_type | text | null |  |
| program_name | text | null |  |
| session_name | text | null |  |
| session_date | text | null |  |
| session_time | text | null |  |
| session_timezone_label | text | null |  |
| session_location | text | null |  |
| break_start_hour | integer | null |  |
| break_start_minute | integer | NOT NULL | 0 |
| break_end_hour | integer | null |  |
| break_end_minute | integer | NOT NULL | 0 |
| retry_window_days | integer | null |  |

Constraints:
- `agent_campaigns_session_type_check` CHECK (((session_type IS NULL) OR (session_type = ANY (ARRAY['masterclass'::text, 'online_program'::text, 'in_person_conference'::text, 'in_person_program'::text, 'camp'::text]))))
- `agent_campaigns_status_check` CHECK ((status = ANY (ARRAY['draft'::text, 'active'::text, 'paused'::text, 'done'::text])))
- `agent_campaigns_agent_id_fkey` FOREIGN KEY (agent_id) REFERENCES ops.agent_profiles(id)
- `agent_campaigns_pkey` PRIMARY KEY (id)

Indexes:
- INDEX agent_campaigns_agent_id_idx USING btree (agent_id)
- UNIQUE INDEX agent_campaigns_pkey USING btree (id)

### ops.agent_profiles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| slug | text | NOT NULL |  |
| name | text | NOT NULL |  |
| organisation | text | NOT NULL |  |
| voice | text | null |  |
| language | text | NOT NULL | 'en'::text |
| purpose | text | null |  |
| behaviour_rules | text | null |  |
| brief_style | text | NOT NULL | 'custom'::text |
| status | text | NOT NULL | 'active'::text |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |
| retell_agent_id | text | null |  |
| retell_llm_id | text | null |  |
| retell_voice_id | text | null |  |
| retell_setup_status | text | NOT NULL | 'not_configured'::text |

Constraints:
- `agent_profiles_brief_style_check` CHECK ((brief_style = ANY (ARRAY['school_delegation'::text, 'custom'::text, 'broadcast'::text, 'school_outreach'::text, 'school_outreach_vn'::text, 'payment_followup'::text])))
- `agent_profiles_slug_check` CHECK ((slug ~ '^[a-z0-9]+(-[a-z0-9]+)*$'::text))
- `agent_profiles_status_check` CHECK ((status = ANY (ARRAY['active'::text, 'paused'::text, 'archived'::text])))
- `agent_profiles_pkey` PRIMARY KEY (id)
- `agent_profiles_retell_agent_id_key` UNIQUE (retell_agent_id)
- `agent_profiles_retell_llm_id_key` UNIQUE (retell_llm_id)
- `agent_profiles_slug_key` UNIQUE (slug)

Indexes:
- UNIQUE INDEX agent_profiles_pkey USING btree (id)
- UNIQUE INDEX agent_profiles_retell_agent_id_key USING btree (retell_agent_id)
- UNIQUE INDEX agent_profiles_retell_llm_id_key USING btree (retell_llm_id)
- UNIQUE INDEX agent_profiles_slug_key USING btree (slug)

### ops.contact_suppressions

| column | type | nullable | default |
|---|---|---|---|
| phone_e164 | text | NOT NULL |  |
| reason | text | NOT NULL | 'Previous do-not-call request'::text |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `contact_suppressions_pkey` PRIMARY KEY (phone_e164)

Indexes:
- UNIQUE INDEX contact_suppressions_pkey USING btree (phone_e164)

### ops.payment_followup_leads

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| campaign_id | uuid | NOT NULL |  |
| contact_id | uuid | NOT NULL |  |
| parent_name | text | null |  |
| parent_phone | text | NOT NULL |  |
| parent_email | text | null |  |
| student_names | text[] | NOT NULL | '{}'::text[] |
| programs | jsonb | NOT NULL | '[]'::jsonb |
| total_amount_due | numeric | null |  |
| currency | text | null | 'USD'::text |
| country | text | NOT NULL | 'United States'::text |
| status | text | NOT NULL | 'pending_call'::text |
| lead_score | integer | null |  |
| issue_tags | text[] | NOT NULL | '{}'::text[] |
| last_call_summary | text | null |  |
| last_call_at | timestamp with time zone | null |  |
| callback_requested_at | timestamp with time zone | null |  |
| payment_commitment_date | date | null |  |
| attempts | integer | NOT NULL | 0 |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `payment_followup_leads_campaign_id_fkey` FOREIGN KEY (campaign_id) REFERENCES ops.agent_campaigns(id) ON DELETE CASCADE
- `payment_followup_leads_contact_id_fkey` FOREIGN KEY (contact_id) REFERENCES ops.agent_campaign_contacts(id) ON DELETE CASCADE
- `payment_followup_leads_pkey` PRIMARY KEY (id)
- `payment_followup_leads_contact_id_key` UNIQUE (contact_id)

Indexes:
- INDEX payment_followup_leads_campaign_idx USING btree (campaign_id)
- UNIQUE INDEX payment_followup_leads_contact_id_key USING btree (contact_id)
- UNIQUE INDEX payment_followup_leads_pkey USING btree (id)
- INDEX payment_followup_leads_status_idx USING btree (status)

### ops.profiles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL |  |
| email | text | NOT NULL |  |
| full_name | text | NOT NULL |  |
| avatar_key | text | null |  |
| phone | text | null |  |
| is_active | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `profiles_pkey` PRIMARY KEY (id)
- `profiles_email_key` UNIQUE (email)

Indexes:
- UNIQUE INDEX profiles_email_key USING btree (email)
- UNIQUE INDEX profiles_pkey USING btree (id)

### ops.user_roles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| role | ops.app_role | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `user_roles_user_id_fkey` FOREIGN KEY (user_id) REFERENCES ops.profiles(id) ON DELETE CASCADE
- `user_roles_pkey` PRIMARY KEY (id)
- `user_roles_user_id_role_key` UNIQUE (user_id, role)

Indexes:
- UNIQUE INDEX user_roles_pkey USING btree (id)
- UNIQUE INDEX user_roles_user_id_role_key USING btree (user_id, role)
- INDEX user_roles_user_idx USING btree (user_id)
