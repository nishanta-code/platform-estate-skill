# Brain project: columns, constraints and indexes for schema `migration`

Live snapshot taken 2026-11-24 from <DATA_REF>. Metadata only.
The migration schema holds the id_map (D-018: every migrated row maps back to its source), run_log, backups taken 2026-09-02 and the Razorpay rules, plus the 18 migration.v_app_* views and migration.sync_* functions that keep public.applications/payments/refunds current from the legacy projects via postgres_fdw.
Legend: one line per table listing `column type` where a trailing `?` means NULLABLE (no `?` = NOT NULL), `=expr` is the column default (generated columns show their expression), and `/*...*/` is the column comment. Below each column line: `PK`/`FK`/`UQ`/`CK`/`EX` constraints (pg_get_constraintdef) and `idx` = indexes not backing a constraint (shown as `name [UNIQUE] USING method (cols) [WHERE ...]`).

## migration

### baseline_20260902
scope text?, provider text?, currency text?, status text?, rows bigint?, total_minor numeric?, first_at timestamp with time zone?, last_at timestamp with time zone?

### id_map
source_system text, source_table text, source_id text, target_schema text='public'::text, target_table text, target_id uuid, wave text, migrated_at timestamp with time zone=now()
- PK id_map_pkey: PRIMARY KEY (source_system, source_table, source_id, target_table)
- idx id_map_source_idx USING btree (source_system, source_table, source_id)
- idx id_map_target_idx USING btree (target_table, target_id)
- idx id_map_wave_idx USING btree (wave, migrated_at DESC)

### payments_backup_20260902
id uuid?, person_id uuid?, cohort_id uuid?, amount_minor bigint?, currency text?, status payment_status?, occurred_at timestamp with time zone?, description text?, source_code text?, needs_review boolean?, external_ids jsonb?, created_at timestamp with time zone?, updated_at timestamp with time zone?, provider text?, provider_payment_id text?

### payments_quarantine_20260902
id uuid?, person_id uuid?, cohort_id uuid?, amount_minor bigint?, currency text?, status payment_status?, occurred_at timestamp with time zone?, description text?, source_code text?, needs_review boolean?, external_ids jsonb?, created_at timestamp with time zone?, updated_at timestamp with time zone?, provider text?, provider_payment_id text?, kept_payment_id uuid?, kept_person_id uuid?, identity_disagreed boolean?, quarantine_reason text?, quarantined_at timestamp with time zone?

### razorpay_date_corrections
payment_id text, stored_local timestamp without time zone, corrected_local timestamp without time zone, evidence text, created_at timestamp with time zone=now()
- PK razorpay_date_corrections_pkey: PRIMARY KEY (payment_id)

### razorpay_programme_rules
app_name text, valid_from date, valid_to date, programme_code text, evidence text, created_at timestamp with time zone=now()
- PK razorpay_programme_rules_pkey: PRIMARY KEY (app_name, valid_from)

### run_log
id uuid=gen_random_uuid(), wave text, started_at timestamp with time zone=now(), finished_at timestamp with time zone?, rows_in integer?, rows_out integer?, run_by text?, notes text?
- CK run_log_check: CHECK (((finished_at IS NULL) OR (finished_at >= started_at)))
- PK run_log_pkey: PRIMARY KEY (id)
- idx run_log_wave_idx USING btree (wave, started_at DESC)
- idx run_log_open_idx USING btree (started_at) WHERE (finished_at IS NULL)

