# Brain project: columns, constraints and indexes for schema `public`

Live snapshot taken 2026-11-24 from <DATA_REF>. Metadata only.
The public schema holds the canonical spine (people, organizations, households, payments, refunds, applications, staff, source_codes) plus ~150 research_* tables. **public.activities is partitioned by RANGE (occurred_at) by month**: partitions activities_2026_08, activities_2026_09, activities_2026_10 and activities_default (bounds shown on each partition heading). Tables are listed in two parts: non-research tables first, then research_* tables.
Legend: one line per table listing `column type` where a trailing `?` means NULLABLE (no `?` = NOT NULL), `=expr` is the column default (generated columns show their expression), and `/*...*/` is the column comment. Below each column line: `PK`/`FK`/`UQ`/`CK`/`EX` constraints (pg_get_constraintdef) and `idx` = indexes not backing a constraint (shown as `name [UNIQUE] USING method (cols) [WHERE ...]`).

## Part 1: public non-research tables

### activities (partitioned by RANGE (occurred_at))
id uuid=gen_random_uuid(), occurred_at timestamp with time zone=now(), person_id uuid?, organization_id uuid?, membership_id uuid?, kind text, channel channel?, actor text, source_code text?, payload jsonb='{}'::jsonb
- PK activities_pkey: PRIMARY KEY (id, occurred_at)
- idx activities_person_time_idx public.activities USING btree (person_id, occurred_at DESC)
- idx activities_time_brin public.activities USING brin (occurred_at)
- idx activities_kind_idx public.activities USING btree (kind, occurred_at DESC)

### activities_2026_08 (partition FOR VALUES FROM ('2026-08-01 00:00:00+00') TO ('2026-09-01 00:00:00+00'))
id uuid=gen_random_uuid(), occurred_at timestamp with time zone=now(), person_id uuid?, organization_id uuid?, membership_id uuid?, kind text, channel channel?, actor text, source_code text?, payload jsonb='{}'::jsonb
- PK activities_2026_08_pkey: PRIMARY KEY (id, occurred_at)
- idx activities_2026_08_person_id_occurred_at_idx USING btree (person_id, occurred_at DESC)
- idx activities_2026_08_occurred_at_idx USING brin (occurred_at)
- idx activities_2026_08_kind_occurred_at_idx USING btree (kind, occurred_at DESC)

### activities_2026_09 (partition FOR VALUES FROM ('2026-09-01 00:00:00+00') TO ('2026-10-01 00:00:00+00'))
id uuid=gen_random_uuid(), occurred_at timestamp with time zone=now(), person_id uuid?, organization_id uuid?, membership_id uuid?, kind text, channel channel?, actor text, source_code text?, payload jsonb='{}'::jsonb
- PK activities_2026_09_pkey: PRIMARY KEY (id, occurred_at)
- idx activities_2026_09_person_id_occurred_at_idx USING btree (person_id, occurred_at DESC)
- idx activities_2026_09_occurred_at_idx USING brin (occurred_at)
- idx activities_2026_09_kind_occurred_at_idx USING btree (kind, occurred_at DESC)

### activities_2026_10 (partition FOR VALUES FROM ('2026-10-01 00:00:00+00') TO ('2026-11-01 00:00:00+00'))
id uuid=gen_random_uuid(), occurred_at timestamp with time zone=now(), person_id uuid?, organization_id uuid?, membership_id uuid?, kind text, channel channel?, actor text, source_code text?, payload jsonb='{}'::jsonb
- PK activities_2026_10_pkey: PRIMARY KEY (id, occurred_at)
- idx activities_2026_10_person_id_occurred_at_idx USING btree (person_id, occurred_at DESC)
- idx activities_2026_10_occurred_at_idx USING brin (occurred_at)
- idx activities_2026_10_kind_occurred_at_idx USING btree (kind, occurred_at DESC)

### activities_default (partition DEFAULT)
id uuid=gen_random_uuid(), occurred_at timestamp with time zone=now(), person_id uuid?, organization_id uuid?, membership_id uuid?, kind text, channel channel?, actor text, source_code text?, payload jsonb='{}'::jsonb
- PK activities_default_pkey: PRIMARY KEY (id, occurred_at)
- idx activities_default_person_id_occurred_at_idx USING btree (person_id, occurred_at DESC)
- idx activities_default_occurred_at_idx USING brin (occurred_at)
- idx activities_default_kind_occurred_at_idx USING btree (kind, occurred_at DESC)

### affiliations
id uuid=gen_random_uuid(), person_id uuid, organization_id uuid, role_title text? /*Free text, and currently holding two different kinds of thing: a controlled relationship kind (student 858, family 498, counsellor 26, parent 5) and a genuine job title ('IB DP Biology and MYP Science Facilitator'). The first should be an enum under CLAUDE.md's controlled-list rule; the second is correctly free text. Splitting them is an open decision — see D-074 — not yet taken. NULL means the affiliation was established by evidence that does not carry a role, such as an email domain match.*/, is_decision_maker boolean=false, started_on date?, ended_on date?, created_at timestamp with time zone=now()
- CK affiliations_check: CHECK (((ended_on IS NULL) OR (started_on IS NULL) OR (ended_on >= started_on)))
- FK affiliations_organization_id_fkey: FOREIGN KEY (organization_id) REFERENCES organizations(id) ON DELETE CASCADE
- FK affiliations_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE CASCADE
- PK affiliations_pkey: PRIMARY KEY (id)
- idx affiliations_person_idx USING btree (person_id)
- idx affiliations_org_idx USING btree (organization_id) WHERE (ended_on IS NULL)
- idx affiliations_person_org_role_uk UNIQUE USING btree (person_id, organization_id, lower(role_title)) NULLS NOT DISTINCT WHERE (ended_on IS NULL)

### agent_runs
id uuid=gen_random_uuid(), agent text, person_id uuid?, segment text?, model text?, input_tokens integer?, output_tokens integer?, cost_minor bigint?, currency text?, latency_ms integer?, confidence numeric(4,3)?, human_verdict verdict='pending'::verdict, verdict_by text?, verdict_note text?, created_at timestamp with time zone=now(), message_id uuid?, meeting_id uuid?, trigger text?, lifecycle_before text?, lifecycle_after text?, next_action text?, reasoning text?, selected_programme_raw text?, source_system text?, external_id text?
- FK agent_runs_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES meetings(id) ON DELETE SET NULL
- FK agent_runs_message_id_fkey: FOREIGN KEY (message_id) REFERENCES messages(id) ON DELETE SET NULL
- FK agent_runs_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE SET NULL
- PK agent_runs_pkey: PRIMARY KEY (id)
- idx agent_runs_agent_idx USING btree (agent, segment, created_at DESC)
- idx agent_runs_verdict_idx USING btree (agent, human_verdict)
- idx agent_runs_person_idx USING btree (person_id)
- idx agent_runs_message_idx USING btree (message_id)
- idx agent_runs_source_external_ux UNIQUE USING btree (source_system, external_id) WHERE ((source_system IS NOT NULL) AND (external_id IS NOT NULL))
- idx agent_runs_meeting_idx USING btree (meeting_id)

### agent_setting_history
id uuid=gen_random_uuid(), key text, was numeric?, became numeric, reason text, changed_by text, changed_at timestamp with time zone=now()
- PK agent_setting_history_pkey: PRIMARY KEY (id)
- idx agent_setting_history_key_ix USING btree (key, changed_at DESC)

### agent_settings
key text, value numeric, unit text?, reason text, set_by text, set_at timestamp with time zone=now()
- PK agent_settings_pkey: PRIMARY KEY (key)

### allocations
id uuid=gen_random_uuid(), staff_id uuid, location_country text, location_state text?, location_city text?, is_active boolean=true, allocated_by uuid?, allocated_at timestamp with time zone=now(), ended_at timestamp with time zone?, note text?
- CK allocation_active_xor_ended: CHECK ((is_active = (ended_at IS NULL)))
- CK allocation_hierarchy: CHECK (((location_city IS NULL) OR (location_state IS NOT NULL)))
- FK allocations_allocated_by_fkey: FOREIGN KEY (allocated_by) REFERENCES staff(id)
- FK allocations_staff_id_fkey: FOREIGN KEY (staff_id) REFERENCES staff(id) ON DELETE CASCADE
- PK allocations_pkey: PRIMARY KEY (id)
- idx allocations_staff_ix USING btree (staff_id) WHERE is_active
- idx allocations_place_ix USING btree (lower(btrim(location_country))) WHERE is_active

### applications
id uuid=gen_random_uuid(), source_table text, source_row_id uuid, programme_code text, cohort_raw text?, student_name text?, student_email text?, student_phone text?, student_grade text?, parent_name text?, parent_email text?, parent_phone text?, school_raw text?, school_code text?, city text?, state text?, country text?, applied_at timestamp with time zone, paid_at timestamp with time zone? /*When the applicant actually paid. NULL means unpaid. Sourced from the programme table's own payment_date / paid_at. This column is what makes "paid after the meeting" answerable at all.*/, payment_status_raw text?, provider_payment_id text?, payment_provider_hint text? /*Rail inferred from the payment id prefix: pi_ = stripe, pay_ = razorpay. A hint for reconciliation, not authoritative - public.payments is.*/, utm_source text?, utm_medium text?, utm_campaign text?, referral_source text?, person_id uuid?, raw jsonb, needs_review boolean=false, synced_at timestamp with time zone=now()
- FK applications_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE SET NULL
- PK applications_pkey: PRIMARY KEY (id)
- UQ applications_source_uniq: UNIQUE (source_table, source_row_id)
- idx idx_app_email USING btree (lower(student_email))
- idx idx_app_parent USING btree (lower(parent_email))
- idx idx_app_paid USING btree (paid_at)
- idx idx_app_applied USING btree (applied_at)
- idx idx_app_prog USING btree (programme_code, applied_at)
- idx idx_app_school USING btree (school_code, applied_at)
- idx idx_app_person USING btree (person_id)
- idx idx_app_ppid USING btree (provider_payment_id)

### approved_facts
id uuid=gen_random_uuid(), subject_kind text, subject_id uuid?, claim_key text, claim_value text, source_url text?, verified_on date, expires_on date?, created_at timestamp with time zone=now()
- PK approved_facts_pkey: PRIMARY KEY (id)
- idx approved_facts_subject_idx USING btree (subject_kind, subject_id)
- idx approved_facts_live_idx USING btree (claim_key, expires_on)

### autonomy_thresholds
id uuid=gen_random_uuid(), agent text, segment text=''::text, threshold numeric(4,3)=0, runs_behind integer=0, approval_rate numeric(4,3)?, raised_by text?, raised_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK autonomy_thresholds_earned: CHECK (((threshold = (0)::numeric) OR ((runs_behind > 0) AND (approval_rate IS NOT NULL))))
- CK autonomy_thresholds_range: CHECK (((threshold >= (0)::numeric) AND (threshold <= (1)::numeric)))
- CK autonomy_thresholds_rate_range: CHECK (((approval_rate IS NULL) OR ((approval_rate >= (0)::numeric) AND (approval_rate <= (1)::numeric))))
- PK autonomy_thresholds_pkey: PRIMARY KEY (id)
- UQ autonomy_thresholds_agent_segment_key: UNIQUE (agent, segment)

### bank_refunds
id uuid, provider text='wallex'::text, programme_code text?, currency text, amount_minor bigint, refunded_on date?, beneficiary_name text?, payment_id uuid?, link_basis text, needs_review boolean, source_code text='SRC-SYNC-202610-WALLEX'::text, synced_at timestamp with time zone=now()
- CK bank_refunds_amount_minor_check: CHECK ((amount_minor > 0))
- FK bank_refunds_payment_id_fkey: FOREIGN KEY (payment_id) REFERENCES payments(id) ON DELETE SET NULL
- PK bank_refunds_pkey: PRIMARY KEY (id)

### campaigns
id uuid=gen_random_uuid(), name text, motion motion, cohort_id uuid?, owner text?, score_floor smallint=0, require_review boolean=true, is_active boolean=false, source_code text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK campaigns_score_floor_check: CHECK (((score_floor >= 0) AND (score_floor <= 100)))
- FK campaigns_cohort_id_fkey: FOREIGN KEY (cohort_id) REFERENCES cohorts(id)
- PK campaigns_pkey: PRIMARY KEY (id)
- idx campaigns_cohort_idx USING btree (cohort_id)

### cohorts
id uuid=gen_random_uuid(), programme_id uuid, label text, starts_on date, ends_on date?, application_deadline date?, price_minor bigint, currency text, capacity integer?, seats_taken integer=0, stripe_price_id text?, payment_link_url text?, is_open boolean=true, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK cohorts_check: CHECK (((ends_on IS NULL) OR (ends_on >= starts_on)))
- CK cohorts_check1: CHECK (((capacity IS NULL) OR (seats_taken <= capacity)))
- CK cohorts_seats_taken_check: CHECK ((seats_taken >= 0))
- FK cohorts_programme_id_fkey: FOREIGN KEY (programme_id) REFERENCES programmes(id) ON DELETE CASCADE
- PK cohorts_pkey: PRIMARY KEY (id)
- idx cohorts_open_idx USING btree (starts_on) WHERE is_open
- idx cohorts_programme_idx USING btree (programme_id)

### consents
id uuid=gen_random_uuid(), person_id uuid, channel channel, purpose consent_purpose, basis consent_basis, evidence text?, source_code text, granted_at timestamp with time zone=now(), revoked_at timestamp with time zone?, created_at timestamp with time zone=now()
- FK consents_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE CASCADE
- PK consents_pkey: PRIMARY KEY (id)
- idx consents_lookup_idx USING btree (person_id, channel, purpose) WHERE (revoked_at IS NULL)

### frequency_counters
person_id uuid, window_start date, window_kind text, touches integer=0
- FK frequency_counters_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE CASCADE
- PK frequency_counters_pkey: PRIMARY KEY (person_id, window_kind, window_start)

### household_members
household_id uuid, person_id uuid, relation text, created_at timestamp with time zone=now()
- FK household_members_household_id_fkey: FOREIGN KEY (household_id) REFERENCES households(id) ON DELETE CASCADE
- FK household_members_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE CASCADE
- PK household_members_pkey: PRIMARY KEY (household_id, person_id)
- idx household_members_person_idx USING btree (person_id)

### households
id uuid=gen_random_uuid(), label text?, source_code text, created_at timestamp with time zone=now()
- PK households_pkey: PRIMARY KEY (id)

### inbound_records
id uuid=gen_random_uuid(), system_key text, kind inbound_kind, idempotency_key text?, payload jsonb, source_code text, received_at timestamp with time zone=now(), processed_at timestamp with time zone?, person_id uuid?, rejected_at timestamp with time zone?, rejected_reason text?
- CK inbound_not_both_outcomes: CHECK (((processed_at IS NULL) OR (rejected_at IS NULL)))
- CK inbound_payload_bounded: CHECK ((pg_column_size(payload) < 65536))
- CK inbound_payload_is_object: CHECK ((jsonb_typeof(payload) = 'object'::text))
- CK inbound_rejection_has_reason: CHECK (((rejected_at IS NULL) OR (btrim(COALESCE(rejected_reason, ''::text)) <> ''::text)))
- FK inbound_records_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id)
- FK inbound_records_system_key_fkey: FOREIGN KEY (system_key) REFERENCES source_systems(key)
- PK inbound_records_pkey: PRIMARY KEY (id)
- idx inbound_records_idem_uk UNIQUE USING btree (system_key, idempotency_key) NULLS NOT DISTINCT WHERE (idempotency_key IS NOT NULL)
- idx inbound_records_pending_ix USING btree (system_key, received_at) WHERE ((processed_at IS NULL) AND (rejected_at IS NULL))
- idx inbound_records_person_linkedin_ix USING btree (identity_key('linkedin'::text, (payload ->> 'linkedin_url'::text))) WHERE ((kind = 'person'::inbound_kind) AND (processed_at IS NULL) AND (rejected_at IS NULL))
- idx inbound_records_person_email_ix USING btree (identity_key('email'::text, (payload ->> 'email'::text))) WHERE ((kind = 'person'::inbound_kind) AND (processed_at IS NULL) AND (rejected_at IS NULL))

### jobs
id uuid=gen_random_uuid(), kind text, payload jsonb='{}'::jsonb, run_after timestamp with time zone=now(), attempts integer=0, max_attempts integer=5, claimed_at timestamp with time zone?, claimed_by text?, completed_at timestamp with time zone?, failed_at timestamp with time zone?, last_error text?, created_at timestamp with time zone=now()
- PK jobs_pkey: PRIMARY KEY (id)
- idx jobs_claimable_idx USING btree (run_after) WHERE ((claimed_at IS NULL) AND (completed_at IS NULL) AND (failed_at IS NULL))
- idx jobs_leased_idx USING btree (claimed_at) WHERE ((claimed_at IS NOT NULL) AND (completed_at IS NULL) AND (failed_at IS NULL))
- idx jobs_kind_idx USING btree (kind, run_after) WHERE ((completed_at IS NULL) AND (failed_at IS NULL))

### li_connections
id uuid=gen_random_uuid(), user_id uuid, full_name text, linkedin_url text, current_title text?, company text?, seniority text?, relevance_reason text?, personalized_note text?, source_url text?, status text='identified'::text, last_checked_at timestamp with time zone?, created_at timestamp with time zone=now()
- CK li_connections_status_check: CHECK ((status = ANY (ARRAY['identified'::text, 'reviewed'::text, 'request_ready'::text, 'sent_manually'::text, 'connected'::text, 'replied'::text, 'suppressed'::text, 'not_relevant'::text])))
- FK li_connections_user_id_fkey: FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- PK li_connections_pkey: PRIMARY KEY (id)
- UQ li_connections_user_id_linkedin_url_key: UNIQUE (user_id, linkedin_url)
- idx li_connections_user_status_idx USING btree (user_id, status)

### li_content_posts
id uuid=gen_random_uuid(), user_id uuid, planned_for date?, status text='idea'::text, topic text, audience text?, hook text?, body text?, closing_question text?, hashtags text[]='{}'::text[], image_prompt text?, image_url text?, image_alt_text text?, source_refs jsonb='[]'::jsonb, quality_review jsonb='{}'::jsonb, approved_at timestamp with time zone?, published_at timestamp with time zone?, linkedin_post_url text?, publish_error text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), scheduled_at timestamp with time zone?, linkedin_author_urn text?, publish_attempts integer=0, last_publish_attempt_at timestamp with time zone?
- FK li_content_posts_user_id_fkey: FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- PK li_content_posts_pkey: PRIMARY KEY (id)
- idx li_content_posts_user_status_idx USING btree (user_id, status, planned_for)
- idx li_content_posts_due_schedule_idx USING btree (status, scheduled_at) WHERE (scheduled_at IS NOT NULL)

### li_daily_runs
id uuid=gen_random_uuid(), user_id uuid, run_type text, started_at timestamp with time zone=now(), finished_at timestamp with time zone?, status text='running'::text, summary text?, error_message text?, metrics jsonb='{}'::jsonb
- CK li_daily_runs_run_type_check: CHECK ((run_type = ANY (ARRAY['daily_research'::text, 'weekly_editorial'::text, 'manual'::text])))
- CK li_daily_runs_status_check: CHECK ((status = ANY (ARRAY['running'::text, 'completed'::text, 'partial'::text, 'failed'::text])))
- FK li_daily_runs_user_id_fkey: FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- PK li_daily_runs_pkey: PRIMARY KEY (id)
- idx li_daily_runs_user_started_idx USING btree (user_id, started_at DESC)

### li_linkedin_connections
user_id uuid, linkedin_member_urn text?, linkedin_name text?, access_token_ciphertext text?, refresh_token_ciphertext text?, access_token_expires_at timestamp with time zone?, scopes text[]='{}'::text[], connection_status text='not_connected'::text, last_error text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK li_linkedin_connections_connection_status_check: CHECK ((connection_status = ANY (ARRAY['not_connected'::text, 'connected'::text, 'expired'::text, 'error'::text])))
- FK li_linkedin_connections_user_id_fkey: FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- PK li_linkedin_connections_pkey: PRIMARY KEY (user_id)

### li_linkedin_oauth_states
state text, user_id uuid, expires_at timestamp with time zone, created_at timestamp with time zone=now()
- FK li_linkedin_oauth_states_user_id_fkey: FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- PK li_linkedin_oauth_states_pkey: PRIMARY KEY (state)
- idx li_linkedin_oauth_states_expiry_idx USING btree (expires_at)

### li_research_items
id uuid=gen_random_uuid(), user_id uuid, researched_at timestamp with time zone=now(), source_name text?, source_url text?, source_published_at timestamp with time zone?, item_type text='trend'::text, topic text, summary text, relevance_score smallint?, freshness_score smallint?, suggested_angle text?, status text='new'::text, created_at timestamp with time zone=now()
- CK li_research_items_freshness_score_check: CHECK (((freshness_score >= 1) AND (freshness_score <= 5)))
- CK li_research_items_item_type_check: CHECK ((item_type = ANY (ARRAY['trend'::text, 'person'::text, 'conversation'::text, 'company'::text, 'post_idea'::text, 'source'::text])))
- CK li_research_items_relevance_score_check: CHECK (((relevance_score >= 1) AND (relevance_score <= 5)))
- CK li_research_items_status_check: CHECK ((status = ANY (ARRAY['new'::text, 'reviewed'::text, 'used'::text, 'dismissed'::text])))
- FK li_research_items_user_id_fkey: FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- PK li_research_items_pkey: PRIMARY KEY (id)
- idx li_research_items_user_date_idx USING btree (user_id, researched_at DESC)
- idx li_research_items_user_topic_idx USING btree (user_id, topic)

### li_work_updates
id uuid=gen_random_uuid(), user_id uuid, work_date date=CURRENT_DATE, title text, details text, impact_notes text?, evidence_url text?, confidentiality text='public_safe'::text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK li_work_updates_confidentiality_check: CHECK ((confidentiality = ANY (ARRAY['public_safe'::text, 'internal_only'::text, 'confidential'::text])))
- FK li_work_updates_user_id_fkey: FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- PK li_work_updates_pkey: PRIMARY KEY (id)
- idx li_work_updates_user_date_idx USING btree (user_id, work_date DESC)

### meetings
id uuid=gen_random_uuid(), person_id uuid?, organization_id uuid?, host_staff_id uuid?, starts_at timestamp with time zone, ends_at timestamp with time zone?, duration_minutes integer?, source meeting_source, external_id text, status meeting_status='scheduled'::meeting_status, purpose text?, attendee_email_norm text?, attendee_name_raw text?, school_name_raw text?, notes text?, notes_origin fact_origin?, programme_pitched_id uuid?, programme_pitched_raw text?, programme_origin fact_origin?, rescheduled_to_id uuid?, source_code text, needs_review boolean=true, external_ids jsonb='{}'::jsonb, field_provenance jsonb='{}'::jsonb, sources jsonb='[]'::jsonb, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), attendee_phone_norm text?, attendee_phone_raw text?, attendee_designation text?, attendee_country text?, attendee_timezone text?, school_code_raw text?, school_match_confidence numeric?, school_match_source text?, school_match_reviewed boolean?, actual_start timestamp with time zone?, actual_end timestamp with time zone?, join_url text?, recording_url text?, enquiry_text text?, utm_source text?, utm_medium text?, utm_campaign text?, ai_summary text?, ai_notes text?, ai_key_quotes text?, ai_objections text?, ai_commitments text?, ai_next_step text?, ai_interest_level text?, ai_lead_score integer?, ai_lead_temperature text?, ai_lead_segment text?, ai_lifecycle_stage text?, ai_score_reasons text?, ai_recommended_goal text?, ai_recommended_programme_codes text[]?, ai_confidence numeric?, ai_read_at timestamp with time zone?, ai_escalation_needed boolean?, ai_escalation_reason text?, attendance_source text?, attendance_confidence numeric?, attendance_review_required boolean=false, no_show_count integer?, last_agent_decision text?, rescheduled_from_external_id text?, rescheduled_to_external_id text?
- CK meetings_actual_ends_after_start: CHECK (((actual_end IS NULL) OR (actual_start IS NULL) OR (actual_end >= actual_start)))
- CK meetings_ai_block_has_a_timestamp: CHECK (((ai_read_at IS NOT NULL) OR ((ai_summary IS NULL) AND (ai_notes IS NULL) AND (ai_key_quotes IS NULL) AND (ai_objections IS NULL) AND (ai_commitments IS NULL) AND (ai_next_step IS NULL) AND (ai_lead_score IS NULL) AND (ai_score_reasons IS NULL))))
- CK meetings_duration_minutes_check: CHECK (((duration_minutes IS NULL) OR ((duration_minutes >= 0) AND (duration_minutes <= 1440))))
- CK meetings_ends_after_start: CHECK (((ends_at IS NULL) OR (ends_at >= starts_at)))
- CK meetings_external_id_check: CHECK ((btrim(external_id) <> ''::text))
- CK meetings_no_self_reschedule: CHECK ((rescheduled_to_id IS DISTINCT FROM id))
- CK meetings_notes_have_origin: CHECK (((notes IS NULL) OR (notes_origin IS NOT NULL)))
- CK meetings_programme_has_origin: CHECK ((((programme_pitched_id IS NULL) AND (programme_pitched_raw IS NULL)) OR (programme_origin IS NOT NULL)))
- FK meetings_host_staff_id_fkey: FOREIGN KEY (host_staff_id) REFERENCES staff(id) ON DELETE SET NULL
- FK meetings_organization_id_fkey: FOREIGN KEY (organization_id) REFERENCES organizations(id) ON DELETE RESTRICT
- FK meetings_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE RESTRICT
- FK meetings_programme_pitched_id_fkey: FOREIGN KEY (programme_pitched_id) REFERENCES programmes(id) ON DELETE SET NULL
- FK meetings_rescheduled_to_id_fkey: FOREIGN KEY (rescheduled_to_id) REFERENCES meetings(id) ON DELETE SET NULL
- PK meetings_pkey: PRIMARY KEY (id)
- idx meetings_source_external_ux UNIQUE USING btree (source, external_id)
- idx meetings_starts_idx USING btree (starts_at DESC)
- idx meetings_person_idx USING btree (person_id)
- idx meetings_org_idx USING btree (organization_id)
- idx meetings_status_idx USING btree (status)
- idx meetings_unlinked_idx USING btree (starts_at DESC) WHERE (person_id IS NULL)
- idx meetings_review_idx USING btree (starts_at DESC) WHERE needs_review
- idx meetings_escalation_idx USING btree (starts_at DESC) WHERE ai_escalation_needed
- idx meetings_attendance_review_idx USING btree (starts_at DESC) WHERE attendance_review_required
- idx meetings_lead_score_idx USING btree (ai_lead_score DESC NULLS LAST)
- idx meetings_utm_source_idx USING btree (utm_source)

### memberships
id uuid=gen_random_uuid(), person_id uuid, campaign_id uuid, sequence_id uuid?, current_step integer=0, stage lifecycle_stage='new'::lifecycle_stage, next_action_at timestamp with time zone?, next_action_type text?, entered_at timestamp with time zone=now(), exited_at timestamp with time zone?, exited_reason text?, score smallint?, score_reasons jsonb='[]'::jsonb, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), owner_staff_id uuid? /*Which staff member owns this lead. Read by the scoping policies in 008. NULL is a real state — an unclaimed lead, visible only at scope_kind = 'all'.*/
- CK memberships_score_check: CHECK (((score >= 0) AND (score <= 100)))
- FK memberships_campaign_id_fkey: FOREIGN KEY (campaign_id) REFERENCES campaigns(id) ON DELETE CASCADE
- FK memberships_owner_staff_id_fkey: FOREIGN KEY (owner_staff_id) REFERENCES staff(id) ON DELETE SET NULL
- FK memberships_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE CASCADE
- FK memberships_sequence_id_fkey: FOREIGN KEY (sequence_id) REFERENCES sequences(id)
- PK memberships_pkey: PRIMARY KEY (id)
- UQ memberships_person_id_campaign_id_key: UNIQUE (person_id, campaign_id)
- idx memberships_due_idx USING btree (next_action_at) WHERE ((exited_at IS NULL) AND (next_action_at IS NOT NULL))
- idx memberships_orphan_idx USING btree (stage) WHERE ((exited_at IS NULL) AND (next_action_at IS NULL))
- idx memberships_campaign_idx USING btree (campaign_id)
- idx memberships_sequence_idx USING btree (sequence_id)
- idx memberships_owner_idx USING btree (owner_staff_id)
- idx memberships_owner_person_idx USING btree (owner_staff_id, person_id)

### merge_lineage
id uuid=gen_random_uuid(), winner_id uuid, loser_id uuid, matched_by text, confidence numeric(4,3)?, field_sources jsonb='{}'::jsonb, merged_at timestamp with time zone=now(), unmerged_at timestamp with time zone?
- FK merge_lineage_loser_id_fkey: FOREIGN KEY (loser_id) REFERENCES people(id) ON DELETE CASCADE
- FK merge_lineage_winner_id_fkey: FOREIGN KEY (winner_id) REFERENCES people(id) ON DELETE CASCADE
- PK merge_lineage_pkey: PRIMARY KEY (id)
- idx merge_lineage_winner_idx USING btree (winner_id)
- idx merge_lineage_loser_idx USING btree (loser_id)

### messages
id uuid=gen_random_uuid(), person_id uuid?, membership_id uuid?, channel channel, direction text, subject text?, body text?, status text='draft'::text, reply_intent text?, reply_confidence numeric(4,3)?, provider_message_id text?, sent_at timestamp with time zone?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), meeting_id uuid?, source_system text?, external_id text?, outside_gate boolean=false, template_slug text?, to_address text?, from_address text?, open_count integer?, click_count integer?, first_opened_at timestamp with time zone?, last_opened_at timestamp with time zone?, replied_at timestamp with time zone?, bounced_at timestamp with time zone?
- CK messages_belongs_to_something: CHECK (((person_id IS NOT NULL) OR (meeting_id IS NOT NULL)))
- CK messages_direction_check: CHECK ((direction = ANY (ARRAY['outbound'::text, 'inbound'::text])))
- FK messages_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES meetings(id) ON DELETE SET NULL
- FK messages_membership_id_fkey: FOREIGN KEY (membership_id) REFERENCES memberships(id) ON DELETE SET NULL
- FK messages_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE CASCADE
- PK messages_pkey: PRIMARY KEY (id)
- idx messages_person_idx USING btree (person_id, created_at DESC)
- idx messages_review_idx USING btree (status) WHERE (status = 'draft'::text)
- idx messages_membership_idx USING btree (membership_id)
- idx messages_source_external_ux UNIQUE USING btree (source_system, external_id) WHERE ((source_system IS NOT NULL) AND (external_id IS NOT NULL))
- idx messages_meeting_idx USING btree (meeting_id)
- idx messages_sent_idx USING btree (sent_at DESC NULLS LAST)
- idx messages_outside_gate_idx USING btree (sent_at DESC) WHERE outside_gate

### org_aliases
id uuid=gen_random_uuid(), organization_id uuid, raw_name text, matched_by text, confidence numeric(4,3)?, created_at timestamp with time zone=now(), times_seen integer=1 /*How often this raw spelling has been seen. A spelling seen 200 times is worth more than one seen once, and it tells you which imports are noisy.*/, reviewed boolean=false, reviewed_by text?, reviewed_at timestamp with time zone?
- CK org_aliases_times_seen_positive: CHECK ((times_seen >= 1))
- FK org_aliases_organization_id_fkey: FOREIGN KEY (organization_id) REFERENCES organizations(id) ON DELETE CASCADE
- PK org_aliases_pkey: PRIMARY KEY (id)
- idx org_aliases_raw_uk UNIQUE USING btree (lower(raw_name))
- idx org_aliases_org_idx USING btree (organization_id)
- idx org_aliases_unreviewed_idx USING btree (confidence) WHERE ((NOT reviewed) AND (matched_by = 'trigram'::text))
- idx org_aliases_raw_trgm_ix USING gin (lower(btrim(raw_name)) gin_trgm_ops)

### org_group_domains
domain text, group_id uuid, added_by text, added_at timestamp with time zone=now(), note text?
- FK org_group_domains_group_id_fkey: FOREIGN KEY (group_id) REFERENCES org_groups(id) ON DELETE CASCADE
- PK org_group_domains_pkey: PRIMARY KEY (domain)
- idx org_group_domains_group_ix USING btree (group_id)

### org_groups
id uuid=gen_random_uuid(), name text, kind text, shared_domain text?, campaign_owner uuid? /*The one person who may approach this group. Without an owner, "who mailed them last" only gets an answer after the complaint.*/, schools_counted integer?, contacts_counted integer?, notes text?, source_code text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), name_is_placeholder boolean=false /*True when the name is the shared domain because no district name could be derived. The domain is accurate; it is simply not what anybody calls this group. A screen should ask for the real one.*/
- CK org_groups_kind_check: CHECK ((kind = ANY (ARRAY['school_group'::text, 'district'::text, 'university_system'::text])))
- FK org_groups_campaign_owner_fkey: FOREIGN KEY (campaign_owner) REFERENCES staff(id) ON DELETE SET NULL
- PK org_groups_pkey: PRIMARY KEY (id)
- idx org_groups_name_uk UNIQUE USING btree (lower(btrim(name)))
- idx org_groups_domain_uk UNIQUE USING btree (shared_domain) WHERE (shared_domain IS NOT NULL)
- idx org_groups_owner_ix USING btree (campaign_owner) WHERE (campaign_owner IS NOT NULL)

### organizations
id uuid=gen_random_uuid(), name text, domain text? /*The organisation's identity. Unique where set, enforced by organizations_domain_uk. Match on THIS, never on name — D-068 measured trigram similarity on school names and found "acs international schools" ~= "FK International Schools" at 0.76, different schools, because school names are mostly boilerplate. A domain qualifies only if exactly ONE organisation uses it. That single structural rule excluded three different families of bad value at once: real school groups (pathways.in, enkoschools.com, dpsi.ac.in), free mail a hand-written blocklist had missed (yahoo.co.uk, yahoo.fr), and misspellings of gmail (gmil.com, gmail.con) — the last of which no blocklist would ever have caught. It also resolved the one apparent conflict correctly rather than flagging it: Pathways Noida and Pathways Gurgaon each took their own branch domain, because the shared group domain failed the exclusivity test. Hence 0 domain conflicts, which is a result and not an unrun check.*/, linkedin_url text?, website text?, school_category text?, curriculum text?, grades text?, accreditation text?, student_count integer?, max_fees_minor bigint?, fees_currency text?, academic_calendar text?, country text?, region text?, city text?, address text?, tier smallint=3, tier_reason text?, source_code text, zone data_zone='raw'::data_zone, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), code text? /*Stable text business key, ported from meetings.school_registry.code. Every foreign key in that database points at it, so it is preserved rather than remapped to the uuid. See docs/decisions.md D-009.*/, needs_review boolean=false, sources jsonb='[]'::jsonb, field_provenance jsonb='{}'::jsonb, external_ids jsonb='{}'::jsonb, state text?, group_id uuid?
- CK organizations_tier_check: CHECK (((tier >= 1) AND (tier <= 3)))
- FK organizations_group_id_fkey: FOREIGN KEY (group_id) REFERENCES org_groups(id) ON DELETE SET NULL
- PK organizations_pkey: PRIMARY KEY (id)
- idx organizations_domain_uk UNIQUE USING btree (lower(domain)) WHERE (domain IS NOT NULL)
- idx organizations_linkedin_uk UNIQUE USING btree (lower(linkedin_url)) WHERE (linkedin_url IS NOT NULL)
- idx organizations_name_trgm USING gin (lower(name) gin_trgm_ops)
- idx organizations_country_tier USING btree (country, tier)
- idx organizations_code_key UNIQUE USING btree (code)
- idx organizations_needs_review_idx USING btree (created_at) WHERE needs_review
- idx organizations_external_ids_gin USING gin (external_ids jsonb_path_ops)
- idx organizations_name_trgm_ix USING gin (lower(btrim(name)) gin_trgm_ops)
- idx organizations_state_idx USING btree (lower(btrim(state))) WHERE (state IS NOT NULL)
- idx organizations_group_ix USING btree (group_id) WHERE (group_id IS NOT NULL)
- idx organizations_name_lower_ix USING btree (lower(btrim(name)))

### payments
id uuid=gen_random_uuid(), person_id uuid? /*NULLABLE on purpose. A payment whose payer email matches nobody is still a real payment; dropping it to satisfy a foreign key would lose revenue. Unmatched rows carry needs_review.*/, cohort_id uuid?, amount_minor bigint, currency text, status payment_status, occurred_at timestamp with time zone, description text?, source_code text, needs_review boolean=false, external_ids jsonb='{}'::jsonb, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), provider text='stripe'::text /*Who processed it. DEFAULT 'stripe' only so that 014's existing 10,386 rows backfill without a second pass — a new payment must state its provider.*/, provider_payment_id text? /*The provider's own id: a Stripe payment_intent, a Razorpay pay_… . Unique per provider, and what makes a payments wave idempotent.*/, payer_email text? /*The address that paid. The join key from a payment to a lead - person_id is authoritative where set, this is the fallback the dashboard can match on.*/, payer_name text?, programme_code text?, utm_source text?, utm_medium text?, utm_campaign text?, method text? /*Razorpay only: upi / card / netbanking. Useful for reconciliation.*/, fee_minor bigint?, refunded_minor bigint?
- CK payments_amount_positive: CHECK ((amount_minor > 0))
- CK payments_currency_iso: CHECK ((currency ~ '^[A-Z]{3}$'::text))
- CK payments_provider_known: CHECK ((provider = ANY (ARRAY['stripe'::text, 'razorpay'::text, 'wallex'::text, 'manual'::text, 'dbs'::text])))
- FK payments_cohort_id_fkey: FOREIGN KEY (cohort_id) REFERENCES cohorts(id) ON DELETE SET NULL
- FK payments_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE SET NULL
- PK payments_pkey: PRIMARY KEY (id)
- UQ payments_provider_ref_uniq: UNIQUE (provider, provider_payment_id)
- idx payments_person_idx USING btree (person_id)
- idx payments_cohort_idx USING btree (cohort_id)
- idx payments_occurred_idx USING btree (occurred_at DESC)
- idx payments_currency_idx USING btree (currency)
- idx payments_review_idx USING btree (occurred_at) WHERE needs_review
- idx payments_unlinked_idx USING btree (occurred_at) WHERE (person_id IS NULL)
- idx payments_provider_payment_uk UNIQUE USING btree (provider, provider_payment_id) WHERE (provider_payment_id IS NOT NULL)
- idx idx_pay_email USING btree (lower(payer_email))
- idx idx_pay_prov USING btree (provider, occurred_at)
- idx idx_pay_utm USING btree (utm_source)

### people
id uuid=gen_random_uuid(), full_name text?, first_name text?, last_name text?, role person_role='unknown'::person_role, title text?, seniority text?, is_minor boolean=false, graduation_year integer? /*Grade is DERIVED from this, never stored as text (CLAUDE.md). Where a source gave a grade string and no year — the huso and yale application tables do — the parsed grade number is kept in external_ids.grade_parsed and this stays NULL. A grade without a year cannot become a year without inventing one.*/, country text?, timezone text?, stage lifecycle_stage='new'::lifecycle_stage, zone data_zone='raw'::data_zone, source_code text, merged_into uuid?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), needs_review boolean=false, sources jsonb='[]'::jsonb, field_provenance jsonb='{}'::jsonb /*Which source_code won each field, as {"full_name":"SRC-WEB-202608-LEO"}. Written by packages/core planMerge(). Without it a merge is irreversible.*/, external_ids jsonb='{}'::jsonb, city text?, state text?
- FK people_merged_into_fkey: FOREIGN KEY (merged_into) REFERENCES people(id)
- PK people_pkey: PRIMARY KEY (id)
- idx people_stage_idx USING btree (stage) WHERE (merged_into IS NULL)
- idx people_name_trgm USING gin (lower(COALESCE(full_name, ''::text)) gin_trgm_ops)
- idx people_minor_idx USING btree (is_minor) WHERE is_minor
- idx people_merged_idx USING btree (merged_into) WHERE (merged_into IS NOT NULL)
- idx people_needs_review_idx USING btree (created_at) WHERE (needs_review AND (merged_into IS NULL))
- idx people_external_ids_gin USING gin (external_ids jsonb_path_ops)
- idx people_city_idx USING btree (lower(btrim(city))) WHERE (city IS NOT NULL)
- idx people_state_idx USING btree (lower(btrim(state))) WHERE (state IS NOT NULL)

### person_identities
id uuid=gen_random_uuid(), person_id uuid, kind identity_kind, value_raw text, value_norm text, is_primary boolean=false, verified_at timestamp with time zone?, verify_status text?, source_code text, created_at timestamp with time zone=now()
- FK person_identities_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE CASCADE
- PK person_identities_pkey: PRIMARY KEY (id)
- idx person_identities_uk UNIQUE USING btree (kind, value_norm)
- idx person_identities_person_idx USING btree (person_id)

### programme_aliases
id uuid=gen_random_uuid(), programme_id uuid?, raw_name text, name_key text, kind text='programme'::text, confidence numeric?, matched_by text?, note text?, reviewed boolean=false, reviewed_by uuid?, reviewed_at timestamp with time zone?, times_seen integer=0, created_at timestamp with time zone=now()
- CK programme_aliases_confidence_check: CHECK (((confidence IS NULL) OR ((confidence >= (0)::numeric) AND (confidence <= (1)::numeric))))
- CK programme_aliases_kind_check: CHECK ((kind = ANY (ARRAY['programme'::text, 'family'::text, 'unknown'::text])))
- CK programme_aliases_kind_matches_target: CHECK ((((kind = 'programme'::text) AND (programme_id IS NOT NULL)) OR ((kind <> 'programme'::text) AND (programme_id IS NULL))))
- FK programme_aliases_programme_id_fkey: FOREIGN KEY (programme_id) REFERENCES programmes(id) ON DELETE RESTRICT
- FK programme_aliases_reviewed_by_fkey: FOREIGN KEY (reviewed_by) REFERENCES staff(id) ON DELETE SET NULL
- PK programme_aliases_pkey: PRIMARY KEY (id)
- idx programme_aliases_name_key_ux UNIQUE USING btree (name_key)
- idx programme_aliases_programme_idx USING btree (programme_id)
- idx programme_aliases_unreviewed_idx USING btree (created_at) WHERE (NOT reviewed)

### programme_codes
code text, category_code text?, category_name text?, organisation_code text?, organisation_name text?, programme_part text?, programme_name text?, school_code text?, school_name text?, country text?, batch text?, starts_on date?, synced_at timestamp with time zone=now()
- PK programme_codes_pkey: PRIMARY KEY (code)

### programmes
id uuid=gen_random_uuid(), code text, name text, partner_name text?, category text?, format text?, age_min integer?, age_max integer?, summary text?, eligibility text?, programme_url text?, is_active boolean=true, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), platform_key text? /*The Learning Portal's platform_registry.platform_key. Enrolments, sessions and notifications all join on it. Carried here so that app keeps working unchanged. See docs/decisions.md D-010.*/
- PK programmes_pkey: PRIMARY KEY (id)
- UQ programmes_code_key: UNIQUE (code)
- idx programmes_platform_key_key UNIQUE USING btree (platform_key)

### refunds
id uuid=gen_random_uuid(), payment_id uuid, amount_minor bigint, currency text, is_full boolean, occurred_at timestamp with time zone? /*NULLABLE. The legacy status string carries an amount but no date, so for every migrated row this is NULL. Null means "we know it happened and not when".*/, source_code text, external_ids jsonb='{}'::jsonb, created_at timestamp with time zone=now()
- CK refunds_amount_positive: CHECK ((amount_minor > 0))
- CK refunds_currency_iso: CHECK ((currency ~ '^[A-Z]{3}$'::text))
- FK refunds_payment_id_fkey: FOREIGN KEY (payment_id) REFERENCES payments(id) ON DELETE CASCADE
- PK refunds_pkey: PRIMARY KEY (id)
- idx refunds_payment_idx USING btree (payment_id)
- idx refunds_occurred_idx USING btree (occurred_at DESC)

### schema_migrations
version text, filename text, checksum text, applied_at timestamp with time zone=now()
- PK schema_migrations_pkey: PRIMARY KEY (version)

### send_attempts
id uuid=gen_random_uuid(), person_id uuid?, message_id uuid?, channel channel, requested_by text, allowed boolean, failed_check text?, created_at timestamp with time zone=now()
- FK send_attempts_message_id_fkey: FOREIGN KEY (message_id) REFERENCES messages(id) ON DELETE SET NULL
- FK send_attempts_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE SET NULL
- PK send_attempts_pkey: PRIMARY KEY (id)
- idx send_attempts_blocked_idx USING btree (failed_check, created_at DESC) WHERE (NOT allowed)
- idx send_attempts_person_idx USING btree (person_id)
- idx send_attempts_message_idx USING btree (message_id)

### sensitive_access_log
id bigint=nextval('sensitive_access_log_id_seq'::regclass), staff_id uuid?, grant_key staff_grant, subject_kind text, subject_id uuid?, context text?, at timestamp with time zone=now()
- CK sensitive_access_log_subject_kind_check: CHECK ((subject_kind = ANY (ARRAY['person'::text, 'message'::text, 'payment'::text])))
- FK sensitive_access_log_staff_id_fkey: FOREIGN KEY (staff_id) REFERENCES staff(id) ON DELETE SET NULL
- PK sensitive_access_log_pkey: PRIMARY KEY (id)
- idx sensitive_access_log_staff_idx USING btree (staff_id, at DESC)
- idx sensitive_access_log_subject_idx USING btree (subject_kind, subject_id)
- idx sensitive_access_log_at_idx USING btree (at DESC)

### sequence_steps
id uuid=gen_random_uuid(), sequence_id uuid, step_number integer, channel channel, delay_hours integer=0, template_key text?, requires_human boolean=true, branch_on jsonb='{}'::jsonb
- FK sequence_steps_sequence_id_fkey: FOREIGN KEY (sequence_id) REFERENCES sequences(id) ON DELETE CASCADE
- PK sequence_steps_pkey: PRIMARY KEY (id)
- UQ sequence_steps_sequence_id_step_number_key: UNIQUE (sequence_id, step_number)

### sequences
id uuid=gen_random_uuid(), campaign_id uuid, version integer=1, is_active boolean=true, created_at timestamp with time zone=now()
- FK sequences_campaign_id_fkey: FOREIGN KEY (campaign_id) REFERENCES campaigns(id) ON DELETE CASCADE
- PK sequences_pkey: PRIMARY KEY (id)
- UQ sequences_campaign_id_version_key: UNIQUE (campaign_id, version)

### source_codes
code text, origin text, owner text, purpose text?, lawful_basis consent_basis?, lia_reference text?, notes text?, created_at timestamp with time zone=now(), retired_at timestamp with time zone?
- CK source_codes_format: CHECK ((code ~ '^SRC-[A-Z0-9]+-[0-9]{6}-[A-Z0-9-]+$'::text))
- CK source_codes_owner_present: CHECK ((btrim(owner) <> ''::text))
- PK source_codes_pkey: PRIMARY KEY (code)
- idx source_codes_owner_idx USING btree (owner)
- idx source_codes_origin_idx USING btree (origin)
- idx source_codes_live_idx USING btree (created_at) WHERE (retired_at IS NULL)

### source_systems
key text, label text, purpose text, app_url text?, is_expected boolean=true, connected_at timestamp with time zone? /*First observed record under this prefix. Written by record_source_arrival() from data, never asserted by hand.*/, sort_order smallint=99, created_at timestamp with time zone=now(), ingest_key_hash bytea? /*SHA-256 of the per-app ingest secret. The plaintext is shown once at rotation and is not recoverable. 030.*/, ingest_key_set_at timestamp with time zone?, ingest_enabled boolean=false /*Off by default. An app with a key still cannot write until an owner turns this on. 030.*/, deep_link_template text? /*URL template for jumping from a queued row to the record in the source app. Supports {id}, {entity}, {kind}. NULL falls back to app_url. 032.*/, default_channel channel? /*The channel a claim from this system carries when it does not say. A payload naming its own channel wins, so LinkedIn Manager can start sending posts or email without a code change.*/
- CK source_systems_connected_was_expected: CHECK (((connected_at IS NULL) OR is_expected))
- CK source_systems_key_shape: CHECK ((key ~ '^[A-Z][A-Z0-9]{2,15}$'::text))
- PK source_systems_pkey: PRIMARY KEY (key)

### staff
id uuid=gen_random_uuid(), auth_user_id uuid?, email text, full_name text?, job_title text?, status staff_status='invited'::staff_status, manager_id uuid?, starts_on date?, ends_on date?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), requested_at timestamp with time zone?, decided_by uuid? /*The owner or admin who approved or rejected this person. NULL for the two bootstrap owners, who predate the queue.*/, decided_at timestamp with time zone?, decision_note text? /*Why. Required on rejection, optional on approval — a refusal that nobody explained is one somebody will silently reverse.*/
- CK staff_dates_ordered: CHECK (((ends_on IS NULL) OR (starts_on IS NULL) OR (ends_on >= starts_on)))
- CK staff_not_own_manager: CHECK ((manager_id IS DISTINCT FROM id))
- FK staff_auth_user_id_fkey: FOREIGN KEY (auth_user_id) REFERENCES auth.users(id) ON DELETE SET NULL
- FK staff_decided_by_fkey: FOREIGN KEY (decided_by) REFERENCES staff(id)
- FK staff_manager_id_fkey: FOREIGN KEY (manager_id) REFERENCES staff(id)
- PK staff_pkey: PRIMARY KEY (id)
- UQ staff_auth_user_id_key: UNIQUE (auth_user_id)
- idx staff_email_key UNIQUE USING btree (lower(email))
- idx staff_auth_user_idx USING btree (auth_user_id)
- idx staff_manager_idx USING btree (manager_id)
- idx staff_active_idx USING btree (auth_user_id) WHERE (status = 'active'::staff_status)
- idx staff_email_uk UNIQUE USING btree (lower(btrim(email)))
- idx staff_pending_idx USING btree (requested_at) WHERE (status = 'requested'::staff_status)

### staff_grants
staff_id uuid, grant_key staff_grant, reason text /*Why this person holds this grant. NOT NULL because unexplained elevated access is indistinguishable from a mistake six months later.*/, granted_by uuid?, granted_at timestamp with time zone=now(), expires_on date=(CURRENT_DATE + 90)
- CK staff_grants_reason_present: CHECK ((btrim(reason) <> ''::text))
- FK staff_grants_granted_by_fkey: FOREIGN KEY (granted_by) REFERENCES staff(id)
- FK staff_grants_staff_id_fkey: FOREIGN KEY (staff_id) REFERENCES staff(id) ON DELETE CASCADE
- PK staff_grants_pkey: PRIMARY KEY (staff_id, grant_key)
- idx staff_grants_key_idx USING btree (grant_key)
- idx staff_grants_granted_by_idx USING btree (granted_by)
- idx staff_grants_expiring_idx USING btree (expires_on)

### staff_invitations
id uuid=gen_random_uuid(), email text, full_name text?, job_title text?, roles staff_role[], grants staff_grant[]='{}'::staff_grant[], scopes jsonb='[]'::jsonb, invited_by uuid, expires_at timestamp with time zone=(now() + '7 days'::interval), accepted_at timestamp with time zone?, revoked_at timestamp with time zone?, created_at timestamp with time zone=now()
- CK staff_invitations_not_both: CHECK (((accepted_at IS NULL) OR (revoked_at IS NULL)))
- FK staff_invitations_invited_by_fkey: FOREIGN KEY (invited_by) REFERENCES staff(id)
- PK staff_invitations_pkey: PRIMARY KEY (id)
- idx staff_invite_open_key UNIQUE USING btree (lower(email)) WHERE ((accepted_at IS NULL) AND (revoked_at IS NULL))
- idx staff_invitations_invited_by_idx USING btree (invited_by)

### staff_role_assignments
staff_id uuid, role staff_role, granted_by uuid?, granted_at timestamp with time zone=now()
- FK staff_role_assignments_granted_by_fkey: FOREIGN KEY (granted_by) REFERENCES staff(id)
- FK staff_role_assignments_staff_id_fkey: FOREIGN KEY (staff_id) REFERENCES staff(id) ON DELETE CASCADE
- PK staff_role_assignments_pkey: PRIMARY KEY (staff_id, role)
- idx staff_role_assignments_role_idx USING btree (role)
- idx staff_role_assignments_granted_by_idx USING btree (granted_by)

### staff_scopes
staff_id uuid, scope_kind text, scope_value text=''::text
- CK staff_scopes_scope_kind_check: CHECK ((scope_kind = ANY (ARRAY['own'::text, 'team'::text, 'programme'::text, 'region'::text, 'all'::text])))
- CK staff_scopes_value_matches_kind: CHECK ((((scope_kind = ANY (ARRAY['own'::text, 'all'::text])) AND (scope_value = ''::text)) OR ((scope_kind = ANY (ARRAY['team'::text, 'programme'::text, 'region'::text])) AND (btrim(scope_value) <> ''::text))))
- FK staff_scopes_staff_id_fkey: FOREIGN KEY (staff_id) REFERENCES staff(id) ON DELETE CASCADE
- PK staff_scopes_pkey: PRIMARY KEY (staff_id, scope_kind, scope_value)
- idx staff_scopes_kind_idx USING btree (scope_kind, scope_value)

### staff_targets
staff_id uuid, rows_per_day integer=100, rows_per_week integer=700, meetings_per_month integer=20, is_part_time boolean=false, set_by uuid?, updated_at timestamp with time zone=now()
- CK staff_targets_meetings_per_month_check: CHECK ((meetings_per_month >= 0))
- CK staff_targets_rows_per_day_check: CHECK ((rows_per_day > 0))
- CK staff_targets_rows_per_week_check: CHECK ((rows_per_week > 0))
- FK staff_targets_set_by_fkey: FOREIGN KEY (set_by) REFERENCES staff(id)
- FK staff_targets_staff_id_fkey: FOREIGN KEY (staff_id) REFERENCES staff(id) ON DELETE CASCADE
- PK staff_targets_pkey: PRIMARY KEY (staff_id)

### suppressions
id uuid=gen_random_uuid(), identity_hash bytea /*sha256 of '<kind>:<value_norm>' — computed ONLY by packages/core/consent/suppressionHash(), stored as bytea via decode(hex). For a person holding no identities yet, sha256 of 'person:<person_id>' so two such people cannot collide on suppressions_uk. Never compute this in SQL.*/, kind identity_kind, person_id uuid?, reason suppression_reason, suppressed_at timestamp with time zone=now()
- FK suppressions_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE SET NULL
- PK suppressions_pkey: PRIMARY KEY (id)
- idx suppressions_uk UNIQUE USING btree (kind, identity_hash)
- idx suppressions_person_idx USING btree (person_id)

### tasks
id uuid=gen_random_uuid(), kind text, person_id uuid?, message_id uuid?, channel channel?, reason text, assigned_to text?, due_at timestamp with time zone?, payload jsonb='{}'::jsonb, completed_at timestamp with time zone?, completed_by text?, created_at timestamp with time zone=now(), meeting_id uuid?, description text?, priority text?, status text?, created_by_agent boolean=false, snooze_until timestamp with time zone?, source_system text?, external_id text?
- FK tasks_meeting_id_fkey: FOREIGN KEY (meeting_id) REFERENCES meetings(id) ON DELETE SET NULL
- FK tasks_message_id_fkey: FOREIGN KEY (message_id) REFERENCES messages(id) ON DELETE SET NULL
- FK tasks_person_id_fkey: FOREIGN KEY (person_id) REFERENCES people(id) ON DELETE CASCADE
- PK tasks_pkey: PRIMARY KEY (id)
- idx tasks_person_idx USING btree (person_id)
- idx tasks_message_idx USING btree (message_id)
- idx tasks_open_idx USING btree (kind, due_at) WHERE (completed_at IS NULL)
- idx tasks_source_external_ux UNIQUE USING btree (source_system, external_id) WHERE ((source_system IS NOT NULL) AND (external_id IS NOT NULL))
- idx tasks_meeting_idx USING btree (meeting_id)

## Part 2: public research_* tables (research_a .. research_l)

### research_activity
id uuid=gen_random_uuid(), channel text='system'::text, title text, detail text=''::text, status text='info'::text, actor text='system'::text, at timestamp with time zone=now(), ms integer?
- PK research_activity_pkey: PRIMARY KEY (id)
- idx research_activity_at_idx USING btree (at DESC)
- idx research_activity_status_idx USING btree (status)

### research_agent_runs
id uuid=gen_random_uuid(), agent text, headline text=''::text, summary text=''::text, findings jsonb='[]'::jsonb, actions jsonb='[]'::jsonb, ran_by text='system'::text, ran_at timestamp with time zone=now()
- PK research_agent_runs_pkey: PRIMARY KEY (id)
- idx research_agent_runs_ran_at_idx USING btree (ran_at DESC)
- idx research_agent_runs_agent_idx USING btree (agent)

### research_agent_settings
id integer=1, enabled boolean=false, model text='gpt-5-mini'::text, max_searches integer=10, daily_budget_usd numeric(10,2)=25, concurrency integer=4, updated_by text=''::text, updated_at timestamp with time zone=now(), agent_fallback boolean=false
- CK research_agent_settings_id_check: CHECK ((id = 1))
- PK research_agent_settings_pkey: PRIMARY KEY (id)

### research_allocations
id uuid=gen_random_uuid(), location_id uuid, researcher_id uuid, daily_target integer=100, weekly_target integer=700, status text='active'::text, assigned_by uuid?, created_at timestamp with time zone=now(), target_rows integer=0, due_on date?, started_on date=CURRENT_DATE
- CK research_allocations_status_check: CHECK ((status = ANY (ARRAY['active'::text, 'paused'::text, 'done'::text])))
- FK research_allocations_assigned_by_fkey: FOREIGN KEY (assigned_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_allocations_location_id_fkey: FOREIGN KEY (location_id) REFERENCES research_locations(id) ON DELETE CASCADE
- FK research_allocations_researcher_id_fkey: FOREIGN KEY (researcher_id) REFERENCES research_staff_profiles(id) ON DELETE CASCADE
- PK research_allocations_pkey: PRIMARY KEY (id)
- idx research_allocations_researcher_idx USING btree (researcher_id)

### research_alumni
id uuid=gen_random_uuid(), serial bigint=nextval('research_alumni_serial_seq'::regclass), dataset_id uuid, student_name text, student_email text=''::text, student_phone text=''::text, grade text=''::text, grade_raw text=''::text, school_name text=''::text, school_contact_id uuid?, city text=''::text, state text=''::text, country text=''::text, region text=''::text, program_type text=''::text, program_name text=''::text, program_year integer?, start_month date?, start_month_raw text=''::text, cohort_group text=''::text, master_sheet text=''::text, payment_status text=''::text, parent_name text=''::text, parent_email text=''::text, parent_phone text=''::text, parent_contact_id uuid?, counsellor_name text=''::text, counsellor_email text=''::text, counsellor_contact_id uuid?, upload_id uuid?, upload_tab_id uuid?, source_row_number integer?, source_file text=''::text, created_by uuid?, created_at timestamp with time zone=now(), deleted_at timestamp with time zone?, deleted_by uuid?, student_person_id uuid?, parent_person_id uuid?, counsellor_person_id uuid?, school_id uuid?
- CK research_alumni_payment_status_chk: CHECK ((payment_status = ANY (ARRAY[''::text, 'Paid'::text, 'Unpaid'::text])))
- CK research_alumni_program_type_chk: CHECK ((program_type = ANY (ARRAY[''::text, 'Online'::text, 'Offline'::text])))
- FK research_alumni_counsellor_contact_id_fkey: FOREIGN KEY (counsellor_contact_id) REFERENCES research_contacts(id) ON DELETE SET NULL
- FK research_alumni_counsellor_person_id_fkey: FOREIGN KEY (counsellor_person_id) REFERENCES research_people(id) ON DELETE SET NULL
- FK research_alumni_dataset_id_fkey: FOREIGN KEY (dataset_id) REFERENCES research_datasets(id) ON DELETE CASCADE
- FK research_alumni_parent_contact_id_fkey: FOREIGN KEY (parent_contact_id) REFERENCES research_contacts(id) ON DELETE SET NULL
- FK research_alumni_parent_person_id_fkey: FOREIGN KEY (parent_person_id) REFERENCES research_people(id) ON DELETE SET NULL
- FK research_alumni_school_contact_id_fkey: FOREIGN KEY (school_contact_id) REFERENCES research_contacts(id) ON DELETE SET NULL
- FK research_alumni_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE SET NULL
- FK research_alumni_student_person_id_fkey: FOREIGN KEY (student_person_id) REFERENCES research_people(id) ON DELETE SET NULL
- PK research_alumni_pkey: PRIMARY KEY (id)
- idx research_alumni_participation_key UNIQUE USING btree (lower(student_email), lower(program_name), COALESCE(program_year, 0)) WHERE ((student_email <> ''::text) AND (deleted_at IS NULL))
- idx research_alumni_nameless_key UNIQUE USING btree (lower(student_name), lower(school_name), lower(program_name), COALESCE(program_year, 0)) WHERE ((student_email = ''::text) AND (deleted_at IS NULL))
- idx research_alumni_dataset_idx USING btree (dataset_id) WHERE (deleted_at IS NULL)
- idx research_alumni_school_idx USING btree (lower(school_name)) WHERE (deleted_at IS NULL)
- idx research_alumni_school_contact_idx USING btree (school_contact_id) WHERE (deleted_at IS NULL)
- idx research_alumni_year_idx USING btree (program_year) WHERE (deleted_at IS NULL)
- idx research_alumni_school_id_idx USING btree (school_id)

### research_applications
id uuid=gen_random_uuid(), application_ref text, student_code text=''::text, student_person_id uuid?, parent_person_id uuid?, counsellor_person_id uuid?, school_id uuid?, program_code text=''::text, program_name text=''::text, program_category text=''::text, grade text=''::text, payment_status text=''::text, amount numeric?, currency text=''::text, registration_status text=''::text, applied_at timestamp with time zone?, raw jsonb='{}'::jsonb, upload_tag text=''::text, created_by text=''::text, created_at timestamp with time zone=now()
- FK research_applications_counsellor_person_id_fkey: FOREIGN KEY (counsellor_person_id) REFERENCES research_people(id) ON DELETE SET NULL
- FK research_applications_parent_person_id_fkey: FOREIGN KEY (parent_person_id) REFERENCES research_people(id) ON DELETE SET NULL
- FK research_applications_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE SET NULL
- FK research_applications_student_person_id_fkey: FOREIGN KEY (student_person_id) REFERENCES research_people(id) ON DELETE SET NULL
- PK research_applications_pkey: PRIMARY KEY (id)
- UQ research_applications_application_ref_key: UNIQUE (application_ref)
- idx research_applications_student_idx USING btree (student_person_id)
- idx research_applications_school_idx USING btree (school_id)

### research_audit_blank_city
school_id uuid?, n bigint?

### research_audit_city_x
co text?, st text?, ci text?, n bigint?, city_states bigint?, one_state text?, ci_is_state boolean?, ci_is_country boolean?, ci_other_country_only boolean?

### research_audit_cityref
country text?, lname text?, n_states bigint?, one_state text?
- idx research_audit_cityref_country_lname_idx USING btree (country, lname)
- idx research_audit_cityref_lname_idx USING btree (lname)

### research_audit_cs
co text?, st text?, n bigint?, no_region bigint?, no_city bigint?

### research_audit_csc
co text?, st text?, ci text?, n bigint?
- idx research_audit_csc_co_lower_idx USING btree (co, lower(ci))

### research_audit_school_city
school_id uuid?, lc text?, tot numeric?
- idx research_audit_school_city_school_id_idx USING btree (school_id)

### research_autopilot
kind text, enabled boolean=true, cadence_minutes integer=30, paused_reason text?, paused_at timestamp with time zone?, last_run_at timestamp with time zone?, last_result text?, runs integer=0, failures integer=0
- PK research_autopilot_pkey: PRIMARY KEY (kind)

### research_b2c_contacts
id uuid=gen_random_uuid(), kind text='parent'::text, full_name text, email text, phone text=''::text, city text=''::text, country text=''::text, school text=''::text, grade text=''::text, source text='webinar'::text, parent_id uuid?, consent boolean=false, notes text?, created_by uuid?, created_at timestamp with time zone=now()
- CK research_b2c_contacts_kind_check: CHECK ((kind = ANY (ARRAY['parent'::text, 'student'::text])))
- FK research_b2c_contacts_parent_id_fkey: FOREIGN KEY (parent_id) REFERENCES research_b2c_contacts(id) ON DELETE SET NULL
- FK research_b2c_creator_fk: FOREIGN KEY (created_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_b2c_contacts_pkey: PRIMARY KEY (id)
- idx research_b2c_email_key UNIQUE USING btree (lower(email), kind)
- idx research_b2c_parent_idx USING btree (parent_id)

### research_b2c_people
id uuid=gen_random_uuid(), kind text='student'::text, name text=''::text, email text=''::text, phone text=''::text, city text=''::text, country text=''::text, school text=''::text, grade text=''::text, programme text=''::text, form_name text=''::text, source text='masterclass'::text, parent_id uuid?, link_confidence text='unlinked'::text, notes text?, created_at timestamp with time zone=now(), state text=''::text, program_type text=''::text, program_year text=''::text, master_sheet text=''::text, status text=''::text, start_month text=''::text, group_name text=''::text, parent_name text=''::text, parent_phone text=''::text, parent_email text=''::text, counsellor_name text=''::text, counsellor_email text=''::text, payment_status text=''::text, source_sheet text=''::text, upload_id uuid?, upload_tab_id uuid?, source_row_number integer?, source_file text=''::text, first_seen_at timestamp with time zone=now(), last_seen_at timestamp with time zone=now(), last_seen_upload_id uuid?, times_seen integer=1, custom jsonb='{}'::jsonb, deleted_at timestamp with time zone?, deleted_by uuid?
- CK research_b2c_people_kind_check: CHECK ((kind = ANY (ARRAY['student'::text, 'parent'::text, 'counselor'::text, 'other'::text])))
- CK research_b2c_people_link_confidence_check: CHECK ((link_confidence = ANY (ARRAY['unlinked'::text, 'inferred'::text, 'confirmed'::text])))
- FK research_b2c_people_deleted_by_fkey: FOREIGN KEY (deleted_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_b2c_people_last_seen_upload_id_fkey: FOREIGN KEY (last_seen_upload_id) REFERENCES research_uploads(id) ON DELETE SET NULL
- FK research_b2c_people_parent_id_fkey: FOREIGN KEY (parent_id) REFERENCES research_b2c_people(id) ON DELETE SET NULL
- FK research_b2c_people_upload_id_fkey: FOREIGN KEY (upload_id) REFERENCES research_uploads(id) ON DELETE SET NULL
- FK research_b2c_people_upload_tab_id_fkey: FOREIGN KEY (upload_tab_id) REFERENCES research_upload_tabs(id) ON DELETE SET NULL
- PK research_b2c_people_pkey: PRIMARY KEY (id)
- idx research_b2c_people_email_idx USING btree (lower(email))
- idx research_b2c_people_parent_idx USING btree (parent_id)
- idx research_b2c_people_upload_idx USING btree (upload_id)
- idx research_b2c_people_deleted_idx USING btree (deleted_at)
- idx research_b2c_people_custom_gin USING gin (custom)

### research_backfill_cursor
id integer, pos integer=0, batch uuid, actor text, totals jsonb='{}'::jsonb
- PK research_backfill_cursor_pkey: PRIMARY KEY (id)

### research_campaign_contacts
id uuid=gen_random_uuid(), campaign_id uuid, contact_id uuid, status text='queued'::text, skip_reason text?, sent_at timestamp with time zone?, replied_at timestamp with time zone?, opened_at timestamp with time zone?, clicked_at timestamp with time zone?, bounced_at timestamp with time zone?, unsubscribed_at timestamp with time zone?, followup_stage integer=0
- CK research_campaign_contacts_status_check: CHECK ((status = ANY (ARRAY['queued'::text, 'sent'::text, 'replied'::text, 'booked'::text, 'skipped'::text])))
- FK research_campaign_contacts_campaign_id_fkey: FOREIGN KEY (campaign_id) REFERENCES research_campaigns(id) ON DELETE CASCADE
- FK research_campaign_contacts_contact_id_fkey: FOREIGN KEY (contact_id) REFERENCES research_contacts(id) ON DELETE CASCADE
- PK research_campaign_contacts_pkey: PRIMARY KEY (id)
- UQ research_campaign_contacts_campaign_id_contact_id_key: UNIQUE (campaign_id, contact_id)
- idx research_campaign_contacts_contact_idx USING btree (contact_id)

### research_campaign_reports
id uuid=gen_random_uuid(), campaign_id uuid, reported_at timestamp with time zone=now(), reported_by uuid?, sent integer=0, opens integer=0, clicks integer=0, replies integer=0, bounces integer=0, unsubscribes integer=0, source text='manual_import'::text, note text=''::text
- FK research_campaign_reports_campaign_id_fkey: FOREIGN KEY (campaign_id) REFERENCES research_campaigns(id) ON DELETE CASCADE
- FK research_campaign_reports_reported_by_fkey: FOREIGN KEY (reported_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_campaign_reports_pkey: PRIMARY KEY (id)
- idx research_campaign_reports_campaign_idx USING btree (campaign_id, reported_at DESC)

### research_campaigns
id uuid=gen_random_uuid(), name text, programme text=''::text, dataset_id uuid?, calendly_url text?, email_subject text?, email_body text?, followup_body text?, status text='draft'::text, done_flag boolean=false, created_by uuid?, created_at timestamp with time zone=now(), send_date date?, send_timezone text='Asia/Kolkata'::text, window_start text='09:30'::text, window_end text='18:00'::text, daily_cap integer=300, skip_weekends boolean=true, skip_holidays text[]='{}'::text[], followup_gaps integer[]='{3,7,14}'::integer[], thread_mode text='same_thread'::text, variant_b_subject text?, variant_b_body text?, ab_split integer=0, trial_schools integer=0
- CK research_campaigns_status_check: CHECK ((status = ANY (ARRAY['draft'::text, 'live'::text, 'paused'::text, 'done'::text])))
- CK research_campaigns_thread_mode_check: CHECK ((thread_mode = ANY (ARRAY['same_thread'::text, 'new_thread'::text])))
- FK research_campaigns_created_by_fkey: FOREIGN KEY (created_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_campaigns_dataset_id_fkey: FOREIGN KEY (dataset_id) REFERENCES research_datasets(id) ON DELETE SET NULL
- PK research_campaigns_pkey: PRIMARY KEY (id)

### research_cell_flags
id uuid=gen_random_uuid(), contact_id uuid, column_name text, kind text, current_value text=''::text, suggested_value text?, suggested_column text?, reason text=''::text, status text='open'::text, raised_by text='agent'::text, resolved_by uuid?, resolved_at timestamp with time zone?, created_at timestamp with time zone=now()
- CK research_cell_flags_kind_check: CHECK ((kind = ANY (ARRAY['wrong_column'::text, 'missing'::text, 'malformed'::text, 'suspect_duplicate'::text])))
- CK research_cell_flags_status_check: CHECK ((status = ANY (ARRAY['open'::text, 'accepted'::text, 'rejected'::text])))
- FK research_cell_flags_contact_fk: FOREIGN KEY (contact_id) REFERENCES research_contacts(id) ON DELETE CASCADE
- FK research_cell_flags_resolver_fk: FOREIGN KEY (resolved_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_cell_flags_pkey: PRIMARY KEY (id)
- idx research_cell_flags_open_key UNIQUE USING btree (contact_id, column_name, kind) WHERE (status = 'open'::text)
- idx research_cell_flags_status_idx USING btree (status, created_at DESC)

### research_city_targets
domain text, school_key text?, school text?, country text?, contacts_no_city integer?, total integer?, share numeric?, checked_at timestamp with time zone?, city text?, state text?, method text?, evidence text?, url text?, snippet text?, read_by text?, applied_at timestamp with time zone?
- PK research_city_targets_pkey: PRIMARY KEY (domain)
- idx research_city_targets_todo_idx USING btree (contacts_no_city DESC) WHERE (checked_at IS NULL)

### research_claims
id uuid=gen_random_uuid(), location_id uuid, researcher_id uuid, purpose text='mapping'::text, note text?, claimed_at timestamp with time zone=now(), expires_at timestamp with time zone=(now() + '3 days'::interval), released_at timestamp with time zone?
- CK research_claims_purpose_check: CHECK ((purpose = ANY (ARRAY['mapping'::text, 'redo'::text, 'verification'::text])))
- FK research_claims_location_fk: FOREIGN KEY (location_id) REFERENCES research_locations(id) ON DELETE CASCADE
- FK research_claims_researcher_fk: FOREIGN KEY (researcher_id) REFERENCES research_staff_profiles(id) ON DELETE CASCADE
- PK research_claims_pkey: PRIMARY KEY (id)
- idx research_claims_live_key UNIQUE USING btree (location_id) WHERE (released_at IS NULL)
- idx research_claims_researcher_idx USING btree (researcher_id) WHERE (released_at IS NULL)

### research_cleaning_runs
id uuid=gen_random_uuid(), ran_at timestamp with time zone=now(), actor text=''::text, result jsonb='{}'::jsonb
- PK research_cleaning_runs_pkey: PRIMARY KEY (id)

### research_contact_facets
id uuid, deleted_at timestamp with time zone?, category text?, person_type text?, school text?, country text?, state text?, city text?, email text?, subject text?, seniority text?, sop_priority text?, grade_band text?, school_type text?, school_type_confidence text?, grade_band_confidence text?, sop_level text?, curriculum_row text?, geo_region text?, school_key text?=lower(btrim(COALESCE(school, ''::text))), trust_state text?, trust_reasons text[]='{}'::text[]
- PK research_contact_facets_pkey: PRIMARY KEY (id)
- idx research_contact_facets_country_idx USING btree (category, lower(btrim(country))) WHERE ((deleted_at IS NULL) AND (person_type = 'staff'::text))
- idx research_contact_facets_state_idx USING btree (category, lower(btrim(state))) WHERE ((deleted_at IS NULL) AND (person_type = 'staff'::text))
- idx research_contact_facets_city_idx USING btree (category, lower(btrim(city))) WHERE ((deleted_at IS NULL) AND (person_type = 'staff'::text))
- idx research_contact_facets_region_idx USING btree (category, geo_region) WHERE ((deleted_at IS NULL) AND (person_type = 'staff'::text))
- idx research_contact_facets_school_idx USING btree (lower(btrim(school))) WHERE (deleted_at IS NULL)

### research_contact_health
contact_id uuid, tier text, reasons text[]='{}'::text[], ai_unverified boolean=false, email_status text?, computed_at timestamp with time zone=now()
- PK research_contact_health_pkey: PRIMARY KEY (contact_id)
- idx research_contact_health_tier_idx USING btree (tier)

### research_contact_segments
id uuid=gen_random_uuid(), segment_id uuid, contact_id uuid, matched_on text='school'::text, added_by uuid?, created_at timestamp with time zone=now()
- FK research_contact_segments_added_by_fkey: FOREIGN KEY (added_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_contact_segments_contact_id_fkey: FOREIGN KEY (contact_id) REFERENCES research_contacts(id) ON DELETE CASCADE
- FK research_contact_segments_segment_id_fkey: FOREIGN KEY (segment_id) REFERENCES research_segments(id) ON DELETE CASCADE
- PK research_contact_segments_pkey: PRIMARY KEY (id)
- idx research_contact_segments_key UNIQUE USING btree (segment_id, contact_id)
- idx research_contact_segments_contact_idx USING btree (contact_id)

### research_contacts
id uuid=gen_random_uuid(), serial bigint=nextval('research_contacts_serial_seq'::regclass), dataset_id uuid, school text, city text=''::text, state text=''::text, country text=''::text, contact_name text, designation text=''::text, email text, domain text=''::text, source text='Manual'::text, group_id uuid?, validated boolean=false, authenticity text='unknown'::text, created_by uuid?, created_at timestamp with time zone=now(), verify_status text='unverified'::text, verified_at timestamp with time zone?, verified_by uuid?, verify_note text?, redo_cycle integer=1, auto_verdict text?, auto_confidence integer?, auto_reason text?, auto_source text?, auto_checked_at timestamp with time zone?, auto_reviewed_at timestamp with time zone?, source_kind text='unknown'::text, legal_state text='allowed'::text, legal_reason text=''::text, consent_note text?, consent_by uuid?, consent_at timestamp with time zone?, category text='school'::text, curriculum text=''::text, subject text=''::text, phone text=''::text, linkedin_url text=''::text, grade text=''::text, upload_id uuid?, upload_tab_id uuid?, source_row_number integer?, source_file text=''::text, first_seen_at timestamp with time zone=now(), last_seen_at timestamp with time zone=now(), last_seen_upload_id uuid?, times_seen integer=1, keep_flag boolean?, website text=''::text, curriculum_raw text=''::text, region text=''::text, custom jsonb='{}'::jsonb, deleted_at timestamp with time zone?, deleted_by uuid?, person_type text='staff'::text, sop_role text=''::text, sop_priority text=''::text, sop_category text=''::text, sop_level text=''::text, sop_signal text=''::text, school_type text='unknown'::text, school_type_signal text=''::text, school_type_confidence text='low'::text, grade_band text='unknown'::text, grade_band_signal text=''::text, grade_band_confidence text='low'::text, classifier_version integer=0, address text=''::text, fee_standard text=''::text, fee_maximum text=''::text, accreditation text=''::text, association text=''::text, direct_contact_link text=''::text, resources text=''::text, verify_state text='unverified'::text, verified_title text=''::text, reverify_due date?, source_detail text=''::text, cross_checked boolean=false, city_derived boolean=false /*True when the portal worked the city out of the school name / web address (SOP §5) instead of a researcher entering it.*/, essentials smallint?=((((((((
CASE
    WHEN (COALESCE(school, ''::text) <> ''::text) THEN 1
    ELSE 0
END +
CASE
    WHEN (COALESCE(city, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(country, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(contact_name, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(designation, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN ((COALESCE(email, ''::text) <> ''::text) OR (COALESCE(phone, ''::text) <> ''::text)) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(website, ''::text) <> ''::text) THEN 1
    ELSE 0
END) * 100) / 7) /*SOP §11.6 tier 1 — percentage of the seven columns a row needs before outreach: school, city, country, contact name, designation, email or phone, website. The full 15-column §5 score stays in `completeness`.*/, verified_note text=''::text /*SOP §7 — how the role was confirmed (single profile check, or the batch note from a bulk confirmation).*/, sop_priority_rank integer?=
CASE sop_priority
    WHEN 'high'::text THEN 0
    WHEN 'medium'::text THEN 1
    WHEN 'do_not_contact'::text THEN 3
    ELSE 2
END, completeness smallint?=((((((((((((((((((((
CASE
    WHEN (COALESCE(school, ''::text) <> ''::text) THEN 1
    ELSE 0
END +
CASE
    WHEN (COALESCE(category, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(website, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(address, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(city, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(state, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(country, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(grade, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(fee_standard, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(fee_maximum, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(accreditation, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(association, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(contact_name, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(designation, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(email, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(phone, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(linkedin_url, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(direct_contact_link, ''::text) <> ''::text) THEN 1
    ELSE 0
END) +
CASE
    WHEN (COALESCE(resources, ''::text) <> ''::text) THEN 1
    ELSE 0
END) * 100) / 19) /*SOP §5 tier 2 — percentage of the nineteen columns §5 asks for on every row. Reported, never a send blocker; the gate is `essentials`.*/, touched_action text?, touched_at timestamp with time zone?, touched_by text?, origin jsonb='{}'::jsonb, person_id uuid?, school_id uuid?, trust_state text?, trust_reasons text[]='{}'::text[], trust_tried jsonb='[]'::jsonb, trust_checked_at timestamp with time zone?, role_seen_at timestamp with time zone?, role_evidence text=''::text, search_checked_at timestamp with time zone?, fee_currency text=''::text, researcher_ids uuid[]='{}'::uuid[], researcher_names text[]='{}'::text[], researched_on date?, researched_until date?, research_source text=''::text, generic_contact boolean=false, geo_region text?, seniority text?, curriculum_row text?=
CASE curriculum
    WHEN 'IB'::text THEN 'IB'::text
    WHEN 'Cambridge/IGCSE'::text THEN 'Cambridge'::text
    WHEN 'American/Common Core'::text THEN 'American'::text
    WHEN 'German'::text THEN 'German'::text
    WHEN 'French'::text THEN 'French'::text
    WHEN 'CBSE'::text THEN 'National Curriculum'::text
    WHEN 'ICSE'::text THEN 'National Curriculum'::text
    WHEN 'National/Local'::text THEN 'National Curriculum'::text
    WHEN 'Other'::text THEN 'Other International'::text
    ELSE 'Not recorded'::text
END, research_tool text?, research_tool_note text=''::text, research_tool_set_by text=''::text
- CK research_contacts_authenticity_check: CHECK ((authenticity = ANY (ARRAY['school_domain'::text, 'generic'::text, 'unknown'::text])))
- CK research_contacts_auto_verdict_chk: CHECK (((auto_verdict IS NULL) OR (auto_verdict = ANY (ARRAY['confirmed'::text, 'left'::text, 'recheck'::text]))))
- CK research_contacts_category_chk: CHECK ((category = ANY (ARRAY['school'::text, 'university'::text, 'corporate'::text])))
- CK research_contacts_curriculum_chk: CHECK ((curriculum = ANY (ARRAY[''::text, 'IB'::text, 'Cambridge/IGCSE'::text, 'CBSE'::text, 'ICSE'::text, 'American/Common Core'::text, 'German'::text, 'French'::text, 'National/Local'::text, 'Other'::text])))
- CK research_contacts_grade_band_chk: CHECK ((grade_band = ANY (ARRAY['unknown'::text, 'elementary'::text, 'middle'::text, 'high'::text, 'k12'::text])))
- CK research_contacts_legal_state_chk: CHECK ((legal_state = ANY (ARRAY['allowed'::text, 'consent_required'::text, 'restricted'::text, 'released'::text])))
- CK research_contacts_person_type_chk: CHECK ((person_type = ANY (ARRAY['staff'::text, 'parent'::text, 'student'::text])))
- CK research_contacts_school_type_chk: CHECK ((school_type = ANY (ARRAY['unknown'::text, 'private'::text, 'public'::text])))
- CK research_contacts_sop_category_chk: CHECK ((sop_category = ANY (ARRAY[''::text, 'leadership'::text, 'counseling'::text, 'academic'::text, 'stem_subject'::text, 'innovation'::text, 'program_coordinator'::text, 'activities'::text, 'external'::text, 'other'::text, 'excluded'::text])))
- CK research_contacts_sop_confidence_chk: CHECK (((school_type_confidence = ANY (ARRAY['high'::text, 'medium'::text, 'low'::text])) AND (grade_band_confidence = ANY (ARRAY['high'::text, 'medium'::text, 'low'::text]))))
- CK research_contacts_sop_level_chk: CHECK ((sop_level = ANY (ARRAY[''::text, 'elementary'::text, 'middle'::text, 'high'::text, 'whole_school'::text])))
- CK research_contacts_sop_priority_chk: CHECK ((sop_priority = ANY (ARRAY[''::text, 'high'::text, 'medium'::text, 'do_not_contact'::text])))
- CK research_contacts_source_check: CHECK ((source = ANY (ARRAY['Apollo'::text, 'Manual'::text])))
- CK research_contacts_source_kind_chk: CHECK ((source_kind = ANY (ARRAY['public_website'::text, 'form'::text, 'master_class'::text, 'webinar'::text, 'alumni'::text, 'vendor'::text, 'manual'::text, 'unknown'::text])))
- CK research_contacts_subject_chk: CHECK ((subject = ANY (ARRAY[''::text, 'Science'::text, 'Math'::text, 'English'::text, 'Business/Economics'::text, 'Computer Science/STEM'::text, 'Social Studies'::text, 'Languages'::text, 'Arts'::text, 'Administration/Leadership'::text, 'Other'::text])))
- CK research_contacts_trust_state_check: CHECK ((trust_state = ANY (ARRAY['validated'::text, 'quarantined'::text])))
- CK research_contacts_verify_state_chk: CHECK ((verify_state = ANY (ARRAY['unverified'::text, 'verified'::text, 'mismatch'::text, 'not_found'::text])))
- CK research_contacts_verify_status_chk: CHECK ((verify_status = ANY (ARRAY['unverified'::text, 'confirmed'::text, 'left'::text, 'new'::text])))
- FK research_contacts_consent_by_fkey: FOREIGN KEY (consent_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_contacts_created_by_fkey: FOREIGN KEY (created_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_contacts_dataset_id_fkey: FOREIGN KEY (dataset_id) REFERENCES research_datasets(id) ON DELETE CASCADE
- FK research_contacts_deleted_by_fkey: FOREIGN KEY (deleted_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_contacts_group_id_fkey: FOREIGN KEY (group_id) REFERENCES research_groups(id) ON DELETE SET NULL
- FK research_contacts_last_seen_upload_id_fkey: FOREIGN KEY (last_seen_upload_id) REFERENCES research_uploads(id) ON DELETE SET NULL
- FK research_contacts_person_id_fkey: FOREIGN KEY (person_id) REFERENCES research_people(id) ON DELETE SET NULL
- FK research_contacts_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE SET NULL
- FK research_contacts_upload_id_fkey: FOREIGN KEY (upload_id) REFERENCES research_uploads(id) ON DELETE SET NULL
- FK research_contacts_upload_tab_id_fkey: FOREIGN KEY (upload_tab_id) REFERENCES research_upload_tabs(id) ON DELETE SET NULL
- FK research_contacts_verified_by_fkey: FOREIGN KEY (verified_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_contacts_pkey: PRIMARY KEY (id)
- idx research_contacts_email_key UNIQUE USING btree (lower(email))
- idx research_contacts_dataset_idx USING btree (dataset_id)
- idx research_contacts_state_idx USING btree (lower(state))
- idx research_contacts_verify_idx USING btree (verify_status)
- idx research_contacts_auto_verdict_idx USING btree (auto_verdict)
- idx research_contacts_legal_state_idx USING btree (legal_state) WHERE (legal_state <> 'allowed'::text)
- idx research_contacts_category_idx USING btree (category)
- idx research_contacts_upload_idx USING btree (upload_id)
- idx research_contacts_keep_idx USING btree (keep_flag)
- idx research_contacts_deleted_idx USING btree (deleted_at)
- idx research_contacts_last_seen_idx USING btree (last_seen_at DESC)
- idx research_contacts_region_idx USING btree (region)
- idx research_contacts_custom_gin USING gin (custom)
- idx research_contacts_email_plain_idx USING btree (email)
- idx research_contacts_school_plain_idx USING btree (school)
- idx research_contacts_person_type_idx USING btree (person_type) WHERE (deleted_at IS NULL)
- idx research_contacts_verify_state_idx USING btree (verify_state)
- idx research_contacts_reverify_due_idx USING btree (reverify_due)
- idx research_contacts_city_derived_idx USING btree (city_derived) WHERE city_derived
- idx research_contacts_essentials_idx USING btree (essentials)
- idx research_contacts_sop_priority_rank_idx USING btree (sop_priority_rank) WHERE (deleted_at IS NULL)
- idx research_contacts_completeness_idx USING btree (completeness)
- idx research_contacts_touched_idx USING btree (touched_at DESC) WHERE (touched_action IS NOT NULL)
- idx research_contacts_person_idx USING btree (person_id)
- idx research_contacts_school_id_idx USING btree (school_id)
- idx research_contacts_trust_state_idx USING btree (trust_state)
- idx research_contacts_trust_checked_idx USING btree (trust_checked_at NULLS FIRST)
- idx research_contacts_trust_reasons_idx USING gin (trust_reasons)
- idx research_contacts_search_checked_idx USING btree (search_checked_at NULLS FIRST)
- idx research_contacts_researcher_ids_idx USING gin (researcher_ids)
- idx research_contacts_researched_on_idx USING btree (researched_on)
- idx research_contacts_generic_idx USING btree (generic_contact) WHERE generic_contact
- idx research_contacts_school_realname_key UNIQUE USING btree (lower(school), lower(contact_name)) WHERE ((deleted_at IS NULL) AND (btrim(contact_name) <> ''::text))
- idx research_contacts_reach_idx USING btree (category, person_type, country, state) WHERE (deleted_at IS NULL)
- idx research_contacts_school_norm_idx USING btree (lower(btrim(school))) WHERE (deleted_at IS NULL)
- idx research_contacts_reach_country_idx USING btree (category, lower(btrim(country))) WHERE ((deleted_at IS NULL) AND (person_type = 'staff'::text))
- idx research_contacts_reach_state_idx USING btree (category, lower(btrim(state))) WHERE ((deleted_at IS NULL) AND (person_type = 'staff'::text))
- idx research_contacts_reach_city_idx USING btree (category, lower(btrim(city))) WHERE ((deleted_at IS NULL) AND (person_type = 'staff'::text))
- idx research_contacts_reach_region_idx USING btree (category, geo_region) WHERE ((deleted_at IS NULL) AND (person_type = 'staff'::text))
- idx research_contacts_school_id_live_idx USING btree (school_id) WHERE (deleted_at IS NULL)
- idx research_contacts_trust_school_idx USING btree (trust_state, school) WHERE (deleted_at IS NULL)
- idx research_contacts_serial_live_idx USING btree (serial, created_at DESC) WHERE (deleted_at IS NULL)
- idx research_contacts_created_at_idx USING btree (created_at DESC)
- idx research_contacts_created_by_at_idx USING btree (created_by, created_at DESC)
- idx research_contacts_validated_idx USING btree (validated) WHERE (validated = true)
- idx research_contacts_school_contact_trgm_live_idx USING gin (school gin_trgm_ops, contact_name gin_trgm_ops) WHERE (deleted_at IS NULL)
- idx research_contacts_research_tool_idx USING btree (research_tool) WHERE (research_tool IS NOT NULL)
- idx research_contacts_researcher_ids_gin USING gin (researcher_ids)

### research_contrib_cursor
id integer=1, pos integer=0, last_cycle_at timestamp with time zone?
- CK research_contrib_cursor_id_check: CHECK ((id = 1))
- PK research_contrib_cursor_pkey: PRIMARY KEY (id)

### research_data_health
id bigint=nextval('research_data_health_id_seq'::regclass), computed_at timestamp with time zone=now(), summary jsonb
- PK research_data_health_pkey: PRIMARY KEY (id)

### research_datasets
id uuid=gen_random_uuid(), code text, location_id uuid, built_by uuid?, built_on date=CURRENT_DATE, redo_of uuid?, status text='active'::text, campaign_done boolean=false, notes text?, created_at timestamp with time zone=now(), category text='school'::text
- CK research_datasets_category_chk: CHECK ((category = ANY (ARRAY['school'::text, 'university'::text, 'corporate'::text])))
- CK research_datasets_status_check: CHECK ((status = ANY (ARRAY['active'::text, 'superseded'::text, 'draft'::text])))
- FK research_datasets_built_by_fkey: FOREIGN KEY (built_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_datasets_location_id_fkey: FOREIGN KEY (location_id) REFERENCES research_locations(id) ON DELETE CASCADE
- FK research_datasets_redo_of_fkey: FOREIGN KEY (redo_of) REFERENCES research_datasets(id) ON DELETE SET NULL
- PK research_datasets_pkey: PRIMARY KEY (id)
- UQ research_datasets_code_key: UNIQUE (code)

### research_derived_map
col text, k text, v text
- PK research_derived_map_pkey: PRIMARY KEY (col, k)

### research_designation_proposals
id uuid=gen_random_uuid(), designation text, suggested_priority text='medium'::text, suggested_level text='whole_school'::text, note text=''::text, rows_affected integer=0, status text='pending'::text, proposed_by uuid?, decided_by uuid?, decided_at timestamp with time zone?, decision_note text=''::text, created_at timestamp with time zone=now(), suggested_category text='other'::text
- CK research_designation_proposals_category_chk: CHECK ((suggested_category = ANY (ARRAY['leadership'::text, 'counseling'::text, 'academic'::text, 'stem_subject'::text, 'innovation'::text, 'program_coordinator'::text, 'activities'::text, 'external'::text, 'other'::text, 'excluded'::text])))
- CK research_designation_proposals_status_check: CHECK ((status = ANY (ARRAY['pending'::text, 'approved'::text, 'rejected'::text])))
- CK research_designation_proposals_suggested_level_check: CHECK ((suggested_level = ANY (ARRAY['elementary'::text, 'middle'::text, 'high'::text, 'whole_school'::text])))
- CK research_designation_proposals_suggested_priority_check: CHECK ((suggested_priority = ANY (ARRAY['high'::text, 'medium'::text, 'do_not_contact'::text])))
- FK research_designation_proposals_decided_by_fkey: FOREIGN KEY (decided_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_designation_proposals_proposed_by_fkey: FOREIGN KEY (proposed_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_designation_proposals_pkey: PRIMARY KEY (id)
- idx research_designation_proposals_key UNIQUE USING btree (lower(designation)) WHERE (status = 'pending'::text)
- idx research_designation_proposals_approved_key UNIQUE USING btree (lower(designation)) WHERE (status = 'approved'::text)

### research_district_directories
id uuid=gen_random_uuid(), domain text, school_name text, school_url text=''::text, source_url text, fetched_at timestamp with time zone=now(), fetched_by text=''::text
- PK research_district_directories_pkey: PRIMARY KEY (id)
- UQ research_district_directories_domain_school_name_key: UNIQUE (domain, school_name)
- idx research_district_directories_domain USING btree (domain)

### research_district_scans
domain text, schools integer=0, leads integer=0, matched integer=0, scanned_at timestamp with time zone=now()
- PK research_district_scans_pkey: PRIMARY KEY (domain)

### research_domain_checks
domain text, mx_ok boolean=false, mx_host text=''::text, disposable boolean=false, note text=''::text, checked_at timestamp with time zone=now()
- PK research_domain_checks_pkey: PRIMARY KEY (domain)

### research_domain_known
domain text, n integer, refreshed_at timestamp with time zone=now()
- PK research_domain_known_pkey: PRIMARY KEY (domain)

### research_domain_names
domain text, school text, n integer, ids uuid[]='{}'::uuid[], refreshed_at timestamp with time zone=now()
- PK research_domain_names_pkey: PRIMARY KEY (domain, school)

### research_domain_sites
domain text, url text=''::text, title text=''::text, kind text='unknown'::text, school_name text=''::text, checked_at timestamp with time zone=now(), ai_kind text=''::text, ai_name text=''::text, ai_city text=''::text, ai_country text=''::text, ai_quote text=''::text, ai_why text=''::text, ai_checked_at timestamp with time zone?
- CK research_domain_sites_kind_check: CHECK ((kind = ANY (ARRAY['school'::text, 'district'::text, 'unknown'::text, 'unreachable'::text])))
- PK research_domain_sites_pkey: PRIMARY KEY (domain)

### research_duplicate_log
id uuid=gen_random_uuid(), attempted_email text?, school text?, contact_name text?, reason text, existing_contact_id uuid?, dataset_id uuid?, attempted_by uuid?, created_at timestamp with time zone=now(), attempted_fields jsonb='{}'::jsonb, decision text='pending'::text, decided_by text?, decided_at timestamp with time zone?, checked_at timestamp with time zone?, quarantined_at timestamp with time zone?, quarantine_note text=''::text, auto_note text=''::text, merged_fields text[]='{}'::text[]
- CK research_duplicate_log_decision_chk: CHECK ((decision = ANY (ARRAY['pending'::text, 'keep_existing'::text, 'update_existing'::text, 'file_separate'::text])))
- FK research_duplicate_log_attempted_by_fkey: FOREIGN KEY (attempted_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_duplicate_log_dataset_id_fkey: FOREIGN KEY (dataset_id) REFERENCES research_datasets(id) ON DELETE SET NULL
- FK research_duplicate_log_existing_contact_id_fkey: FOREIGN KEY (existing_contact_id) REFERENCES research_contacts(id) ON DELETE SET NULL
- PK research_duplicate_log_pkey: PRIMARY KEY (id)
- idx research_duplicate_log_created_idx USING btree (created_at DESC)
- idx research_duplicate_log_decision_idx USING btree (decision, created_at DESC)
- idx research_duplicate_log_checked_idx USING btree (decision, checked_at NULLS FIRST, created_at)
- idx research_duplicate_log_decided_idx USING btree (decided_at DESC) WHERE (decision <> 'pending'::text)

### research_email_check_requests
email text, label text=''::text, requested_by text=''::text, requested_at timestamp with time zone=now()
- PK research_email_check_requests_pkey: PRIMARY KEY (email)

### research_email_check_runs
id uuid=gen_random_uuid(), started_at timestamp with time zone=now(), finished_at timestamp with time zone?, trigger text=''::text, checked integer=0, ready integer=0, risky integer=0, do_not_send integer=0, credits_left integer?, outcome text=''::text, note text=''::text
- PK research_email_check_runs_pkey: PRIMARY KEY (id)
- idx research_email_check_runs_started_idx USING btree (started_at DESC)

### research_email_check_state
id integer=1, daily_limit integer=2000, paused_reason text=''::text, paused_at timestamp with time zone?, lease_holder text=''::text, lease_until timestamp with time zone?, updated_by text=''::text, updated_at timestamp with time zone=now()
- CK research_email_check_state_id_check: CHECK ((id = 1))
- PK research_email_check_state_pkey: PRIMARY KEY (id)

### research_email_checks
email text, status text, zb_status text=''::text, zb_sub_status text=''::text, free_email boolean?, mx_found text=''::text, did_you_mean text=''::text, credits_used integer=1, recheck_requested boolean=false, checked_at timestamp with time zone=now(), checked_by text=''::text, raw jsonb='{}'::jsonb
- CK research_email_checks_status_check: CHECK ((status = ANY (ARRAY['ready'::text, 'risky'::text, 'do_not_send'::text])))
- PK research_email_checks_pkey: PRIMARY KEY (email)
- idx research_email_checks_status_idx USING btree (status)
- idx research_email_checks_checked_idx USING btree (checked_at DESC)

### research_email_health
id uuid=gen_random_uuid(), contact_id uuid?, email text, state text='unknown'::text, reason text=''::text, bounce_kind text?, bounce_count integer=0, last_bounce_at timestamp with time zone?, verified_at timestamp with time zone?, checked_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), mx_ok boolean?, mx_host text=''::text, confidence integer=0, verified_source text=''::text
- CK research_email_health_state_check: CHECK ((state = ANY (ARRAY['valid'::text, 'risky'::text, 'invalid'::text, 'quarantined'::text, 'unknown'::text])))
- FK research_email_health_contact_id_fkey: FOREIGN KEY (contact_id) REFERENCES research_contacts(id) ON DELETE CASCADE
- PK research_email_health_pkey: PRIMARY KEY (id)
- idx research_email_health_email_key UNIQUE USING btree (lower(email))
- idx research_email_health_state_idx USING btree (state)

### research_email_templates
id uuid=gen_random_uuid(), name text, kind text='first_touch'::text, subject text=''::text, body text=''::text, programme text=''::text, created_by uuid?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), audience text='school_leader'::text, tone text='formal'::text, channel text='email'::text, notes text=''::text, source text='manual'::text, archived boolean=false, usage_count integer=0, last_used_at timestamp with time zone?
- CK research_email_templates_kind_check: CHECK ((kind = ANY (ARRAY['first_touch'::text, 'followup'::text, 'reminder'::text, 'meeting_invite'::text, 'meeting_confirmation'::text, 'post_meeting'::text, 'proposal'::text, 'reengagement'::text, 'counsellor_intro'::text, 'principal_intro'::text, 'parent_b2c'::text, 'thank_you'::text, 'escalation'::text, 'other'::text])))
- CK research_email_templates_source_check: CHECK ((source = ANY (ARRAY['manual'::text, 'upload'::text, 'ai'::text])))
- FK research_email_templates_created_by_fkey: FOREIGN KEY (created_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_email_templates_pkey: PRIMARY KEY (id)
- idx research_email_templates_kind_idx USING btree (kind)
- idx research_email_templates_updated_idx USING btree (archived, updated_at DESC)

### research_events
id uuid=gen_random_uuid(), name text, organiser text=''::text, kind text='conference'::text, city text=''::text, country text=''::text, association text=''::text, starts_on date?, ends_on date?, link text=''::text, notes text=''::text, status text='spotted'::text, spotted_by uuid?, created_at timestamp with time zone=now()
- CK research_events_kind_check: CHECK ((kind = ANY (ARRAY['conference'::text, 'counselor_fair'::text, 'association_summit'::text, 'other'::text])))
- CK research_events_status_check: CHECK ((status = ANY (ARRAY['spotted'::text, 'shortlisted'::text, 'attending'::text, 'passed'::text])))
- FK research_events_spotted_by_fkey: FOREIGN KEY (spotted_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_events_pkey: PRIMARY KEY (id)
- idx research_events_starts_idx USING btree (starts_on)

### research_exports
id uuid=gen_random_uuid(), at timestamp with time zone=now(), who text, folder_id uuid?, filters jsonb='{}'::jsonb, row_count integer=0
- PK research_exports_pkey: PRIMARY KEY (id)

### research_fact_fill_cols
col text, field text, planned integer?, done_at timestamp with time zone?
- PK research_fact_fill_cols_pkey: PRIMARY KEY (col)

### research_fact_fill_plan
id bigint=nextval('research_fact_fill_plan_id_seq'::regclass), contact_id uuid, col text, value text, school_id uuid, source text=''::text, done_at timestamp with time zone?
- PK research_fact_fill_plan_pkey: PRIMARY KEY (id)
- UQ research_fact_fill_plan_contact_id_col_key: UNIQUE (contact_id, col)
- idx research_fact_fill_plan_todo_idx USING btree (id) WHERE (done_at IS NULL)

### research_fact_fill_schools
school_id uuid
- PK research_fact_fill_schools_pkey: PRIMARY KEY (school_id)

### research_field_changes
id uuid=gen_random_uuid(), contact_id uuid, field text, old_value text=''::text, new_value text=''::text, action text, source text=''::text, actor text=''::text, batch_id uuid, at timestamp with time zone=now(), reverted_at timestamp with time zone?, reverted_by text?
- PK research_field_changes_pkey: PRIMARY KEY (id)
- idx research_field_changes_batch_idx USING btree (batch_id)
- idx research_field_changes_contact_idx USING btree (contact_id, at DESC)
- idx research_field_changes_at_idx USING btree (at DESC)
- idx research_field_changes_source_at_idx USING btree (source, at DESC)

### research_field_defs
id uuid=gen_random_uuid(), lane text, key text, label text, kind text='text'::text, options jsonb='[]'::jsonb, position integer=0, active boolean=true, created_by uuid?, created_at timestamp with time zone=now()
- CK research_field_defs_kind_chk: CHECK ((kind = ANY (ARRAY['text'::text, 'number'::text, 'date'::text, 'select'::text, 'bool'::text])))
- CK research_field_defs_lane_chk: CHECK ((lane = ANY (ARRAY['contact'::text, 'b2c'::text, 'lead'::text])))
- FK research_field_defs_created_by_fkey: FOREIGN KEY (created_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_field_defs_pkey: PRIMARY KEY (id)
- idx research_field_defs_lane_key_uidx UNIQUE USING btree (lane, key)

### research_fix_cityst
co text?, lc text?, st text?, n bigint?
- idx research_fix_cityst_co_lc_idx USING btree (co, lc)

### research_fix_g22
co text?, lc text?, new_state text?, blank_n numeric?, known_n numeric?, agree_n numeric?, rejected boolean=false

### research_fix_g45
co text?, st text?, lc text?, n bigint?, new_co text?, new_state text?, rejected boolean=false

### research_fix_g45src
co text?, st text?, lc text?, n bigint?

### research_fix_g6
co text?, raw text?, ci text?, n bigint?, new_city text?, to_address text?, kind text?

### research_fix_plan
id bigint=nextval('research_fix_plan_id_seq'::regclass), contact_id uuid, field text, old_value text=''::text, new_value text=''::text, rule text, applied_at timestamp with time zone?, skipped text?
- CK research_fix_plan_field_check: CHECK ((field = ANY (ARRAY['city'::text, 'state'::text, 'country'::text, 'region'::text, 'website'::text, 'phone'::text, 'linkedin_url'::text, 'contact_name'::text, 'email'::text, 'address'::text, 'direct_contact_link'::text])))
- PK research_fix_plan_pkey: PRIMARY KEY (id)
- UQ research_fix_plan_contact_id_field_key: UNIQUE (contact_id, field)
- idx research_fix_plan_todo_idx USING btree (id) WHERE ((applied_at IS NULL) AND (skipped IS NULL))

### research_fix_r1
co text?, st text?, city_name text?, new_state text?

### research_fix_r6
co text?, st text?, new_state text?

### research_fix_typo
co text?, st text?, n bigint?, best text?, sim numeric?, sim2 numeric?, rejected boolean=false

### research_fix_web
d text?, k text?, school_id uuid?, n bigint?
- idx research_fix_web_d_idx USING btree (d)

### research_fix_web_a
d text?, k text?, site text?, method text?, sure text?, name_match boolean?

### research_folder_members
folder_id uuid, contact_id uuid, method text, added_by text=''::text, added_at timestamp with time zone=now(), removed boolean=false, removed_by text?, removed_at timestamp with time zone?
- CK research_folder_members_method_check: CHECK ((method = ANY (ARRAY['rule'::text, 'manual'::text, 'upload'::text])))
- FK research_folder_members_folder_id_fkey: FOREIGN KEY (folder_id) REFERENCES research_folders(id)
- PK research_folder_members_pkey: PRIMARY KEY (folder_id, contact_id)
- idx research_folder_members_contact_idx USING btree (contact_id)
- idx research_folder_members_live_idx USING btree (folder_id) WHERE (NOT removed)

### research_folders
id uuid=gen_random_uuid(), key text, name text, description text=''::text, kind text='custom'::text, programme_family text?, rules jsonb='{}'::jsonb, color text=''::text, sort integer=100, archived boolean=false, created_by text=''::text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), refreshed_at timestamp with time zone?
- CK research_folders_kind_check: CHECK ((kind = ANY (ARRAY['programme'::text, 'custom'::text])))
- PK research_folders_pkey: PRIMARY KEY (id)
- UQ research_folders_key_key: UNIQUE (key)

### research_generated_docs
id uuid=gen_random_uuid(), kind text, title text=''::text, contact_id uuid?, country text=''::text, state text=''::text, city text=''::text, body jsonb='{}'::jsonb, facts jsonb='{}'::jsonb, model text=''::text, status text='draft'::text, created_by text?, created_at timestamp with time zone=now(), decided_by text?, decided_at timestamp with time zone?
- FK research_generated_docs_contact_id_fkey: FOREIGN KEY (contact_id) REFERENCES research_contacts(id) ON DELETE SET NULL
- PK research_generated_docs_pkey: PRIMARY KEY (id)
- idx research_generated_docs_kind USING btree (kind, created_at DESC)
- idx research_generated_docs_contact USING btree (contact_id)

### research_groups
id uuid=gen_random_uuid(), name text, notes text?, owner_team text='group'::text, created_at timestamp with time zone=now(), pitch_intro text?, campaign_owner text?, hold_from_country boolean=true, exclude_from_country_lists boolean=true, region text=''::text
- CK research_groups_owner_team_check: CHECK ((owner_team = ANY (ARRAY['group'::text, 'country'::text])))
- PK research_groups_pkey: PRIMARY KEY (id)
- UQ research_groups_name_key: UNIQUE (name)

### research_identity_matches
id uuid=gen_random_uuid(), kind text, a_id uuid, b_id uuid, reason text=''::text, status text='open'::text, decided_by text?, decided_at timestamp with time zone?, created_at timestamp with time zone=now()
- CK research_identity_matches_kind_check: CHECK ((kind = ANY (ARRAY['person'::text, 'school'::text])))
- CK research_identity_matches_status_check: CHECK ((status = ANY (ARRAY['open'::text, 'merged'::text, 'distinct'::text])))
- PK research_identity_matches_pkey: PRIMARY KEY (id)
- UQ research_identity_matches_kind_a_id_b_id_key: UNIQUE (kind, a_id, b_id)

### research_intel_chats
id uuid=gen_random_uuid(), title text='New chat'::text, owner_id uuid, owner_name text=''::text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK research_intel_chats_pkey: PRIMARY KEY (id)
- idx research_intel_chats_owner USING btree (owner_id, updated_at DESC)

### research_intel_messages
id uuid=gen_random_uuid(), chat_id uuid, role text, content text=''::text, payload jsonb?, created_at timestamp with time zone=now()
- CK research_intel_messages_role_check: CHECK ((role = ANY (ARRAY['user'::text, 'assistant'::text])))
- FK research_intel_messages_chat_id_fkey: FOREIGN KEY (chat_id) REFERENCES research_intel_chats(id) ON DELETE CASCADE
- PK research_intel_messages_pkey: PRIMARY KEY (id)
- idx research_intel_messages_chat USING btree (chat_id, created_at)

### research_job_locks
name text, holder text, expires_at timestamp with time zone
- PK research_job_locks_pkey: PRIMARY KEY (name)

### research_jobs
id uuid=gen_random_uuid(), kind text, dedupe_key text, payload jsonb='{}'::jsonb, status text='pending'::text, attempts integer=0, last_error text?, result text?, next_attempt_at timestamp with time zone=now(), created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK research_jobs_status_check: CHECK ((status = ANY (ARRAY['pending'::text, 'running'::text, 'done'::text, 'failed'::text, 'parked'::text])))
- PK research_jobs_pkey: PRIMARY KEY (id)
- idx research_jobs_open_key UNIQUE USING btree (kind, dedupe_key) WHERE (status = ANY (ARRAY['pending'::text, 'running'::text, 'parked'::text]))
- idx research_jobs_claim_idx USING btree (status, next_attempt_at)

### research_leads
id uuid=gen_random_uuid(), full_name text=''::text, email text=''::text, phone text=''::text, organisation text=''::text, designation text=''::text, city text=''::text, state text=''::text, country text=''::text, region text=''::text, source text=''::text, stage text='new'::text, notes text=''::text, custom jsonb='{}'::jsonb, upload_id uuid?, upload_tab_id uuid?, source_row_number integer?, source_file text=''::text, first_seen_at timestamp with time zone=now(), last_seen_at timestamp with time zone=now(), last_seen_upload_id uuid?, times_seen integer=1, created_by uuid?, created_at timestamp with time zone=now(), deleted_at timestamp with time zone?, deleted_by uuid?
- FK research_leads_created_by_fkey: FOREIGN KEY (created_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_leads_deleted_by_fkey: FOREIGN KEY (deleted_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_leads_last_seen_upload_id_fkey: FOREIGN KEY (last_seen_upload_id) REFERENCES research_uploads(id) ON DELETE SET NULL
- FK research_leads_upload_id_fkey: FOREIGN KEY (upload_id) REFERENCES research_uploads(id) ON DELETE SET NULL
- FK research_leads_upload_tab_id_fkey: FOREIGN KEY (upload_tab_id) REFERENCES research_upload_tabs(id) ON DELETE SET NULL
- PK research_leads_pkey: PRIMARY KEY (id)
- idx research_leads_email_idx USING btree (lower(email))
- idx research_leads_deleted_idx USING btree (deleted_at)
- idx research_leads_custom_gin USING gin (custom)

### research_legacy_source_map
value text, tool_key text?, researcher_label text?, note text=''::text, applied_at timestamp with time zone?, applied_by text?, applied_batch uuid?, applied_rows integer?, rows_n integer=0
- PK research_legacy_source_map_pkey: PRIMARY KEY (value)

### research_library_cursor
id integer=1, pos integer=0, cycle_started_at timestamp with time zone?, last_cycle_at timestamp with time zone?
- CK research_library_cursor_id_check: CHECK ((id = 1))
- PK research_library_cursor_pkey: PRIMARY KEY (id)

### research_locations
id uuid=gen_random_uuid(), country text, state text, city text=''::text, redo_status text='current'::text, created_at timestamp with time zone=now(), redo_started_at timestamp with time zone?, redo_by uuid?, last_verified_on date?, redo_cycle integer=1, next_redo_on date?, redo_note text?, mapped_by uuid?
- CK research_locations_redo_status_check: CHECK ((redo_status = ANY (ARRAY['current'::text, 'needs_redo'::text, 'in_progress'::text])))
- FK research_locations_mapped_by_fkey: FOREIGN KEY (mapped_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_locations_redo_by_fkey: FOREIGN KEY (redo_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_locations_pkey: PRIMARY KEY (id)
- idx research_locations_key UNIQUE USING btree (lower(country), lower(state), lower(city))

## public (research_m.. onwards)

### research_optouts
id uuid=gen_random_uuid(), email text, contact_id uuid?, campaign_id uuid?, token text=''::text, reason text=''::text, source text='one_click'::text, user_agent text=''::text, created_at timestamp with time zone=now()
- FK research_optouts_campaign_id_fkey: FOREIGN KEY (campaign_id) REFERENCES research_campaigns(id) ON DELETE SET NULL
- PK research_optouts_pkey: PRIMARY KEY (id)
- idx research_optouts_email_key UNIQUE USING btree (lower(email))
- idx research_optouts_campaign_idx USING btree (campaign_id, created_at DESC)

### research_outreach_claims
id uuid=gen_random_uuid(), scope text='school'::text, claim_key text, label text=''::text, campaign_id uuid?, note text=''::text, claimed_by uuid?, claimed_by_name text=''::text, expires_at timestamp with time zone=(now() + '14 days'::interval), released_at timestamp with time zone?, created_at timestamp with time zone=now()
- CK research_outreach_claims_scope_check: CHECK ((scope = ANY (ARRAY['school'::text, 'contact'::text, 'location'::text])))
- FK research_outreach_claims_campaign_id_fkey: FOREIGN KEY (campaign_id) REFERENCES research_campaigns(id) ON DELETE SET NULL
- FK research_outreach_claims_claimed_by_fkey: FOREIGN KEY (claimed_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_outreach_claims_pkey: PRIMARY KEY (id)
- idx research_outreach_claims_live_key UNIQUE USING btree (scope, claim_key) WHERE (released_at IS NULL)
- idx research_outreach_claims_recent_idx USING btree (created_at DESC)

### research_pages
url text, school_id uuid?, kind text=''::text, title text=''::text, text text=''::text, fetched_at timestamp with time zone=now(), ok boolean=true
- FK research_pages_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE SET NULL
- PK research_pages_pkey: PRIMARY KEY (url)
- idx research_pages_school_idx USING btree (school_id)

### research_people
id uuid=gen_random_uuid(), contact_code text=research_gen_code('CON'::text, 'research_people'::text, 'contact_code'::text), full_name text=''::text, person_type text='staff'::text, designation text=''::text, current_school_id uuid?, primary_email text=''::text, primary_phone text=''::text, name_history jsonb='[]'::jsonb, merged_into uuid?, created_by text=''::text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), legacy_code text?, student_code text?
- CK research_people_person_type_check: CHECK ((person_type = ANY (ARRAY['staff'::text, 'student'::text, 'parent'::text, 'counsellor'::text, 'other'::text])))
- FK research_people_current_school_id_fkey: FOREIGN KEY (current_school_id) REFERENCES research_schools(id) ON DELETE SET NULL
- FK research_people_merged_into_fkey: FOREIGN KEY (merged_into) REFERENCES research_people(id) ON DELETE SET NULL
- PK research_people_pkey: PRIMARY KEY (id)
- UQ research_people_contact_code_key: UNIQUE (contact_code)
- idx research_people_school_idx USING btree (current_school_id)
- idx research_people_type_idx USING btree (person_type)
- idx research_people_student_code_key UNIQUE USING btree (upper(student_code)) WHERE ((student_code IS NOT NULL) AND (student_code <> ''::text))
- idx research_people_legacy_idx USING btree (legacy_code)
- idx research_people_primary_email_trgm_live_idx USING gin (primary_email gin_trgm_ops) WHERE (merged_into IS NULL)

### research_person_emails
id uuid=gen_random_uuid(), person_id uuid, email text, first_seen_at timestamp with time zone=now(), last_seen_at timestamp with time zone=now(), source text=''::text
- FK research_person_emails_person_id_fkey: FOREIGN KEY (person_id) REFERENCES research_people(id) ON DELETE CASCADE
- PK research_person_emails_pkey: PRIMARY KEY (id)
- idx research_person_emails_key UNIQUE USING btree (lower(email))
- idx research_person_emails_person_idx USING btree (person_id)
- idx research_person_emails_email_plain_idx USING btree (email)

### research_person_events
id uuid=gen_random_uuid(), person_id uuid?, contact_id uuid?, school_id uuid?, kind text, detail text=''::text, evidence text=''::text, source text=''::text, status text='done'::text, happened_at timestamp with time zone=now(), created_by text=''::text, created_at timestamp with time zone=now()
- CK research_person_events_status_check: CHECK ((status = ANY (ARRAY['done'::text, 'proposed'::text, 'rejected'::text])))
- FK research_person_events_contact_id_fkey: FOREIGN KEY (contact_id) REFERENCES research_contacts(id) ON DELETE SET NULL
- FK research_person_events_person_id_fkey: FOREIGN KEY (person_id) REFERENCES research_people(id) ON DELETE CASCADE
- FK research_person_events_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE SET NULL
- PK research_person_events_pkey: PRIMARY KEY (id)
- idx research_person_events_person_idx USING btree (person_id, happened_at DESC)
- idx research_person_events_contact_idx USING btree (contact_id)
- idx research_person_events_open_idx USING btree (status) WHERE (status = 'proposed'::text)

### research_person_links
id uuid=gen_random_uuid(), person_id uuid, related_id uuid, kind text, source text=''::text, created_at timestamp with time zone=now()
- CK research_person_links_kind_check: CHECK ((kind = ANY (ARRAY['parent_of'::text, 'sibling_of'::text, 'counsellor_of'::text])))
- FK research_person_links_person_id_fkey: FOREIGN KEY (person_id) REFERENCES research_people(id) ON DELETE CASCADE
- FK research_person_links_related_id_fkey: FOREIGN KEY (related_id) REFERENCES research_people(id) ON DELETE CASCADE
- PK research_person_links_pkey: PRIMARY KEY (id)
- UQ research_person_links_person_id_related_id_kind_key: UNIQUE (person_id, related_id, kind)

### research_person_phones
id uuid=gen_random_uuid(), person_id uuid, phone text, first_seen_at timestamp with time zone=now(), source text=''::text
- FK research_person_phones_person_id_fkey: FOREIGN KEY (person_id) REFERENCES research_people(id) ON DELETE CASCADE
- PK research_person_phones_pkey: PRIMARY KEY (id)
- idx research_person_phones_key UNIQUE USING btree (phone)

### research_person_schools
id uuid=gen_random_uuid(), person_id uuid, school_id uuid, role text='staff'::text, designation text=''::text, started_on date?, ended_on date?, is_current boolean=true, source text=''::text, created_at timestamp with time zone=now()
- CK research_person_schools_role_check: CHECK ((role = ANY (ARRAY['staff'::text, 'student'::text, 'parent'::text, 'counsellor'::text, 'other'::text])))
- FK research_person_schools_person_id_fkey: FOREIGN KEY (person_id) REFERENCES research_people(id) ON DELETE CASCADE
- FK research_person_schools_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE CASCADE
- PK research_person_schools_pkey: PRIMARY KEY (id)
- idx research_person_schools_key UNIQUE USING btree (person_id, school_id, role)
- idx research_person_schools_school_idx USING btree (school_id)

### research_pipeline_links
id uuid=gen_random_uuid(), contact_id uuid, match_kind text, match_confidence numeric(4,3)=0, matched_value text?, person_id uuid?, organization_id uuid?, org_code text?, status text='linked'::text, export_id uuid?, note text?, first_linked_at timestamp with time zone=now(), linked_by text='matcher'::text, decided_at timestamp with time zone?, decided_by text?
- CK research_pipeline_links_match_kind_check: CHECK ((match_kind = ANY (ARRAY['email'::text, 'domain'::text, 'school_name'::text, 'school_alias'::text])))
- CK research_pipeline_links_status_check: CHECK ((status = ANY (ARRAY['linked'::text, 'suggested'::text, 'rejected'::text])))
- FK research_pipeline_links_contact_id_fkey: FOREIGN KEY (contact_id) REFERENCES research_contacts(id) ON DELETE CASCADE
- PK research_pipeline_links_pkey: PRIMARY KEY (id)
- idx research_pipeline_links_unique UNIQUE USING btree (contact_id, match_kind)
- idx research_pipeline_links_org USING btree (organization_id)
- idx research_pipeline_links_person USING btree (person_id)
- idx research_pipeline_links_status USING btree (status)

### research_place_fill_plan
contact_id uuid, new_city text?, new_state text?, old_city text?, old_state text?, support text=''::text, batch uuid, done_at timestamp with time zone?
- PK research_place_fill_plan_pkey: PRIMARY KEY (contact_id)
- idx research_place_fill_plan_todo_idx USING btree (contact_id) WHERE (done_at IS NULL)

### research_proposals
id uuid=gen_random_uuid(), contact_id uuid, kind text, field text?, current_value text?, proposed_value text?, confidence numeric(4,3)=0, reasoning text=''::text, source_run text?, status text='open'::text, created_at timestamp with time zone=now(), decided_at timestamp with time zone?, decided_by text?
- CK research_proposals_status_check: CHECK ((status = ANY (ARRAY['open'::text, 'accepted'::text, 'rejected'::text])))
- FK research_proposals_contact_id_fkey: FOREIGN KEY (contact_id) REFERENCES research_contacts(id) ON DELETE CASCADE
- PK research_proposals_pkey: PRIMARY KEY (id)
- idx research_proposals_unique UNIQUE USING btree (contact_id, kind, COALESCE(field, ''::text))
- idx research_proposals_status USING btree (status, kind)
- idx research_proposals_confidence USING btree (confidence DESC)

### research_raw_rows
id bigint=nextval('research_raw_rows_id_seq'::regclass), upload_tab_id uuid, row_number integer, data jsonb, outcome text?, reason text?, contact_id uuid?, claimed_at timestamp with time zone?
- CK research_raw_rows_outcome_chk: CHECK (((outcome IS NULL) OR (outcome = ANY (ARRAY['imported'::text, 'merged'::text, 'duplicate'::text, 'review'::text, 'skipped'::text, 'failed'::text]))))
- FK research_raw_rows_upload_tab_id_fkey: FOREIGN KEY (upload_tab_id) REFERENCES research_upload_tabs(id) ON DELETE CASCADE
- PK research_raw_rows_pkey: PRIMARY KEY (id)
- idx research_raw_rows_tab_idx USING btree (upload_tab_id)
- idx research_raw_rows_outcome_idx USING btree (upload_tab_id, outcome)
- idx research_raw_rows_tab_row_uidx UNIQUE USING btree (upload_tab_id, row_number)
- idx research_raw_rows_contact_idx USING btree (contact_id)
- idx research_raw_rows_pending_idx USING btree (upload_tab_id, row_number) WHERE ((outcome IS NULL) OR (outcome = 'processing'::text))
- idx research_raw_rows_unprocessed_idx USING btree (upload_tab_id, row_number) WHERE (outcome IS NULL)

### research_reach_cache
key text, filters jsonb, page integer=1, payload jsonb, computed_at timestamp with time zone=now(), last_used_at timestamp with time zone=now(), ms integer=0
- PK research_reach_cache_pkey: PRIMARY KEY (key)

### research_record_actions
id uuid=gen_random_uuid(), record_type text, record_id text, record_label text=''::text, kind text='note'::text, note text=''::text, status text='open'::text, due_at timestamp with time zone?, owner_email text=''::text, owner_name text=''::text, created_by text=''::text, created_by_agent boolean=false, created_at timestamp with time zone=now(), completed_at timestamp with time zone?, completed_by text=''::text
- CK research_record_actions_kind_check: CHECK ((kind = ANY (ARRAY['own'::text, 'note'::text, 'snooze'::text, 'escalate'::text, 'done'::text, 'dismiss'::text, 'email_drafted'::text])))
- CK research_record_actions_status_check: CHECK ((status = ANY (ARRAY['open'::text, 'done'::text, 'dismissed'::text])))
- PK research_record_actions_pkey: PRIMARY KEY (id)
- idx research_record_actions_record_idx USING btree (record_type, record_id, created_at DESC)
- idx research_record_actions_open_idx USING btree (status, due_at)
- idx research_record_actions_owner_idx USING btree (owner_email, status)

### research_ref_cities
country text, state text, name text
- PK research_ref_cities_pkey: PRIMARY KEY (country, state, name)
- idx research_ref_cities_lname_idx USING btree (country, lower(name))

### research_ref_countries
name text, iso2 text, tld text=''::text, phonecode text=''::text, region text=''::text
- PK research_ref_countries_pkey: PRIMARY KEY (name)

### research_ref_states
country text, name text, code text=''::text
- PK research_ref_states_pkey: PRIMARY KEY (country, name)
- idx research_ref_states_lname_idx USING btree (lower(name))
- idx research_ref_states_lcode_idx USING btree (country, lower(code))

### research_registry_dry
at timestamp with time zone?=now(), result jsonb?

### research_registry_schools
source text, registry_id text, name text, name_key text, street text=''::text, city text=''::text, state text=''::text, zip text=''::text, phone text=''::text, district text=''::text, district_id text=''::text, county_code text=''::text, lowest_grade integer?, highest_grade integer?, school_level integer?, school_type integer?, charter boolean?, virtual boolean?, status integer?, enrollment integer?, year integer?, imported_at timestamp with time zone=now(), name_key2 text=''::text
- PK research_registry_schools_pkey: PRIMARY KEY (source, registry_id)
- idx research_registry_schools_match_idx USING btree (name_key, state)
- idx research_registry_schools_match2_idx USING btree (name_key2, state)

### research_resolve_items
id uuid=gen_random_uuid(), run_id uuid, review_row_id uuid?, line integer=0, school text=''::text, email text=''::text, outcome text, confidence numeric=0, changes jsonb='[]'::jsonb, note text=''::text, original jsonb='{}'::jsonb, contact_id uuid?, existing_id uuid?, applied_at timestamp with time zone?, undone_at timestamp with time zone?, created_at timestamp with time zone=now()
- CK research_resolve_items_outcome_check: CHECK ((outcome = ANY (ARRAY['fix'::text, 'held'::text, 'duplicate'::text])))
- FK research_resolve_items_run_id_fkey: FOREIGN KEY (run_id) REFERENCES research_resolve_runs(id) ON DELETE CASCADE
- PK research_resolve_items_pkey: PRIMARY KEY (id)
- idx research_resolve_items_run_idx USING btree (run_id, outcome)
- idx research_resolve_items_applied_live_idx USING btree (applied_at DESC) WHERE (undone_at IS NULL)

### research_resolve_runs
id uuid=gen_random_uuid(), started_by text=''::text, started_by_id uuid?, started_at timestamp with time zone=now(), finished_at timestamp with time zone?, scope text='selected'::text, threshold numeric=0.9, counts jsonb='{}'::jsonb, status text='previewed'::text
- CK research_resolve_runs_status_check: CHECK ((status = ANY (ARRAY['previewed'::text, 'applying'::text, 'applied'::text, 'undone'::text])))
- PK research_resolve_runs_pkey: PRIMARY KEY (id)
- idx research_resolve_runs_started_idx USING btree (started_at DESC)

### research_review_domain_school
domain text, school_key text?, school text?, support integer?, total integer?, share numeric?, city text?, city_share numeric?, state text?, state_share numeric?, country text?, country_share numeric?, school_id uuid?, built_at timestamp with time zone?=now()
- PK research_review_domain_school_pkey: PRIMARY KEY (domain)

### research_review_outcomes
id uuid, dataset_id uuid?, dataset_code text=''::text, line integer=0, fields jsonb='{}'::jsonb, reason text=''::text, reason_bucket text=''::text, source text=''::text, origin text=''::text, queued_by text=''::text, queued_at timestamp with time zone?, outcome text, note text=''::text, evidence text=''::text, contact_id uuid?, batch uuid, settled_by text=''::text, settled_at timestamp with time zone=now()
- FK research_review_outcomes_contact_id_fkey: FOREIGN KEY (contact_id) REFERENCES research_contacts(id) ON DELETE SET NULL
- PK research_review_outcomes_pkey: PRIMARY KEY (id)
- idx research_review_outcomes_outcome_idx USING btree (outcome, settled_at DESC)
- idx research_review_outcomes_batch_idx USING btree (batch)
- idx research_review_outcomes_lead_domain_idx USING btree (lower(split_part((fields ->> 'email'::text), '@'::text, 2))) WHERE (outcome = 'lead'::text)

### research_review_rows
id uuid=gen_random_uuid(), batch_id uuid=gen_random_uuid(), dataset_id uuid?, dataset_code text='-'::text, line integer=0, fields jsonb='{}'::jsonb, missing text[]='{}'::text[], reason text=''::text, autofilled text[]='{}'::text[], state text='hold'::text, source text=''::text, origin text='Import'::text, queued_by text=''::text, queued_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), quarantined_at timestamp with time zone?, tried jsonb='[]'::jsonb, reason_bucket text?=research_review_bucket(reason)
- CK research_review_rows_state_check: CHECK ((state = ANY (ARRAY['hold'::text, 'unverified'::text])))
- FK research_review_rows_dataset_id_fkey: FOREIGN KEY (dataset_id) REFERENCES research_datasets(id) ON DELETE SET NULL
- PK research_review_rows_pkey: PRIMARY KEY (id)
- idx research_review_rows_state_idx USING btree (state, queued_at DESC)
- idx research_review_rows_batch_idx USING btree (batch_id)
- idx research_review_rows_quarantined_idx USING btree (quarantined_at)
- idx research_review_rows_open_idx USING btree (queued_at DESC) WHERE ((state = 'hold'::text) AND (quarantined_at IS NULL))
- idx research_review_rows_bucket_idx USING btree (reason_bucket, queued_at DESC) WHERE ((state = 'hold'::text) AND (quarantined_at IS NULL))

### research_saved_views
id uuid=gen_random_uuid(), name text, description text=''::text, query jsonb='{}'::jsonb, view_mode text='table'::text, group_by text?, group_by_2 text?, pinned boolean=false, created_by text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- PK research_saved_views_pkey: PRIMARY KEY (id)
- idx research_saved_views_name UNIQUE USING btree (lower(name))
- idx research_saved_views_pinned USING btree (pinned)

### research_schema_export_test
id integer, sql text, result text?
- PK research_schema_export_test_pkey: PRIMARY KEY (id)

### research_school_facts
school_id uuid, field text, value text=''::text, status text='UNKNOWN'::text, source_url text=''::text, quote text=''::text, method text=''::text, note text=''::text, task_id uuid?, verified_at timestamp with time zone?, expires_at timestamp with time zone?, updated_at timestamp with time zone=now()
- CK research_school_facts_status_check: CHECK ((status = ANY (ARRAY['VERIFIED'::text, 'NOT_APPLICABLE'::text, 'NOT_PUBLIC'::text, 'UNKNOWN'::text, 'CONFLICT'::text, 'HOLD'::text, 'ON_FILE'::text])))
- FK research_school_facts_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE CASCADE
- PK research_school_facts_pkey: PRIMARY KEY (school_id, field)
- idx research_school_facts_field_idx USING btree (field, status)

### research_school_findings
id uuid=gen_random_uuid(), task_id uuid?, school_id uuid?, domain text=''::text, kind text, field text=''::text, value text=''::text, extra jsonb='{}'::jsonb, source_url text=''::text, quote text=''::text, verified boolean=false, why text=''::text, applied text=''::text, created_at timestamp with time zone=now()
- FK research_school_findings_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE CASCADE
- FK research_school_findings_task_id_fkey: FOREIGN KEY (task_id) REFERENCES research_tasks(id) ON DELETE CASCADE
- PK research_school_findings_pkey: PRIMARY KEY (id)
- idx research_school_findings_school_idx USING btree (school_id, created_at DESC)
- idx research_school_findings_task_idx USING btree (task_id)
- idx research_school_findings_kind_idx USING btree (kind, verified)
- idx research_school_findings_person_idx USING btree (kind, verified, school_id)

### research_school_intel
id uuid=gen_random_uuid(), school_id uuid, profile jsonb='{}'::jsonb, facts jsonb='{}'::jsonb, model text=''::text, generated_by text=''::text, generated_at timestamp with time zone=now()
- FK research_school_intel_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE CASCADE
- PK research_school_intel_pkey: PRIMARY KEY (id)
- UQ research_school_intel_school_id_key: UNIQUE (school_id)

### research_school_link_plan
contact_id uuid, school text?, city text?, state text?, country text?, d text?, rule text?, sid uuid?, done_at timestamp with time zone?
- PK research_school_link_plan_pkey: PRIMARY KEY (contact_id)
- idx research_school_link_plan_todo_idx USING btree (contact_id) WHERE (done_at IS NULL)

### research_school_links
id bigint=nextval('research_school_links_id_seq'::regclass), contact_id uuid, school_id uuid, rule text, created_school boolean=false, batch uuid, actor text=''::text, at timestamp with time zone=now()
- PK research_school_links_pkey: PRIMARY KEY (id)
- idx research_school_links_batch_idx USING btree (batch)

### research_school_notes
id uuid=gen_random_uuid(), school_id uuid, kind text='note'::text, title text=''::text, body text=''::text, source_url text=''::text, happened_on date?, created_by text=''::text, created_at timestamp with time zone=now(), deleted_at timestamp with time zone?
- CK research_school_notes_kind_check: CHECK ((kind = ANY (ARRAY['note'::text, 'news'::text, 'report'::text, 'activity'::text, 'focus'::text])))
- FK research_school_notes_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE CASCADE
- PK research_school_notes_pkey: PRIMARY KEY (id)
- idx research_school_notes_school_idx USING btree (school_id, created_at DESC)

### research_school_profiles
id uuid=gen_random_uuid(), school text, city text=''::text, state text=''::text, country text=''::text, board text=''::text, grades_offered text=''::text, eligible_grades text=''::text, accreditations text=''::text, accreditation_changed_at date?, status text='existing'::text, first_seen_year integer?, last_verified_on date?, verified_by uuid?, notes text?, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now()
- CK research_school_profiles_status_check: CHECK ((status = ANY (ARRAY['new'::text, 'existing'::text, 'closed'::text, 'merged'::text])))
- FK research_school_profiles_verified_by_fkey: FOREIGN KEY (verified_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_school_profiles_pkey: PRIMARY KEY (id)
- idx research_school_profiles_key UNIQUE USING btree (lower(school), lower(city), lower(country))
- idx research_school_profiles_status_idx USING btree (status)

### research_school_web
school_id uuid, checked_at timestamp with time zone=now(), url text=''::text, pages jsonb='[]'::jsonb, city text=''::text, city_evidence text=''::text, people jsonb='[]'::jsonb, error text=''::text
- FK research_school_web_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE CASCADE
- PK research_school_web_pkey: PRIMARY KEY (school_id)

### research_schools
id uuid=gen_random_uuid(), school_code text=research_gen_code('SCH'::text, 'research_schools'::text, 'school_code'::text), name text, branch text=''::text, city text=''::text, state text=''::text, country text=''::text, domain text=''::text, website text=''::text, school_type text=''::text, curriculum text=''::text, merged_into uuid?, created_by text=''::text, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), legacy_code text?, registry_source text=''::text, registry_id text=''::text
- FK research_schools_merged_into_fkey: FOREIGN KEY (merged_into) REFERENCES research_schools(id) ON DELETE SET NULL
- PK research_schools_pkey: PRIMARY KEY (id)
- UQ research_schools_school_code_key: UNIQUE (school_code)
- idx research_schools_key UNIQUE USING btree (lower(name), lower(branch), lower(city), lower(country))
- idx research_schools_domain_idx USING btree (lower(domain))
- idx research_schools_legacy_idx USING btree (legacy_code)

### research_segments
id uuid=gen_random_uuid(), name text, kind text='custom'::text, exclude_from_country_lists boolean=true, notes text=''::text, created_by uuid?, created_at timestamp with time zone=now()
- CK research_segments_kind_check: CHECK ((kind = ANY (ARRAY['dnd'::text, 'scholarship'::text, 'group'::text, 'custom'::text])))
- FK research_segments_created_by_fkey: FOREIGN KEY (created_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_segments_pkey: PRIMARY KEY (id)
- UQ research_segments_name_key: UNIQUE (name)

### research_send_exports
id uuid=gen_random_uuid(), label text=''::text, scope text='dataset'::text, dataset_id uuid?, segment_id uuid?, country text=''::text, state text=''::text, row_count integer=0, removed jsonb='{}'::jsonb, cap_per_day integer=0, contact_ids uuid[]='{}'::uuid[], exported_by uuid?, created_at timestamp with time zone=now()
- FK research_send_exports_dataset_id_fkey: FOREIGN KEY (dataset_id) REFERENCES research_datasets(id) ON DELETE SET NULL
- FK research_send_exports_exported_by_fkey: FOREIGN KEY (exported_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_send_exports_segment_id_fkey: FOREIGN KEY (segment_id) REFERENCES research_segments(id) ON DELETE SET NULL
- PK research_send_exports_pkey: PRIMARY KEY (id)
- idx research_send_exports_created_idx USING btree (created_at DESC)

### research_settings
key text, value jsonb, updated_at timestamp with time zone=now()
- PK research_settings_pkey: PRIMARY KEY (key)

### research_short_links
slug text, target_url text, label text=''::text, campaign_id uuid?, created_by uuid?, created_by_email text=''::text, hits integer=0, last_hit_at timestamp with time zone?, created_at timestamp with time zone=now()
- PK research_short_links_pkey: PRIMARY KEY (slug)
- idx research_short_links_campaign_idx USING btree (campaign_id, created_at DESC)

### research_sop_acknowledgements
id uuid=gen_random_uuid(), version_id uuid, staff_id uuid, acknowledged_at timestamp with time zone=now()
- FK research_sop_acknowledgements_staff_id_fkey: FOREIGN KEY (staff_id) REFERENCES research_staff_profiles(id) ON DELETE CASCADE
- FK research_sop_acknowledgements_version_id_fkey: FOREIGN KEY (version_id) REFERENCES research_sop_versions(id) ON DELETE CASCADE
- PK research_sop_acknowledgements_pkey: PRIMARY KEY (id)
- UQ research_sop_acknowledgements_version_id_staff_id_key: UNIQUE (version_id, staff_id)

### research_sop_versions
id uuid=gen_random_uuid(), version text, summary text=''::text, effective_on date=CURRENT_DATE, created_at timestamp with time zone=now()
- PK research_sop_versions_pkey: PRIMARY KEY (id)
- UQ research_sop_versions_version_key: UNIQUE (version)

### research_source_tools
key text, label text, kind text, needs_verification boolean=false, aliases text[]='{}'::text[], active boolean=true, sort integer=100
- CK research_source_tools_kind_check: CHECK ((kind = ANY (ARRAY['website'::text, 'ai_tool'::text, 'contact_tool'::text, 'directory'::text, 'manual'::text, 'agent'::text, 'other'::text])))
- PK research_source_tools_pkey: PRIMARY KEY (key)

### research_spend
id uuid=gen_random_uuid(), at timestamp with time zone=now(), kind text, model text=''::text, task_id uuid?, school_id uuid?, input_tokens integer=0, output_tokens integer=0, searches integer=0, cost_usd numeric(10,5)=0, note text=''::text, actor text=''::text
- FK research_spend_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE SET NULL
- FK research_spend_task_id_fkey: FOREIGN KEY (task_id) REFERENCES research_tasks(id) ON DELETE SET NULL
- PK research_spend_pkey: PRIMARY KEY (id)
- idx research_spend_at_idx USING btree (at DESC)
- idx research_spend_task_idx USING btree (task_id)

### research_staff_profiles
id uuid=gen_random_uuid(), user_id uuid?, name text, email text, role text='researcher'::text, staff_type text='FT'::text, daily_contact_target integer=100, active boolean=true, created_at timestamp with time zone=now(), can_export boolean=false
- CK research_staff_profiles_role_check: CHECK ((role = ANY (ARRAY['admin'::text, 'researcher'::text, 'sales'::text, 'intern'::text])))
- CK research_staff_profiles_staff_type_check: CHECK ((staff_type = ANY (ARRAY['FT'::text, 'PT'::text])))
- FK research_staff_profiles_user_id_fkey: FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
- PK research_staff_profiles_pkey: PRIMARY KEY (id)
- UQ research_staff_profiles_email_key: UNIQUE (email)
- UQ research_staff_profiles_user_id_key: UNIQUE (user_id)

### research_stats_contributions
upload_tab_id uuid, researcher_label text, researcher_staff_id uuid?, researcher_basis text, tool_key text, researched_month date='1900-01-01'::date, outcome text, rows_n integer, clean_now integer, email_ready_now integer, ai_rows integer, computed_at timestamp with time zone, chunk_no integer=0, bad_email_now integer=0
- PK research_stats_contributions_pkey: PRIMARY KEY (upload_tab_id, chunk_no, researcher_label, researcher_basis, tool_key, researched_month, outcome)

### research_stats_contributions_meta
id integer=1, computed_at timestamp with time zone?
- CK research_stats_contributions_meta_id_check: CHECK ((id = 1))
- PK research_stats_contributions_meta_pkey: PRIMARY KEY (id)

### research_stats_email_checks
id integer=1, ready bigint=0, risky bigint=0, do_not_send bigint=0, unchecked bigint=0, refreshed_at timestamp with time zone?
- CK research_stats_email_checks_id_check: CHECK ((id = 1))
- PK research_stats_email_checks_pkey: PRIMARY KEY (id)

### research_tasks
id uuid=gen_random_uuid(), kind text, dedupe_key text, status text='open'::text, priority integer=0, title text=''::text, reason text=''::text, school_id uuid?, domain text=''::text, payload jsonb='{}'::jsonb, result jsonb='{}'::jsonb, attempts integer=0, last_error text=''::text, cost_usd numeric(10,4)=0, created_at timestamp with time zone=now(), updated_at timestamp with time zone=now(), started_at timestamp with time zone?, finished_at timestamp with time zone?, created_by text=''::text, retry_class text=''::text, next_retry_at timestamp with time zone?, max_attempts integer=3, retry_parked boolean=false
- CK research_tasks_status_check: CHECK ((status = ANY (ARRAY['open'::text, 'queued'::text, 'running'::text, 'done'::text, 'needs_review'::text, 'failed'::text, 'skipped'::text])))
- FK research_tasks_school_id_fkey: FOREIGN KEY (school_id) REFERENCES research_schools(id) ON DELETE CASCADE
- PK research_tasks_pkey: PRIMARY KEY (id)
- UQ research_tasks_dedupe_key_key: UNIQUE (dedupe_key)
- idx research_tasks_queue_idx USING btree (status, priority DESC, created_at)
- idx research_tasks_kind_idx USING btree (kind, status)
- idx research_tasks_school_idx USING btree (school_id)
- idx research_tasks_retry_due_idx USING btree (next_retry_at, priority DESC) WHERE ((status = 'failed'::text) AND (retry_parked = false) AND (next_retry_at IS NOT NULL))

### research_territory_briefs
id uuid=gen_random_uuid(), country text, state text=''::text, city text=''::text, official_source text=''::text, associations text=''::text, accreditation_bodies text=''::text, counselor_associations text=''::text, expected_curricula text=''::text, notes text=''::text, snapshot jsonb?, snapshot_at timestamp with time zone?, signed_off_by uuid?, signed_off_at timestamp with time zone?, created_by uuid?, created_at timestamp with time zone=now()
- FK research_territory_briefs_created_by_fkey: FOREIGN KEY (created_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_territory_briefs_signed_off_by_fkey: FOREIGN KEY (signed_off_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_territory_briefs_pkey: PRIMARY KEY (id)
- idx research_territory_briefs_key UNIQUE USING btree (lower(country), lower(state), lower(city))

### research_tracking_events
id uuid=gen_random_uuid(), token text, campaign_id uuid?, contact_id uuid?, email text=''::text, kind text, target_url text=''::text, user_agent text=''::text, occurred_at timestamp with time zone=now()
- CK research_tracking_events_kind_check: CHECK ((kind = ANY (ARRAY['open'::text, 'click'::text])))
- PK research_tracking_events_pkey: PRIMARY KEY (id)
- idx research_tracking_events_campaign_idx USING btree (campaign_id, occurred_at DESC)
- idx research_tracking_events_token_idx USING btree (token, kind)

### research_tracking_tokens
token text, campaign_id uuid?, contact_id uuid?, email text=''::text, target_url text=''::text, created_at timestamp with time zone=now(), kind text='track'::text
- FK research_tracking_tokens_campaign_id_fkey: FOREIGN KEY (campaign_id) REFERENCES research_campaigns(id) ON DELETE CASCADE
- PK research_tracking_tokens_pkey: PRIMARY KEY (token)
- idx research_tracking_tokens_campaign_idx USING btree (campaign_id)

### research_trust_totals_cache
state text, reason text=''::text, n bigint, computed_at timestamp with time zone=now()
- PK research_trust_totals_cache_pkey: PRIMARY KEY (state, reason)

### research_upload_tabs
id uuid=gen_random_uuid(), upload_id uuid, tab_name text, row_count integer=0, column_count integer=0, detected_headers jsonb='[]'::jsonb, mapping jsonb='{}'::jsonb, target_lane text='skip'::text, dataset_id uuid?, status text='pending'::text, imported_rows integer=0, merged_rows integer=0, duplicate_rows integer=0, review_rows integer=0, failed_rows integer=0, imported_at timestamp with time zone?, default_research_tool text?, default_research_tool_note text=''::text, folder_keys text[]='{}'::text[]
- CK research_upload_tabs_lane_chk: CHECK ((target_lane = ANY (ARRAY['contact'::text, 'b2c'::text, 'lead'::text, 'skip'::text])))
- CK research_upload_tabs_status_chk: CHECK ((status = ANY (ARRAY['pending'::text, 'skipped'::text, 'imported'::text, 'failed'::text])))
- FK research_upload_tabs_dataset_id_fkey: FOREIGN KEY (dataset_id) REFERENCES research_datasets(id) ON DELETE SET NULL
- FK research_upload_tabs_upload_id_fkey: FOREIGN KEY (upload_id) REFERENCES research_uploads(id) ON DELETE CASCADE
- PK research_upload_tabs_pkey: PRIMARY KEY (id)
- idx research_upload_tabs_upload_idx USING btree (upload_id)

### research_uploads
id uuid=gen_random_uuid(), file_name text, file_size bigint=0, source_kind text='file'::text, uploaded_by uuid?, uploaded_at timestamp with time zone=now(), tab_count integer=0, status text='pending'::text, note text?, researcher_id uuid?, researched_on date?, researcher_ids uuid[]='{}'::uuid[], researched_until date?
- CK research_uploads_status_chk: CHECK ((status = ANY (ARRAY['pending'::text, 'partial'::text, 'done'::text, 'failed'::text])))
- FK research_uploads_researcher_id_fkey: FOREIGN KEY (researcher_id) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- FK research_uploads_uploaded_by_fkey: FOREIGN KEY (uploaded_by) REFERENCES research_staff_profiles(id) ON DELETE SET NULL
- PK research_uploads_pkey: PRIMARY KEY (id)
- idx research_uploads_uploaded_at_idx USING btree (uploaded_at DESC)
- idx research_uploads_researcher_idx USING btree (researcher_id)

### research_us_states
code text, name text
- PK research_us_states_pkey: PRIMARY KEY (code)

### research_view_snapshots
id uuid=gen_random_uuid(), view_id uuid, taken_on date=(now())::date, row_count integer=0, created_at timestamp with time zone=now()
- FK research_view_snapshots_view_id_fkey: FOREIGN KEY (view_id) REFERENCES research_saved_views(id) ON DELETE CASCADE
- PK research_view_snapshots_pkey: PRIMARY KEY (id)
- idx research_view_snapshots_unique UNIQUE USING btree (view_id, taken_on)

