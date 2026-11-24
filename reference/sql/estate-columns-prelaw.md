# Shared estate columns: prelaw (7 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### prelaw.email_logs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| template_key | text | null |  |
| to_email | text | NOT NULL |  |
| subject | text | null |  |
| status | text | NOT NULL |  |
| error | text | null |  |
| provider_response | jsonb | null |  |
| idempotency_key | text | null |  |
| sent_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `email_logs_pkey` PRIMARY KEY (id)
- `email_logs_idempotency_key_key` UNIQUE (idempotency_key)

Indexes:
- UNIQUE INDEX email_logs_idempotency_key_key USING btree (idempotency_key)
- UNIQUE INDEX email_logs_pkey USING btree (id)

### prelaw.email_templates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| key | text | NOT NULL |  |
| name | text | NOT NULL |  |
| subject | text | NOT NULL | ''::text |
| html_body | text | NOT NULL | ''::text |
| text_body | text | NOT NULL | ''::text |
| variables | jsonb | NOT NULL | '[]'::jsonb |
| enabled | boolean | NOT NULL | true |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `email_templates_pkey` PRIMARY KEY (id)
- `email_templates_key_key` UNIQUE (key)

Indexes:
- UNIQUE INDEX email_templates_key_key USING btree (key)
- UNIQUE INDEX email_templates_pkey USING btree (id)

### prelaw.promo_slot_claims

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

### prelaw.unmatched_payments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| stripe_session_id | text | NOT NULL |  |
| payment_intent_id | text | null |  |
| payment_link_id | text | null |  |
| email | text | NOT NULL |  |
| customer_name | text | null |  |
| amount | numeric | null |  |
| currency | text | null |  |
| cohort | text | null |  |
| paid_at | timestamp with time zone | NOT NULL | now() |
| status | text | NOT NULL | 'pending_application'::text |
| matched_application_id | uuid | null |  |
| matched_at | timestamp with time zone | null |  |
| reminder_count | integer | NOT NULL | 0 |
| last_reminder_at | timestamp with time zone | null |  |
| raw | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `unmatched_payments_pkey` PRIMARY KEY (id)
- `unmatched_payments_stripe_session_id_key` UNIQUE (stripe_session_id)

Indexes:
- INDEX unmatched_payments_email_idx USING btree (lower(email))
- UNIQUE INDEX unmatched_payments_pkey USING btree (id)
- INDEX unmatched_payments_status_idx USING btree (status)
- UNIQUE INDEX unmatched_payments_stripe_session_id_key USING btree (stripe_session_id)

### prelaw.user_roles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| role | prelaw.app_role | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `user_roles_pkey` PRIMARY KEY (id)
- `user_roles_user_id_role_key` UNIQUE (user_id, role)

Indexes:
- UNIQUE INDEX user_roles_pkey USING btree (id)
- UNIQUE INDEX user_roles_user_id_role_key USING btree (user_id, role)

### prelaw.whatsapp_messages

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

### prelaw.whatsapp_settings

| column | type | nullable | default |
|---|---|---|---|
| key | text | NOT NULL |  |
| value | jsonb | NOT NULL | '{}'::jsonb |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `whatsapp_settings_pkey` PRIMARY KEY (key)

Indexes:
- UNIQUE INDEX whatsapp_settings_pkey USING btree (key)
