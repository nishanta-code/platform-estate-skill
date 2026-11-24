# Shared estate columns: finance (27 tables)

**Live snapshot taken 2026-11-24 from <ESTATE_REF>** ("the Group - Website", shared Group Supabase estate, PostgreSQL 17.6). Metadata only; no business rows were read. Secret-looking literals and e-mail local parts are redacted.

Per table: columns (`column | type | nullable | default`, ordinal order, from pg_attribute/pg_attrdef), then constraints from `pg_constraint` (PRIMARY KEY / FOREIGN KEY / UNIQUE / CHECK via `pg_get_constraintdef`), then indexes from `pg_indexes` (`CREATE ` and `ON <table>` stripped). Unqualified table names in FOREIGN KEY ... REFERENCES mean `public`.

### finance.app_settings

| column | type | nullable | default |
|---|---|---|---|
| id | text | NOT NULL | 'global'::text |
| currency | text | NOT NULL | 'SGD'::text |
| mode | text | NOT NULL | 'program'::text |
| selected_month | text | NOT NULL | 'all'::text |
| financial_year_id | text | null |  |
| updated_at | timestamp with time zone | NOT NULL | now() |
| updated_by | uuid | null |  |
| flag_absolute_threshold_usd | numeric | NOT NULL | 5000 |
| flag_nature_thresholds_usd | jsonb | NOT NULL | '{}'::jsonb |
| flag_outlier_multiplier | numeric | NOT NULL | 1.5 |

Constraints:
- `app_settings_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX app_settings_pkey USING btree (id)

### finance.attachments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| entity_type | attachment_entity_type | NOT NULL |  |
| entity_id | text | NOT NULL |  |
| kind | attachment_kind | NOT NULL |  |
| title | text | null |  |
| note_text | text | null |  |
| link_url | text | null |  |
| storage_path | text | null |  |
| mime_type | text | null |  |
| size_bytes | bigint | null |  |
| duration_ms | integer | null |  |
| transcript | text | null |  |
| uploaded_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `attachments_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX attachments_pkey USING btree (id)

### finance.code_sheet_sync_state

| column | type | nullable | default |
|---|---|---|---|
| id | boolean | NOT NULL | true |
| callback_url | text | null |  |
| callback_secret | text | null |  |
| dirty | boolean | NOT NULL | true |
| claim | uuid | null |  |
| claimed_at | timestamp with time zone | null |  |
| last_synced_at | timestamp with time zone | null |  |
| last_error | text | null |  |

Constraints:
- `code_sheet_sync_state_id_check` CHECK (id)
- `code_sheet_sync_state_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX code_sheet_sync_state_pkey USING btree (id)

### finance.codes_dropdown

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| ref_type | text | NOT NULL |  |
| code | text | NOT NULL |  |
| name | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `codes_dropdown_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX codes_dropdown_pkey USING btree (id)
- UNIQUE INDEX codes_dropdown_reftype_code_key USING btree (ref_type, upper(code))
- UNIQUE INDEX codes_dropdown_type_code_key USING btree (ref_type, code)

### finance.dbs_sync_logs

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| started_at | timestamp with time zone | NOT NULL | now() |
| finished_at | timestamp with time zone | null |  |
| rows_received | integer | NOT NULL | 0 |
| rows_parsed | integer | NOT NULL | 0 |
| rows_upserted | integer | NOT NULL | 0 |
| rows_deleted | integer | NOT NULL | 0 |
| rows_needs_review | integer | NOT NULL | 0 |
| error_message | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `dbs_sync_logs_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX dbs_sync_logs_pkey USING btree (id)

### finance.dbs_transactions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| uetr_ref | text | NOT NULL |  |
| initiated_date | date | null |  |
| completed_date | date | null |  |
| description_raw | text | null |  |
| bank_ref | text | null |  |
| transaction_id | text | null |  |
| sender_name | text | null |  |
| currency | text | null |  |
| amount | numeric(18,2) | null |  |
| amount_raw | text | null |  |
| program_code | text | null |  |
| program_name | text | null |  |
| student_names | text[] | NOT NULL | '{}'::text[] |
| batch_start_date | text | null |  |
| synced_at | timestamp with time zone | NOT NULL | now() |
| created_at | timestamp with time zone | NOT NULL | now() |
| transaction_type | text | null |  |
| narrative | text | null |  |
| stable_row_id | text | null |  |

Constraints:
- `dbs_transactions_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX dbs_transactions_pkey USING btree (id)
- UNIQUE INDEX dbs_transactions_user_uetr_unique USING btree (user_id, uetr_ref)
- INDEX idx_dbs_transactions_user_stable_row_id USING btree (user_id, stable_row_id) WHERE (stable_row_id IS NOT NULL)

### finance.expense_codes

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| code | text | NOT NULL |  |
| expense_type | text | NOT NULL |  |
| person_name | text | NOT NULL |  |
| purpose | text | null |  |
| description | text | null |  |
| period | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `expense_codes_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX expense_codes_code_key USING btree (upper(code))
- UNIQUE INDEX expense_codes_pkey USING btree (id)

### finance.fx_rates

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| base_currency | text | NOT NULL |  |
| quote_currency | text | NOT NULL |  |
| rate | numeric(18,8) | NOT NULL |  |
| source | text | NOT NULL | 'ai-daily'::text |
| as_of_date | date | NOT NULL | ((now() AT TIME ZONE 'UTC'::text))::date |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `fx_rates_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX fx_rates_pkey USING btree (id)
- UNIQUE INDEX fx_rates_unique_pair_per_day USING btree (base_currency, quote_currency, as_of_date)

### finance.overhead_allocations

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| overhead_entry_id | uuid | NOT NULL |  |
| program_code | text | NOT NULL |  |
| period_month | date | NOT NULL |  |
| weight_pct | numeric | NOT NULL | 0 |
| amount_allocated_usd | numeric | NOT NULL | 0 |
| source | text | NOT NULL | 'manual'::text |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `overhead_alloc_source_chk` CHECK ((source = ANY (ARRAY['manual'::text, 'revenue_share'::text, 'suggested'::text])))
- `overhead_allocations_overhead_entry_id_fkey` FOREIGN KEY (overhead_entry_id) REFERENCES finance.overhead_entries(id) ON DELETE CASCADE
- `overhead_allocations_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX overhead_allocations_overhead_entry_id_program_code_period__key USING btree (overhead_entry_id, program_code, period_month)
- UNIQUE INDEX overhead_allocations_pkey USING btree (id)

### finance.overhead_categories

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| slug | text | NOT NULL |  |
| color | text | null |  |
| sort_order | integer | NOT NULL | 100 |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `overhead_categories_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX overhead_categories_name_key USING btree (name)
- UNIQUE INDEX overhead_categories_pkey USING btree (id)
- UNIQUE INDEX overhead_categories_slug_key USING btree (slug)

### finance.overhead_entries

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| category_id | uuid | NOT NULL |  |
| label | text | NOT NULL |  |
| employee_name | text | null |  |
| vendor_name | text | null |  |
| amount | numeric | NOT NULL | 0 |
| currency | text | NOT NULL | 'USD'::text |
| amount_usd | numeric | NOT NULL | 0 |
| period_start | date | NOT NULL |  |
| period_end | date | null |  |
| recurrence | text | NOT NULL | 'one_off'::text |
| notes | text | null |  |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `overhead_recurrence_chk` CHECK ((recurrence = ANY (ARRAY['one_off'::text, 'monthly'::text, 'annual'::text])))
- `overhead_entries_category_id_fkey` FOREIGN KEY (category_id) REFERENCES finance.overhead_categories(id) ON DELETE RESTRICT
- `overhead_entries_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX overhead_entries_pkey USING btree (id)

### finance.payment_matches

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| dbs_transaction_id | uuid | null |  |
| xero_invoice_id | text | null |  |
| xero_payment_id | text | null |  |
| amount_applied | numeric | NOT NULL |  |
| currency | text | null |  |
| confidence | numeric | null |  |
| match_source | text | null |  |
| matched_rule | text | null |  |
| matched_by | uuid | null |  |
| matched_at | timestamp with time zone | NOT NULL | now() |
| notes | text | null |  |

Constraints:
- `payment_matches_dbs_transaction_id_fkey` FOREIGN KEY (dbs_transaction_id) REFERENCES finance.dbs_transactions(id) ON DELETE CASCADE
- `payment_matches_xero_invoice_id_fkey` FOREIGN KEY (xero_invoice_id) REFERENCES finance.xero_invoices(invoice_id) ON DELETE CASCADE
- `payment_matches_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX payment_matches_pkey USING btree (id)

### finance.program_budgets

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_code | text | NOT NULL |  |
| nature_of_expense | text | null |  |
| amount_usd | numeric | NOT NULL |  |
| source | text | NOT NULL | 'manual'::text |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_budgets_source_check` CHECK ((source = ANY (ARRAY['manual'::text, 'peer_avg'::text])))
- `program_budgets_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX program_budgets_pkey USING btree (id)
- UNIQUE INDEX program_budgets_program_code_nature_of_expense_key USING btree (program_code, nature_of_expense)

### finance.program_codes

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_code | text | NOT NULL |  |
| category | text | null |  |
| organisation | text | null |  |
| program | text | null |  |
| school | text | null |  |
| country | text | null |  |
| batch | text | null |  |
| date | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| status | text | NOT NULL | 'active'::text |

Constraints:
- `program_codes_status_check` CHECK ((status = ANY (ARRAY['active'::text, 'cancelled'::text])))
- `program_codes_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX program_codes_pkey USING btree (id)
- UNIQUE INDEX program_codes_program_code_key USING btree (upper(program_code))

### finance.program_completions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_code | text | NOT NULL |  |
| status | text | NOT NULL | 'suggested'::text |
| completed_at | timestamp with time zone | null |  |
| completed_by | uuid | null |  |
| notes | text | null |  |
| snapshot | jsonb | null |  |
| ai_report | text | null |  |
| ai_generated_at | timestamp with time zone | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_completions_status_check` CHECK ((status = ANY (ARRAY['suggested'::text, 'completed'::text])))
- `program_completions_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX program_completions_pkey USING btree (id)
- UNIQUE INDEX program_completions_program_code_key USING btree (program_code)

### finance.program_marketing_metrics

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_code | text | NOT NULL |  |
| period_month | date | NOT NULL |  |
| leads | integer | null |  |
| enquiries | integer | null |  |
| enrolments | integer | null |  |
| marketing_spend_usd | numeric | null |  |
| notes | text | null |  |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_marketing_metrics_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX program_marketing_metrics_pkey USING btree (id)
- UNIQUE INDEX program_marketing_metrics_program_code_period_month_key USING btree (program_code, period_month)

### finance.program_mentor_assignments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| program_code | text | NOT NULL |  |
| mentor_id | uuid | NOT NULL |  |
| role | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `program_mentor_assignments_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX program_mentor_assignments_pkey USING btree (id)
- UNIQUE INDEX program_mentor_assignments_program_code_mentor_id_key USING btree (program_code, mentor_id)

### finance.travel_expenses

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| traveller_name | text | NOT NULL |  |
| mentor_id | uuid | null |  |
| route | text | null |  |
| travel_type | text | NOT NULL | 'flight'::text |
| travel_start_date | date | null |  |
| travel_end_date | date | null |  |
| program_code | text | null |  |
| program_start_date | date | null |  |
| purpose_notes | text | null |  |
| cost | numeric | NOT NULL | 0 |
| currency | text | NOT NULL | 'USD'::text |
| cost_usd | numeric | NOT NULL | 0 |
| source_wallex_id | uuid | null |  |
| source_vendor_bill_id | uuid | null |  |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `travel_expenses_source_vendor_bill_id_fkey` FOREIGN KEY (source_vendor_bill_id) REFERENCES finance.vendor_bills(id) ON DELETE SET NULL
- `travel_expenses_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX travel_expenses_pkey USING btree (id)

### finance.user_profiles

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| display_name | text | null |  |
| avatar_url | text | null |  |
| role | finance.app_role | NOT NULL | 'viewer'::finance.app_role |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `user_profiles_pkey` PRIMARY KEY (id)
- `user_profiles_user_id_key` UNIQUE (user_id)

Indexes:
- UNIQUE INDEX user_profiles_pkey USING btree (id)
- UNIQUE INDEX user_profiles_user_id_key USING btree (user_id)

### finance.vendor_bill_expense_links

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| bill_id | uuid | NOT NULL |  |
| wallex_stable_row_id | text | NOT NULL |  |
| created_by | uuid | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `vendor_bill_expense_links_bill_id_fkey` FOREIGN KEY (bill_id) REFERENCES finance.vendor_bills(id) ON DELETE CASCADE
- `vendor_bill_expense_links_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX vendor_bill_expense_links_bill_id_wallex_stable_row_id_key USING btree (bill_id, wallex_stable_row_id)
- UNIQUE INDEX vendor_bill_expense_links_pkey USING btree (id)

### finance.vendor_bill_lines

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| bill_id | uuid | NOT NULL |  |
| description | text | null |  |
| kind | text | null |  |
| quantity | numeric | null |  |
| unit_price | numeric | null |  |
| amount | numeric | null |  |
| currency | text | null |  |
| passenger_name | text | null |  |
| mentor_id | uuid | null |  |
| travel_date | date | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `vendor_bill_lines_kind_check` CHECK ((kind = ANY (ARRAY['flight'::text, 'hotel'::text, 'meal'::text, 'other'::text, 'materials'::text, 'ground_transport'::text])))
- `vendor_bill_lines_bill_id_fkey` FOREIGN KEY (bill_id) REFERENCES finance.vendor_bills(id) ON DELETE CASCADE
- `vendor_bill_lines_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX vendor_bill_lines_pkey USING btree (id)

### finance.vendor_bills

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| vendor_id | uuid | null |  |
| uploaded_by | uuid | null |  |
| file_path | text | NOT NULL |  |
| file_name | text | null |  |
| mime_type | text | null |  |
| status | text | NOT NULL | 'pending'::text |
| program_code | text | null |  |
| invoice_no | text | null |  |
| invoice_date | date | null |  |
| currency | text | null |  |
| amount | numeric | null |  |
| amount_usd | numeric | null |  |
| ai_extracted | jsonb | null |  |
| ai_confidence | numeric | null |  |
| ai_generated_at | timestamp with time zone | null |  |
| flags | jsonb | NOT NULL | '{}'::jsonb |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `vendor_bills_status_check` CHECK ((status = ANY (ARRAY['pending'::text, 'confirmed'::text, 'rejected'::text])))
- `vendor_bills_vendor_id_fkey` FOREIGN KEY (vendor_id) REFERENCES finance.vendors(id) ON DELETE SET NULL
- `vendor_bills_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX vendor_bills_pkey USING btree (id)

### finance.vendors

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| name | text | NOT NULL |  |
| contact_email | text | null |  |
| contact_phone | text | null |  |
| category | text | null |  |
| notes | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `vendors_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX vendors_pkey USING btree (id)

### finance.wallex_transactions

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| user_id | uuid | NOT NULL |  |
| stable_row_id | text | NOT NULL |  |
| sr_no | text | null |  |
| creation_date | text | null |  |
| completed_date | text | null |  |
| sm_conversion_reference | text | null |  |
| beneficiary_name | text | null |  |
| bank_name | text | null |  |
| account_number_iban | text | null |  |
| bic_swift_code | text | null |  |
| nature_of_expense | text | null |  |
| program_code | text | null |  |
| payment_channel | text | null |  |
| transaction_type | text | null |  |
| currency_buy | text | null |  |
| amount_buy | text | null |  |
| currency_sell | text | null |  |
| amount_sell | text | null |  |
| rate | text | null |  |
| wallex_fees | text | null |  |
| bank_charges | text | null |  |
| total_payable | text | null |  |
| status | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| synced_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `wallex_transactions_pkey` PRIMARY KEY (id)
- `wallex_transactions_stable_row_id_key` UNIQUE (stable_row_id)

Indexes:
- INDEX idx_wallex_creation USING btree (creation_date)
- INDEX idx_wallex_program_code USING btree (program_code)
- INDEX idx_wallex_status USING btree (status)
- INDEX idx_wallex_user_id USING btree (user_id)
- UNIQUE INDEX wallex_transactions_pkey USING btree (id)
- UNIQUE INDEX wallex_transactions_stable_row_id_key USING btree (stable_row_id)

### finance.xero_invoice_lines

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| invoice_id | text | NOT NULL |  |
| line_item_id | text | null |  |
| description | text | null |  |
| quantity | numeric | null |  |
| unit_amount | numeric | null |  |
| line_amount | numeric | null |  |
| tracking | jsonb | null |  |
| inferred_student_name | text | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `xero_invoice_lines_invoice_id_fkey` FOREIGN KEY (invoice_id) REFERENCES finance.xero_invoices(invoice_id) ON DELETE CASCADE
- `xero_invoice_lines_pkey` PRIMARY KEY (id)

Indexes:
- UNIQUE INDEX xero_invoice_lines_pkey USING btree (id)

### finance.xero_invoices

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| invoice_id | text | NOT NULL |  |
| invoice_number | text | null |  |
| contact_id | text | null |  |
| contact_name | text | null |  |
| program_code | text | null |  |
| program_code_source | text | null |  |
| issue_date | date | null |  |
| due_date | date | null |  |
| currency | text | null |  |
| subtotal | numeric | null |  |
| total | numeric | null |  |
| amount_paid | numeric | null | 0 |
| amount_due | numeric | null | 0 |
| status | text | null |  |
| reference | text | null |  |
| raw | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |
| updated_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `xero_invoices_pkey` PRIMARY KEY (id)
- `xero_invoices_invoice_id_key` UNIQUE (invoice_id)

Indexes:
- UNIQUE INDEX xero_invoices_invoice_id_key USING btree (invoice_id)
- UNIQUE INDEX xero_invoices_pkey USING btree (id)

### finance.xero_payments

| column | type | nullable | default |
|---|---|---|---|
| id | uuid | NOT NULL | gen_random_uuid() |
| payment_id | text | NOT NULL |  |
| invoice_id | text | null |  |
| payment_date | date | null |  |
| amount | numeric | null |  |
| currency | text | null |  |
| reference | text | null |  |
| bank_account | text | null |  |
| raw | jsonb | null |  |
| created_at | timestamp with time zone | NOT NULL | now() |

Constraints:
- `xero_payments_invoice_id_fkey` FOREIGN KEY (invoice_id) REFERENCES finance.xero_invoices(invoice_id) ON DELETE SET NULL
- `xero_payments_pkey` PRIMARY KEY (id)
- `xero_payments_payment_id_key` UNIQUE (payment_id)

Indexes:
- UNIQUE INDEX xero_payments_payment_id_key USING btree (payment_id)
- UNIQUE INDEX xero_payments_pkey USING btree (id)
