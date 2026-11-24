# Shared estate columns: affiliate (10 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### affiliate.admins

| column | type | nullable | default |
|---|---|---|---|
| email | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `admins_email_check` CHECK ((email = lower(email)))
- `admins_pkey` PRIMARY KEY (email)

Indexes:
- UNIQUE INDEX admins_pkey USING btree (email)

### affiliate.announcements

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| kind | text | NOT NULL |  |
| title | text | NOT NULL |  |
| link_url | text | NOT NULL |  |
| created_by_email | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `announcements_kind_check` CHECK ((kind = ANY (ARRAY['brochure'::text, 'form'::text, 'update'::text])))
- `announcements_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX announcements_pkey USING btree (id)

### affiliate.partners

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| auth_user_id | uuid | NOT NULL |  |
| full_name | text | NOT NULL |  |
| email | text | NOT NULL |  |
| channel_handle | text | null |  |
| channel_type | text | null |  |
| audience_size | text | null |  |
| status | text | NOT NULL | 'approved'::text |
| rate_camps | numeric | NOT NULL | 1 |
| rate_inperson | numeric | NOT NULL | 1.5 |
| rate_online | numeric | NOT NULL | 2 |
| discount_cap_percent | numeric | NOT NULL | 10 |
| payout_method | text | null |  |
| payout_account_last4 | text | null |  |
| payout_currency | text | NOT NULL | 'USD'::text |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `partners_status_check` CHECK ((status = ANY (ARRAY['pending'::text, 'approved'::text, 'rejected'::text, 'suspended'::text])))
- `partners_pkey` PRIMARY KEY (id)
- `partners_auth_user_id_key` UNIQUE (auth_user_id)
- `partners_email_key` UNIQUE (email)

Indexes:
- UNIQUE INDEX partners_auth_user_id_key USING btree (auth_user_id)
- UNIQUE INDEX partners_email_key USING btree (email)
- UNIQUE INDEX partners_pkey USING btree (id)

### affiliate.payment_link_links

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| partner_id | uuid | NOT NULL |  |
| central_payment_link_id | uuid | NOT NULL |  |
| slug | text | NOT NULL |  |
| program_pay_slug | text | null |  |
| program_name | text | null |  |
| list_price | numeric | null |  |
| offer_price | numeric | null |  |
| currency | text | null |  |
| rate_percent | numeric | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `payment_link_links_partner_id_fkey` FOREIGN KEY (partner_id) REFERENCES affiliate.partners(id) ON DELETE CASCADE
- `payment_link_links_pkey` PRIMARY KEY (id)
- `payment_link_links_central_payment_link_id_key` UNIQUE (central_payment_link_id)

Indexes:
- UNIQUE INDEX payment_link_links_central_payment_link_id_key USING btree (central_payment_link_id)
- INDEX payment_link_links_partner_id_idx USING btree (partner_id)
- UNIQUE INDEX payment_link_links_pkey USING btree (id)

### affiliate.payouts

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| partner_id | uuid | NOT NULL |  |
| amount | numeric(12,2) | NOT NULL |  |
| currency | text | NOT NULL |  |
| status | text | NOT NULL | 'processing'::text |
| period_label | text | null |  |
| reference | text | null |  |
| paid_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `payouts_amount_check` CHECK ((amount > (0)::numeric))
- `payouts_status_check` CHECK ((status = ANY (ARRAY['processing'::text, 'paid'::text, 'cancelled'::text])))
- `payouts_partner_id_fkey` FOREIGN KEY (partner_id) REFERENCES affiliate.partners(id) ON DELETE RESTRICT
- `payouts_pkey` PRIMARY KEY (id)

Indexes:
- INDEX payouts_partner_idx USING btree (partner_id)
- UNIQUE INDEX payouts_pkey USING btree (id)

### affiliate.promo_code_links

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| partner_id | uuid | NOT NULL |  |
| central_promo_code_id | uuid | NOT NULL |  |
| code | text | NOT NULL |  |
| program_pay_slug | text | null |  |
| program_name | text | null |  |
| list_price | numeric | null |  |
| currency | text | null |  |
| rate_percent | numeric | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `promo_code_links_partner_id_fkey` FOREIGN KEY (partner_id) REFERENCES affiliate.partners(id) ON DELETE CASCADE
- `promo_code_links_pkey` PRIMARY KEY (id)
- `promo_code_links_central_promo_code_id_key` UNIQUE (central_promo_code_id)

Indexes:
- UNIQUE INDEX promo_code_links_central_promo_code_id_key USING btree (central_promo_code_id)
- INDEX promo_code_links_partner_id_idx USING btree (partner_id)
- UNIQUE INDEX promo_code_links_pkey USING btree (id)

### affiliate.registration_attempts

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| ip_hash | text | NOT NULL |  |
| email_hash | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `registration_attempts_pkey` PRIMARY KEY (id)

Indexes:
- INDEX registration_attempts_email_idx USING btree (email_hash, created_at)
- INDEX registration_attempts_ip_idx USING btree (ip_hash, created_at)
- UNIQUE INDEX registration_attempts_pkey USING btree (id)

### affiliate.resources

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| title | text | NOT NULL |  |
| type | text | NOT NULL |  |
| url | text | NOT NULL |  |
| description | text | null |  |
| featured | boolean | NOT NULL | false |
| created_by_email | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `resources_type_check` CHECK ((type = ANY (ARRAY['Image'::text, 'Video'::text, 'Sheet'::text, 'PDF'::text, 'Doc'::text])))
- `resources_pkey` PRIMARY KEY (id)

Indexes:
- INDEX resources_featured_idx USING btree (featured, created_at DESC)
- UNIQUE INDEX resources_pkey USING btree (id)

### affiliate.school_achievements

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| school_id | uuid | NOT NULL |  |
| kind | text | NOT NULL |  |
| title | text | NOT NULL |  |
| description | text | null |  |
| student_name | text | null |  |
| program_name | text | null |  |
| link_url | text | null |  |
| cover_image_url | text | null |  |
| featured | boolean | NOT NULL | false |
| published_at | date | NOT NULL | CURRENT_DATE |
| created_by_email | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `school_achievements_kind_check` CHECK ((kind = ANY (ARRAY['paper'::text, 'certificate'::text, 'testimonial'::text, 'project'::text])))
- `school_achievements_school_id_fkey` FOREIGN KEY (school_id) REFERENCES affiliate.schools(id) ON DELETE CASCADE
- `school_achievements_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX school_achievements_pkey USING btree (id)
- INDEX school_achievements_school_idx USING btree (school_id, kind)

### affiliate.schools

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| auth_user_id | uuid | null |  |
| school_name | text | NOT NULL |  |
| contact_name | text | NOT NULL |  |
| email | text | NOT NULL |  |
| school_code | text | null |  |
| status | text | NOT NULL | 'invited'::text |
| invited_by_email | text | NOT NULL |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `schools_email_check` CHECK ((email = lower(email)))
- `schools_status_check` CHECK ((status = ANY (ARRAY['invited'::text, 'active'::text, 'suspended'::text])))
- `schools_pkey` PRIMARY KEY (id)
- `schools_auth_user_id_key` UNIQUE (auth_user_id)
- `schools_email_key` UNIQUE (email)

Indexes:
- UNIQUE INDEX schools_auth_user_id_key USING btree (auth_user_id)
- UNIQUE INDEX schools_email_key USING btree (email)
- UNIQUE INDEX schools_pkey USING btree (id)
