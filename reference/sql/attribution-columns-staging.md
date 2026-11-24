# Brain project: columns, constraints and indexes for schema `staging`

Live snapshot taken 2026-11-24 from <DATA_REF>. Metadata only.
The staging schema holds one-off reconciliation/migration working tables (49 tables; 37 of them have RLS DISABLED, see schemas-and-tables file). Not exposed to PostgREST roles.
Legend: one line per table listing `column type` where a trailing `?` means NULLABLE (no `?` = NOT NULL), `=expr` is the column default (generated columns show their expression), and `/*...*/` is the column comment. Below each column line: `PK`/`FK`/`UQ`/`CK`/`EX` constraints (pg_get_constraintdef) and `idx` = indexes not backing a constraint (shown as `name [UNIQUE] USING method (cols) [WHERE ...]`).

## staging

### akbar_huso_rzp
payment_id text?, email text?, phone10 text?, amount numeric(12,2)?, created_date timestamp without time zone?, app_no text?

### akbar_offline_sheet
sr_no integer, month_year text?, programme_sheet text?, payment_status text?, student_name text?, student_email text?, student_mobile text?, grade text?, school_name text?, city text?, state text?, country text?, parent_name text?, parent_mobile text?, parent_email text?, counsellor_name text?, counsellor_email text?, student_email_norm text?=NULLIF(lower(TRIM(BOTH FROM student_email)), ''::text), parent_email_norm text?=NULLIF(lower(TRIM(BOTH FROM parent_email)), ''::text), student_phone_digits text?=NULLIF(regexp_replace(COALESCE(student_mobile, ''::text), 'D'::text, ''::text, 'g'::text), ''::text), parent_phone_digits text?=NULLIF(regexp_replace(COALESCE(parent_mobile, ''::text), 'D'::text, ''::text, 'g'::text), ''::text), source_file text='Offline_Program_Mini_Master_3_UPDATED - Copy.xlsx'::text, loaded_at timestamp with time zone=now()
- PK akbar_offline_sheet_pkey: PRIMARY KEY (sr_no)

### akbar_payments_snapshot
payment_id uuid?, email text?, provider text?, status text?, amount_minor bigint?, currency text?, occurred_at timestamp with time zone?, programme_code text?, description text?, refunded_minor bigint?, snapshot_at timestamp with time zone?
- idx akbar_payments_snapshot_email_idx USING btree (email)

### akbar_prog_code
p text?, dominant_code text?, n bigint?, coded_total numeric?, programme_code text?, code_source text?

### akbar_prog_evidence
p text?, d text?, rows_p_d bigint?, rows_p_any bigint?, pays_p_d bigint?, pays_d_any_sheet bigint?

### akbar_prog_map
p text?, d text?, rows_p_d bigint?, share_pct numeric?, exclusive_pct numeric?, verdict text?, note text?, valid_from date?, valid_to date?

### akbar_sheet_emails
sr_no integer?, email text?, who text?
- idx akbar_sheet_emails_email_idx USING btree (email)

### akbar_verification
sr_no integer, sheet_status text, db_status text, verdict text, matched_via text?, confidence text?, payment_ids uuid[], money jsonb, first_paid_at timestamp with time zone?, last_paid_at timestamp with time zone?, verified_at timestamp with time zone=now()
- FK akbar_verification_sr_no_fkey: FOREIGN KEY (sr_no) REFERENCES staging.akbar_offline_sheet(sr_no)
- PK akbar_verification_pkey: PRIMARY KEY (sr_no)

### akbar_verification_v1
sr_no integer?, sheet_status text?, db_status text?, verdict text?, matched_via text?, confidence text?, payment_ids uuid[]?, money jsonb?, first_paid_at timestamp with time zone?, last_paid_at timestamp with time zone?, verified_at timestamp with time zone?

### app_household_of
parent_id uuid?, household_id uuid?

### app_pairs
student_id uuid?, parent_id uuid?

### applications
source_table text, source_id text, submitted_at timestamp with time zone?, student_name text?, student_email text?, student_dob date?, grade text?, school text?, city text?, country text?, parent_name text?, parent_email text?, counsellor_name text?, counsellor_email text?, cohort text?, payment_status text?
- PK applications_pkey: PRIMARY KEY (source_table, source_id)

### applications_removed_20261003
id uuid?, source_table text?, source_row_id uuid?, programme_code text?, cohort_raw text?, student_name text?, student_email text?, student_phone text?, student_grade text?, parent_name text?, parent_email text?, parent_phone text?, school_raw text?, school_code text?, city text?, state text?, country text?, applied_at timestamp with time zone?, paid_at timestamp with time zone?, payment_status_raw text?, provider_payment_id text?, payment_provider_hint text?, utm_source text?, utm_medium text?, utm_campaign text?, referral_source text?, person_id uuid?, raw jsonb?, needs_review boolean?, synced_at timestamp with time zone?, why text?, removed_at timestamp with time zone?

### code_lead
code text?, edition_date date?, n bigint?, p95_lead double precision?, p02_lead double precision?

### derived_codes
id uuid?, occurred_at timestamp with time zone?, currency text?, net bigint?, st text?, family text?, code text?, edition_date date?, lead_days integer?, max_lead_days integer?, min_lead_days integer?, accepted boolean?

### desc_code_evidence
id uuid?, d text?, occurred_at timestamp with time zone?, currency text?, net bigint?, st text?, codes_for_desc bigint?, codes_in_window bigint?, code_in_window text?

### desc_family
d text?, families text[]?

### edge_claims
e text?, role text?, nm text?, school_raw text?, grade_num integer?, country text?, city text?, lead_type text?, created_at timestamp with time zone?

### edge_household_of
parent_id uuid?, household_id uuid?

### edge_pairs
parent_id uuid?, student_id uuid?

### edge_people
e text?, roles_seen text[]?, nm text?, school_raw text?, country text?, city text?, grade_num integer?, first_seen timestamp with time zone?, ever_submitted integer?, ambiguous boolean?, role_resolved text?, is_minor boolean?

### edge_rows
lead_id uuid?, pe text?, se text?, pn text?, sn text?, school_raw text?, grade_num integer?, country text?, city text?, lead_type text?, created_at timestamp with time zone?

### edition_candidates
code text?, family text?, edition_date date?

### email_suppression_raw
id text?, email text?, reason text?, meeting_id text?, created_at text?

### family_backtest
family text?, coded_n bigint?, correct bigint?, wrong bigint?

### family_lead
family text?, max_lead_days integer?, min_lead_days integer?

### household_of
parent_id uuid?, household_id uuid?

### household_pairs
student_id uuid?, parent_id uuid?

### huso_household_of
parent_id uuid?, household_id uuid?

### huso_norm
source_table text?, source_id text?, application_number text?, student_name text?, se text?, se_raw text?, parent_name text?, pe text?, pe_raw text?, counsellor_name text?, ce text?, ce_raw text?, country text?, school_name text?, class_raw text?, grade_parsed integer?, status text?, order_id text?, payment_id text?, currency text?, amount numeric(12,2)?, scholarship_code text?, payment_date date?, registered_date date?, created_at timestamp with time zone?
- idx huso_norm_se_idx USING btree (se)
- idx huso_norm_pe_idx USING btree (pe)
- idx huso_norm_ce_idx USING btree (ce)
- idx huso_norm_payment_id_idx USING btree (payment_id)

### huso_pairs
student_id uuid?, parent_id uuid?

### huso_people
e text?, roles_seen text?, role_count bigint?, role_resolved text?, is_minor boolean?, raw text?, nm text?, country text?, source_table text?, created_at timestamp with time zone?, application_number text?, grade_parsed integer?, class_raw text?, scholarship_code text?
- idx huso_people_e_idx USING btree (e)

### inbound_people
source_table text, source_id text, submitted_at timestamp with time zone?, full_name text?, email text?, country text?, who text?, grade text?, school_name text?, parent_name text?, parent_email text?, consent text?
- PK inbound_people_pkey: PRIMARY KEY (source_table, source_id)

### link_derived
id uuid?, code text?, occurred_at timestamp with time zone?, net bigint?, currency text?, edition_date date?, lead integer?, p95_lead double precision?, p02_lead double precision?, codes_on_link bigint?

### payer_household_of
parent_id uuid?, household_id uuid?

### payer_pairs
student_id uuid?, parent_id uuid?

### payer_people
e text?, roles_seen text?, role_count bigint?, ambiguous boolean?, role_resolved text?, is_minor boolean?, nm text?, school_name text?, occurred_at timestamp with time zone?
- idx payer_people_e_idx USING btree (e)

### payments_raw
source_id text?, payment_intent_id text?, payer_email text?, student_email text?, amount numeric(12,2)?, currency text?, status_raw text?, program_code text?, description text?, payout_id text?, created_date_utc timestamp without time zone?, status_parsed text?, refund_amount numeric?, refund_currency text?
- idx payments_raw_payment_intent_id_idx USING btree (payment_intent_id)
- idx payments_raw_payer_email_idx USING btree (payer_email)

### phone_claims
person_id uuid?, is_minor boolean?, raw_phone text?, digits text?, country text?, already_international boolean?

### phone_owner
digits text?, rule text?, owner_id uuid?, country text?, already_international boolean?, raw_phone text?

### plink_code_windows
plink text?, code text?, n bigint?, lo timestamp with time zone?, hi timestamp with time zone?

### plink_codes
plink text?, codes text[]?, coded_n bigint?, lo timestamp with time zone?, hi timestamp with time zone?

### plink_evidence
id uuid?, currency text?, net bigint?, occurred_at timestamp with time zone?, plink text?, codes_on_link bigint?, codes_in_window bigint?, code_in_window text?

### prospects
key text?, source_id text?, e text?, email_raw text?, li text?, first_name text?, last_name text?, title text?, company_name text?, country text?, phone_raw text?, industry text?, website text?, email_source text?, bucket text?, status_raw text?, created_at timestamp with time zone?
- idx prospects_key_idx USING btree (key)
- idx prospects_e_idx USING btree (e)

### prospects_raw
source_id text?, e text?, email_raw text?, li text?, first_name text?, last_name text?, title text?, company_name text?, country text?, phone_raw text?, industry text?, website text?, email_source text?, bucket text?, status_raw text?, created_at timestamp with time zone?
- idx prospects_raw_e_idx USING btree (e)
- idx prospects_raw_li_idx USING btree (li)

### school_aliases_raw
id text?, raw_text text?, school_code text?, match_confidence text?, match_source text?, times_seen text?

### school_registry_raw
id text?, code text?, canonical_name text?, country text?, city text?, region text?, is_active text?, auto_created text?

### uncoded
id uuid?, provider text?, provider_payment_id text?, d text?, currency text?, net bigint?, occurred_at timestamp with time zone?

### w5b_linkedin_suppressions_hashes
source_id text, kind identity_kind, identity_hash bytea

